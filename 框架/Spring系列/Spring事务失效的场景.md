> Spring事务失效主要有以下几种情况：

> - 数据库本身不支持事务
> - 数据源没有配置事务管理器
> - 主动设置不支持事务
> - 异常被catch了
> - 异常不是指定回滚的类型
> - 调用不是Spring的代理对象
> - 方法没有用public修饰



## 一、数据库本身不支持事务

数据库本身并不支持事务，没什么好说的，本身就不支持，怎么可能生效。

## 二、数据源没有配置事务管理器

如下是SpringBoot做的自动配置

```java
/**
 * {@link EnableAutoConfiguration Auto-configuration} for
 * {@link DataSourceTransactionManager}.
 *
 * @author Dave Syer
 * @author Stephane Nicoll
 * @author Andy Wilkinson
 * @author Kazuki Shimizu
 * @since 1.0.0
 */
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ JdbcTemplate.class, PlatformTransactionManager.class })
@AutoConfigureOrder(Ordered.LOWEST_PRECEDENCE)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceTransactionManagerAutoConfiguration {

   @Configuration(proxyBeanMethods = false)
   @ConditionalOnSingleCandidate(DataSource.class)
   static class DataSourceTransactionManagerConfiguration {

      @Bean
      @ConditionalOnMissingBean(PlatformTransactionManager.class)
      DataSourceTransactionManager transactionManager(DataSource dataSource,
            ObjectProvider<TransactionManagerCustomizers> transactionManagerCustomizers) {
         DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
         transactionManagerCustomizers.ifAvailable((customizers) -> customizers.customize(transactionManager));
         return transactionManager;
      }

   }

}
```

## 三、主动设置不支持事务

通过设置事务的隔离级别，主动设定不支持事务，自然事务失效。

```
@Override
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void test() {
    // ...
}
```

## 四、异常被catch了

Spring的事务需要有异常抛出才有可能会回滚。

有的项目中通过切面或者是其他的方式，设置了统一的异常处理，会catch掉所有的异常，异常不抛出，自然不存在事务回滚的情况。

## 五、异常不是指定回滚的类型

@Transactional注解中有设置在抛出何种异常的情况下进行回滚的设置，如果不进行指定的话，只有在抛出_RuntimeException_和_Error_这两种情况下才会回滚，所以如果不进行设置，在所有抛出可检查的异常的情况下均不会回滚。

所以最好在使用指定在抛出何种异常的时候才会回滚事务

```
@Override
@Transactional(rollbackFor = {Exception.class})
public void test() {
    // ...
}
```

## 六、调用不是Spring的代理对象

Spring的事务是通过切面实现的，如果方法的调用根本没有经过切面（当前对象没有通过Spring容器管理或通过this调用），事务自然也不会生效，在调用当前对象本身的事务方法的时候可以考虑使用如下方案：

```
@Service
public class HelloServiceImpl implements HelloService {
    @Autowired
    @Lazy
    private HelloService helloService;
    
    @Override
    public void hello() {
        helloService.test();
    }

    @Override
    @Transactional(rollbackFor = {Exception.class})
    public void test() {
        // ...
    }
}
```

## 七、方法没有用public修饰

在[Spring的官方文档](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html)中有对应的说明，意思就是@Transactional注解仅当方法为public类型的时候才生效，如果需要在其他情况下也生效，需要开启AspectJ代理模式。
