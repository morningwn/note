# 开始



## 方法一

创建项目的时候可以先创建maven项目，然后添加各种坐标，创建文件，文件夹啥的

##  方法二

如果是用的专业版的IDEA，可以直接用他自带的springBoot创建器，创建项目

这样会根据你自己的选择去添加坐标，然后默认创建各种文件夹。

## 方法三

使用spring官方提供的创建器来创建，效果和上面那个差不多，就是需要联网，这个是那个网站的地址：[传送门](https://start.spring.io/)

选择以下选项：

1.  Project类型是：Maven Project
2. Spring Boot版本：根据需要选择
3. 然后填上包名、团队名称、项目名称等等
4. 最后选择项目的打包方式，Spring Boot基本上都是jar吧
5. 选择jdk版本
6. 另外就是可以额外添加各种坐标，如果不选择的话，默认只有一个Spring Boot启动器和测试模块，别的都没
7. 选好后 ，下载，然后导入到自己的IDE中



# 启动项目

将上面下载好的项目导入到IDE中，直接找到{项目名称}Appliction.java文件，运行他，然后就行了；但是因为这个是一个空项目，点击运行之后马上就结束了。

下面这个是我的项目的pom文件的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
        
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

## 添加web模块

想要把这个项目变成web项目，需要添加Spring Boot的web模块

添加下面的坐标

```xml
		<!-- web模块-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

然后，写一个类，添加上@Controller或者是@RestController注解，搞一个方法，同样写上注解@RequestMapping

下面是我的代码

~~~java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello word";
    }

}
~~~

## 启动

还是运行一开始说的那个类，然后访问对应的地址；如果没有问题的话，像我这个就能能够看到Hello word了。



