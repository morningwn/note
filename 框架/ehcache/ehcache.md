# Ehcache的使用

这里着重说一下3.x的用法，至于2.x只说一下咋配置。

## 2.x

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```

### 配置文件

需要在properties文件中指定对应的ehcache的配置文件。

以下是配置文件的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
    <defaultCache
            maxEntriesLocalHeap ="10000"
        eternal ="false"
        timeToIdleSeconds ="120"
        timeToLiveSeconds ="120"
        maxEntriesLocalDisk ="10000000"
        diskExpiryThreadIntervalSeconds ="120"
        memoryStoreEvictionPolicy = "LRU">
        <persistence strategy ="localTempSwap" />
    </defaultCache>
</ehcache>
```

### 代码

可以直接使用Spring框架的注解进行使用，也可以使用工具类，获取到对应的Cache来手动维护数据。

下面是工具类的代码：

```java
public class CacheUtil {
    @Autowired
    private CacheManager cacheManager;

    public CacheUtil(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }
    
    public void put(String cacheName, String key, String value) {
        Cache cache = cacheManager.getCache(cacheName);

        assert cache != null;
        cache.put(key, value);
    }
    
    public String get(String cacheName, String key) {
        Cache cache = cacheManager.getCache(cacheName);

        assert cache != null;

        return cache.get(key, String.class);

    }
    
}
```



## 3.x

### 依赖

Spring Boot使用3.x版本的形式发生了变化，原本是使用的ehcache的manager，在修改之后使用的是JSR107统一的manager，也需要额外添加一个依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
</dependency>
```

### 配置

同样，这个也需要在properties文件中指定配置文件的地址。

以下是一个简单的配置

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://www.ehcache.org/v3"
        xmlns:jsr107="http://www.ehcache.org/v3/jsr107"
        xsi:schemaLocation="
      http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core-3.0.xsd
      http://www.ehcache.org/v3/jsr107 http://www.ehcache.org/schema/ehcache-107-ext-3.0.xsd">
    <service>
        <jsr107:defaults enable-statistics="true"/>
    </service>

    <cache alias="demo">
        <key-type>java.lang.String</key-type>
        <value-type>java.lang.String</value-type>
        <heap unit="entries">100</heap>
    </cache>
</config>
```



### 配置文件

