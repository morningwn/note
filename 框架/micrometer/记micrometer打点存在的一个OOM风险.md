# 记micrometer打点存在的一个OOM风险
> 最近在将dubbo服务自动打点封装的时候被大佬提出来一个OOM的风险，去研究了一下，发现虽然正常情况下很难出现，但是的确是有这种风险，在这里记录一下问题。

## 场景
### 打点
>我封装的自动打点工具类默认情况下会将调用方的ip、调用的方法名称、抛出的异常等给记录下来。代码如下：
```Java
    sample.stop(Timer.builder("")
        .tag("exception", exceptionClass)
        .tag("className", className)
        .tag("simpleClassName", simpleClassName)
        .tag("method", methodName)
        .tag("ip", ip)
        .publishPercentileHistogram(false)
        .publishPercentiles(null)
        .register(registry));
```

### 问题
> 打点一看挺正常的啊，都是可能会用到的内容。但是如果对原理有了解的话应该很容易就明白了问题所在：监控的指标是保存在**内存**里面的，不同的指标是根据tag和名称做区分的；在这个地方，异常、类名、方法名正常情况都是可控的，也就是说如果只有这几个指标的话，你内存里面保存多少个指标的数量是固定的，但是ip就不可控了，如果你的服务调用方特别多的话就有一定的OOM风险。

涉及到的关键代码如下：

- ``io.micrometer.core.instrument.Timer``
```Java
        public Timer register(MeterRegistry registry) {
            return registry.timer(....);
        }
```
- ``io.micrometer.core.instrument.MeterRegistry``
```Java
        // 一个特殊的map
        private volatile PMap<Id, Meter> meterMap = HashTreePMap.empty();

        Timer timer(....) {
            return registerMeterIfNecessary(......);
        }

        private <M extends Meter> M registerMeterIfNecessary(....) {
            ....
            Meter m = getOrCreateMeter(....);
            ....
            return meterClass.cast(m);
        }

        private Meter getOrCreateMeter(....) {

            // 保存在内存中的指标
            Meter m = meterMap.get(mappedId);

            if (m == null) {
                ....
                synchronized (meterMapLock) {
                    m = meterMap.get(mappedId);

                    if (m == null) {
                        ....
                        // 没有找到创建后放入map
                        meterMap = meterMap.plus(mappedId, m);
                    }
                }
            }
        return m;
    }
```

### 尝试复现
> 因为我没有办法去不停的切换ip，就把方法的入参作为一个tag，但是结果是类似的
```Java
    sample.stop(Timer.builder("")
        .tag("exception", exceptionClass)
        .tag("className", className)
        .tag("simpleClassName", simpleClassName)
        .tag("method", methodName)
        .tag("args", args)
        .publishPercentileHistogram(false)
        .publishPercentiles(null)
        .register(registry));
```
>JVM设置最大堆内存2G
```
-XX:+PrintGC 
-Xmx2048m
```
结果：在不停的运行10分钟左右，项目不行了，因为是使用的nacos，项目被下线了，最后项目还活着，没有OOM，但是在不停的GC，已经对外不可用了。最后的直线下降是因为项目停了
![图一](/.images/2022-01-18-3.png)
![图二](/.images/2022-01-18-4.png)

## 解决方案
1. 限制指标数量
2. 清理内存中的指标

>上述两种解决方案都存在一定的弊端，限制数量可能会导致超过指标后的数据无法保存或者是打点方案的限制；定时清理会带来一定的性能损耗且因为配合使用的prometheus是定时从项目拉取数据，会导致一定时间内的数据被清理掉，导致数据不全。

### 限制指标数量
1. 修改打点方案，去掉调用方IP这个tag，这样其他几个tag数量相对来说都是可控的
2. 添加filter，在filter中记录指标的数量，然后根据需要去指定上限

### 清理内存中的指标
1. 定时重启服务，简单粗暴
2. 添加定时任务，通过调用``io.micrometer.core.instrument.MeterRegistry``的``clear``方法，定时清理全部的指标
3. 添加filter，在filter中保存所有的指标的id，到达上限后通过``io.micrometer.core.instrument.MeterRegistry``的``remove``方法清理指定的指标

### 代码
>下面是关于在filter中保存指标，超过指定数量后清除的方案

```java
@Slf4j
public class DubboMeterFilter implements MeterFilter {
    private final int maxMeters;
    /**
     * 用来存储所有的tag
     */
    private final Set<Meter.Id> ids;

    /**
     * 大于阈值后怎么处理数据
     */
    private final Consumer<Set<Meter.Id>> handler;

    public DubboMeterFilter(int maxMeters, Consumer<Set<Meter.Id>> handler) {
        this.maxMeters = maxMeters;
        this.handler = handler;
        this.ids = ConcurrentHashMap.newKeySet(maxMeters);
    }

    @Override
    public MeterFilterReply accept(Meter.Id id) {
        if (matchNameAndGetTagValue(id)) {
            // 如果大于设置的阈值，进行处理
            if (ids.size() > maxMeters) {
                try {
                    log.info("start handler Meters size: {}", ids.size());
                    this.handler.accept(ids);
                    log.info("end handler Meters");
                } catch (Throwable throwable) {
                    log.error("handler Meters fail, message: {}", throwable.getMessage(), throwable);
                }
            }
            ids.add(id);
        }

        return MeterFilterReply.NEUTRAL;
    }

    /**
     * 判断是否符合当前标签
     *
     * @param id
     * @return
     */
    private boolean matchNameAndGetTagValue(Meter.Id id) {
        return id.getName().startsWith("");
    }
```
```java
    public DubboMeterFilter clearDubboMeterFilter() {
        return new DubboMeterFilter(maxMeters, ids -> {
            for (Meter.Id id : ids) {
                registry.remove(id);
            }
            ids.clear();
        });
    }
```
### 效果
最后的那个直线下降是我在停下请求后手动GC的效果
![图一](/.images/2022-01-18-1.png)
![图二](/.images/2022-01-18-2.png)