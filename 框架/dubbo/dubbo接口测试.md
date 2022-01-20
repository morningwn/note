# dubbo接口压测

使用jmeter来进行dubbo接口的压测

有两种方法，一种是自己去写Java代码，封装各种请求和参数，最后使用jmeter发送Java请求的方式来实现。第二种是使用大佬提供的Jmeter压测插件来进行。

## 发送Java请求的方式

首先需要在项目中添加两个依赖：

这两个依赖是Jmeter的，不算其他的依赖

```xml
<dependency>
    <groupId>org.apache.jmeter</groupId>
    <artifactId>ApacheJMeter_core</artifactId>
    <version>5.4</version>
</dependency>
<dependency>
    <groupId>org.apache.jmeter</groupId>
    <artifactId>ApacheJMeter_java</artifactId>
    <version>5.4</version>
</dependency>
```

之后创建一个类继承```AbstractJavaSamplerClient```这个抽象类，然后实现对应方法即可，这个是发送Java请求所需的代码。

在Jmeter中使用的话，需要将项目打包，放在Jmeter安装目录下的lib\ext下面（需要注意的是，需要将项目所有的依赖都放在jar包里面），直接选择--Java请求，然后填上对应的类的包名+类名，即可。

其他的用法可以参考```org.apache.jmeter.protocol.java.test.JavaTest```，还可以自己设置发送请求的时候需要传入什么参数。

然后就是对upc项目进行压测，因为存在验签，所以需要对发送的请求加签，添加一个filter即可。需要注意的是，在打包的时候存在文件会被覆盖，导致filter不工作，加签失败。解决方案就是在项目打包完成之后，手动替换对应的配置。



## Jmeter插件

原理和上面的类似，也在项目中添加一个filter即可。





















