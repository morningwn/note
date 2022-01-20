# Dubbo

## Zookeeper注册中心下的demo

使用zookeeper作为注册中心，需要一个安装了zookeeper的主机，我这里在本机的docker里面跑了一个zookeeper，就不说怎么安装了。

### 依赖

在这里我把版本信息都放在了父项目里面处理，然后不管是服务提供方还是消费方，他们的依赖都是相同，版本信息如下：

```xml
<properties>
  <maven.compiler.source>8</maven.compiler.source>
  <maven.compiler.target>8</maven.compiler.target>
  <spring.version>5.3.2</spring.version>
  <dubbbo.version>2.7.7</dubbbo.version>
  <lombok.version>1.18.4</lombok.version>
  <slf4j.version>1.7.25</slf4j.version>
</properties>
```

这个是服务提供者的依赖，其中```dubbo-demo-interface```是抽象出来的接口

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-demo</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbo-provider</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>dubbo-demo-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-registry-zookeeper</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-configcenter-zookeeper</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-metadata-report-zookeeper</artifactId>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 配置

配置非常简单，配置好注册中心的地址，使用的协议，就行了，其他的就是暴露接口这些。

#### provider

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <dubbo:application name="dubbo-demo-provider">
    </dubbo:application>

    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>

    <dubbo:protocol name="dubbo" port="-1"/>

    <bean id="demoInterface" class="org.example.dubboprovider.service.DemoInterfaceImpl"/>

    <dubbo:service interface="org.example.dubbodemointerface.DemoInterface" timeout="30000" ref="demoInterface" group="development"/>
    
</beans>
```

#### consumer

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="dubbo-demo-consumer">
    </dubbo:application>

    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>

    <dubbo:protocol name="dubbo" port="-1"/>

    <dubbo:reference id="demoInterface" check="false" interface="org.example.dubbodemointerface.DemoInterface" group="development"/>
</beans>
```

### 启动代码

#### provider

因为提供方需要等待消费者的请求，所以最后加了个```System.in.read();```，让程序阻塞，不然直接就关闭了。

换成其他的也是可以的。

```java
package org.example.dubboprovider;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * @author 
 * @version 1.0.0
 * @date 2021年01月20日 10:51
 */
public class ProviderMain {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

        System.out.println("=============ProviderMain启动了============");

        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### consumer

```java
package org.example.dubboconsumer;

import org.example.dubboconsumer.service.MyService;
import org.example.dubbodemointerface.DemoInterface;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 
 * @version 1.0.0
 * @date 2021年01月20日 10:58
 */
public class ConsumerMain {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

        System.out.println("=============ConsumerMain启动了============");

        DemoInterface bean = applicationContext.getBean(DemoInterface.class);

        String res = bean.sayHello("张三");

        System.out.println(res);

    }
}
```

然后先启动provider，再启动consumer就能看到结果了



## Filter

dubbo提供了非常简单的filter的实现方法

具体有两步：

1. 创建一个类，实现接口```org.apache.dubbo.rpc.Filter```
2. 在resources下创建文本文件META-INF/dubbo/internal/org.apache.dubbo.rpc.Filter，在里面配置一下自己的Filter即可

具体的配置格式如下：

```
myFilter=org.example.dubboprovider.filter.ProviderFilter
```

类似k-v键值对，等号前面的名字没有要求，后面的是类的路径

### 创建类

group指定在什么地方生效，provider是提供放，consumer是消费方

order是指定filter的顺序，越小优先级越高

```java
package org.example.dubboprovider.filter;

import org.apache.dubbo.common.extension.Activate;
import org.apache.dubbo.rpc.*;

/**
 * @author 
 * @version 1.0.0
 * @date 2021年01月20日 11:38
 */
@Activate(group = {"provider"}, order = -10000)
public class ProviderFilter implements Filter {
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {

        System.out.println("filter生效了");

        System.out.println(invoker.getUrl());

        return invoker.invoke(invocation);
    }
}
```

## 5、Dubbo启停原理解析





































