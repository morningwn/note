# Redis

## 五大数据类型



## Spring Boot中使用Redis



### 直接使用Spring Cache

比较简单的一种用法是添加redis对应的start，springBoot会帮我们直接配置好

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

下面是简单的配置，配置了主机、端口、密码，其他的可以看对应的配置文件``RedisProperties``

```properties
spring.redis.host=
spring.redis.port=
spring.redis.password=
```

使用的时候需要使用``@EnableCaching``注解开启缓存

然后在对应的类上面添加注解：``@CacheConfig(cacheNames = "user")``

想要使用缓存的方法上面添加注解：``@Cacheable``这样就可以了。

需要注意的是：对应的实体类需要实现``Serializable``接口，不然会报错。

另外上面这种方式可以自己创建方法，定义方法进行各种的操作；具体的获取cache是通过cacheManage对象来获取的。

### 使用jedis创建工具类

需要的依赖：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.4.1</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```

这种方法需要手动创建JedisPool对象，并放入容器。

代码如下：

```java
package com.xiaomi.newretail.demo.config;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import redis.clients.jedis.JedisPool;

/**
 * @author 
 * @version 1.0.0
 * @date 2021年01月04日 14:14
 */
@Configuration
public class MyRedisConfig {

    @Value("${spring.redis.host}")
    private String host;
    @Value("${spring.redis.port}")
    private int port;
    @Value("${spring.redis.password}")
    private String passWord;

    @Bean
    public JedisPool jedisPool(GenericObjectPoolConfig<JedisPool> poolConfig) {
        return new JedisPool(poolConfig, host, port, 10000, passWord);
    }

    @Bean
    public GenericObjectPoolConfig<JedisPool> genericObjectPoolConfig() {
        GenericObjectPoolConfig<JedisPool> poolConfig = new GenericObjectPoolConfig<>();

        return poolConfig;
    }

}
```

接下来据需要创建对应的工具类：

```java
package com.xiaomi.newretail.demo.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.netty.util.internal.StringUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

/**
 * @author 
 * @version 1.0.0
 * @date 2021年01月04日 10:57
 */
@Component
public class RedisUtil {
    @Autowired
    private JedisPool jedisPool;
    @Autowired
    private ObjectMapper objectMapper;

    public void put(String key, Object value) {
        Jedis jedis = jedisPool.getResource();

        try {
            jedis.set(key, objectMapper.writeValueAsString(value));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }finally {
            jedis.close();
        }
    }

    public Object get(String key, Class<?> type) {

        Jedis jedis = jedisPool.getResource();

        String value = jedis.get(key);
        if (StringUtil.isNullOrEmpty(value)) { return null; }

        Object object = null;
        try {
            object = objectMapper.readValue(value, type);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }finally {
            jedis.close();
        }

        return object;
    }
}
```

通过工具类实现各种操作。