以下是我整理的配置文件的部分解释，详细的高级用法去官网

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://www.ehcache.org/v3"
        xmlns:jsr107="http://www.ehcache.org/v3/jsr107"
        xsi:schemaLocation="
      http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core-3.0.xsd
      http://www.ehcache.org/v3/jsr107 http://www.ehcache.org/schema/ehcache-107-ext-3.0.xsd">

    <service>
        <!-- 使用jsr的服务扩展-->
        <!-- default-template：是否使用-->
        <!-- jsr-107-compliant-atomics：是否使用-->
        <!-- enable-management：是否使用管理MBean-->
        <!-- enable-statistics：是否使用统计MBean-->
        <jsr107:defaults default-template="" jsr-107-compliant-atomics="false" enable-management="false" enable-statistics="true">
            <jsr107:cache name="" template=""/>
        </jsr107:defaults>
    </service>

    <!-- 配置线程池，注意，这个没有默认值-->
    <!-- alias：线程池的名称，后续引用的时候通过此名称引用-->
    <thread-pools>
        <thread-pool alias="" min-size="" max-size=""/>
    </thread-pools>
    <!-- 接下来用于发送事件的默认线程池-->
    <event-dispatch thread-pool=""/>
    <!--    配置接下来用于写工作的默认线程池-->
    <write-behind thread-pool=""/>
    <!--    配置用于磁盘存储的默认线程池-->
    <disk-store thread-pool=""/>

    <!-- 指定全局的对象复制方法-->
    <!-- 可以自定义对象复制器，需要实现接口Copier<T>-->
    <default-copiers>
        <copier type="">com.example.ehcachedemobase3.config.MyCopier</copier>
    </default-copiers>

    <!-- 指定全局的序列化方法，如果缓存中没有设置，使用此处的设置-->
    <!-- 默认情况下，支持自动化的处理，也可以自己实现序列化器，需要实现接口Serializer-->
    <default-serializers>
        <serializer type="">com.example.ehcachedemobase3.config.MySerializer</serializer>
    </default-serializers>

    <!-- 全局的堆存储设置-->
    <heap-store>
        <!-- 设置移动对象的时候需要遍历的最大对象数，默认是1000-->
        <max-object-graph-size>10</max-object-graph-size>
        <!-- 设置单个对象的最大大小， 默认是Long.MAX_VALUE-->
        <max-object-size unit="B">10</max-object-size>
    </heap-store>

    <!-- 指定持久化数据存储的位置-->
    <persistence directory=""/>

    <!-- 缓存配置的模板，可以在cache标签中使用属性uses-template引用-->
    <!-- 这个里面的标签，和cache的相同-->
    <cache-template name="simpleCache">
        <heap unit="entries">100</heap>
    </cache-template>


    <!-- 一个cache标签对应一个cache对象-->
    <!-- alias对应cacheName，可以通过这个属性的值获取到对应的cache-->
    <!-- uses-template属性对应模板的名称，可以将通用的属性抽取出来，创建一个cache-template-->
    <cache alias="demo" uses-template="simpleCache">
        <!-- 对应的键值对的类型，如果没有指定，默认为Object-->
        <!-- copier：指定使用的对象复制器，如果没指定，使用全局设置，值为对象拷贝器的全类名-->
        <!-- serializer：指定使用的序列化器，如果没有指定，使用全局设置的，值为序列化器的全类名-->
        <key-type copier="" serializer="">java.lang.String</key-type>
        <value-type copier="" serializer="">java.lang.String</value-type>

        <!-- 为磁盘存储指定特定的线程池-->
        <disk-store-settings disk-segments="16" thread-pool="" writer-concurrency="1"/>

        <!-- 设置到期类型及其参数-->
        <expiry>
            <!-- 指定自定义的过期方案，需要实现ExpiryPolicy接口-->
            <class>com.example.ehcachedemobase3.config.MyExpiryPolicy</class>
            <!-- 设置缓存永远不会到期-->
            <none/>
            <!-- 设置缓存生存时间-->
            <ttl unit="days">10</ttl>
            <!-- 设置缓存空闲时间-->
            <tti unit="days">10</tti>
        </expiry>

        <!-- 指定堆内可以存放多少数据-->
        <!-- unit属性代表单位，其中，entries指数据条数-->
        <heap unit="entries">10</heap>

        <!-- 堆存储的设置-->
        <heap-store-settings>
            <!-- 设置移动对象的时候需要遍历的最大对象数，默认是1000-->
            <max-object-graph-size>10</max-object-graph-size>
            <!-- 设置单个对象的最大大小， 默认是Long.MAX_VALUE-->
            <max-object-size unit="B">10</max-object-size>
        </heap-store-settings>

        <!-- 为缓存配置侦听器，该类需要实现接口CacheEventListener-->
        <!-- dispatcher-thread-pool:为当前缓存配置特定的线程池，用于发送事件-->
        <listeners  dispatcher-thread-pool="">com.example.ehcachedemobase3.config.MyCacheEventListener</listeners>

        <loader-writer>
            <!-- 对应处理方法的全类名，需要实现接口CacheLoaderWriter-->
            <class>com.example.ehcachedemobase3.config.MyCacheLoaderWriter</class>

            <!-- 配置后写-->
            <!-- thread-pool：为当前缓存配置特定的后写线程池-->
            <!-- concurrency：并发级别，指示将有多少个并行处理线程队列可以写在后面。实际上正在写的最大操作数是并发级别*队列大小-->
            <!-- size：队列大小，指示在对缓存操作之前可以有多少个挂起的写操作-->
            <write-behind thread-pool="" concurrency="" size="">
                <!-- batch-size：批次大小-->
                <!-- coalesce：合并-->
                <batching batch-size="" coalesce="true">
                    <!-- 最大写入延迟-->
                    <max-write-delay unit="days">1</max-write-delay>
                </batching>

                <non-batching/>
            </write-behind>
        </loader-writer>

        <!-- 配置在缓存的基础存储发生故障时使用的弹性策略-->
        <resilience></resilience>

        <resources>
            <!-- 指定此缓存在堆内存储的数据数量-->
            <heap unit="B">10</heap>
            <!-- 为了减轻JVM垃圾回收的压力，将部分数据存在堆外，但是仍在JVM的内存里面-->
            <!-- 指定此缓存在堆外存储数据的大小-->
            <offheap unit="B">19</offheap>
            <!-- 指定此缓存在磁盘上持久化存储数据的大小-->
            <disk unit="B" persistent="false"/>
        </resources>

        <!-- 指定当前缓存下的jsr107扩展服务配置-->
        <jsr107:mbeans enable-management="false" enable-statistics="false"/>

        <eviction-advisor>
        </eviction-advisor>
    </cache>

</config>
```