# Spring-retry

## 使用

#### 添加依赖

> 如果是SpringBoot可以不加版本号

> ```xml
<dependency>
   <groupId>org.springframework.retry</groupId>
   <artifactId>spring-retry</artifactId>
   <version>1.2.5.RELEASE</version>
</dependency>
```



#### 单独使用

```java
public class RetryTest {

    @Test
    public void test() throws Throwable {
        RetryTemplate retryTemplate = new RetryTemplate();

        // 设置重试方案
        retryTemplate.setRetryPolicy(new SimpleRetryPolicy());

        // 设置退避策略
        retryTemplate.setBackOffPolicy(new NoBackOffPolicy());

        AtomicInteger count = new AtomicInteger();

        String res = retryTemplate.execute((RetryCallback<String, Throwable>) context -> {
            System.out.println("rrr");
            if (count.getAndIncrement() == 2) {
                return "res";
            } else {
                throw new RuntimeException("test");
            }

        }, context -> "null");


        System.out.println(res);

    }
}
```

#### 注解方式

#### 各种配置解释

##### 重试策略

- AlwaysRetryPolicy：允许无限重试，直到成功；也就是如果一直不成功的话就在这里卡死了
- CircuitBreakerRetryPolicy：
- CompositeRetryPolicy：组合重试策略，将其他的策略组合起来；有两种方案，乐观模式：只有一种允许重试就进行重试；悲观模式：只有一种不允许重试就不进行重试
- ExceptionClassifierRetryPolicy：
- ExpressionRetryPolicy：
- NeverRetryPolicy：只执行一次，不进行重试，跟不使用这个框架没啥区别。
- SimpleRetryPolicy：固定次数重试策略，默认最大重试次数为三次
- TimeoutRetryPolicy：超时重试策略

##### 退避策略

- ExponentialBackOffPolicy：指数退避策略，每次休眠时间为初始休眠时间（默认100毫秒）×乘数，直到最大休眠时间（默认30秒）
- ExponentialRandomBackOffPolicy：随机指数退避策略，与指数退避策略的区别为引入随机乘数产生每一次的休眠时间
- FixedBackOffPolicy：固定时间的退避策略，默认休眠一秒
- NoBackOffPolicy：没有退避策略，出问题立即重试
- UniformRandomBackOffPolicy：随机时间的退避策略，在最大休眠时间（默认1500毫秒）和最小休眠时间（默认500毫秒）直接随机产生一个值

## 原理
