# SpringBoot教程

[TOC]



## 补充内容：Bean的生命周期

- Spring容器创建时：

1. 实例化
2. 初始化属性。
3. 若实现了`BeanNameAware`接口，则执行其`setBeanName()`方法，设置bean工厂创建这个bean的名字。
4. 若使用了``@PostConstruct``注解标注了某个方法，则会执行这个方法。
5. 若实现了`InitializingBean`接口，则会执行这个接口的`afterPropertiesSet()`方法。
6. 
7. 若使用了`@preDestory`注解标注了某个方法，则会执行这个方法。
8. 若实现了`DisposableBean`j接口，则会执行这个接口的`destroy()`方法。





## 一、SpringBoot应用的启动及退出

### 1. 应用启动

通常在控制台可以通过java命令并加上`--debug`参数启动，即可以调试模式查看更详细的日志信息：

```shell
java -jar xyz.jar --debug
```

### 2.Bean的懒加载

spring应用会在启动时就创建bean，若要开启懒加载，可以在application.properties中添加如下配置：

```properties
spring.main.lazy-initialization=true
```

应用的bean会根据需要来配置创建。但是可能会导致应用发生问题，**通常不建议开启**。此外`@Lazy`注解可以作用于某个bean上，用于配置该bean的懒加载模式。

### 3. 定制化SpringBoot启动的Banner

项目的resources目录下创建banner.txt文件，然后将任意文本内容复制进去，会替换默认的banner，启动时即可看到输出。

### 4.定制化SpringBoot应用

通常我们都是通过`SpringApplication.run(...)`方法来启动应用，这是一个静态方法。然而，我们也可以通过实例化一个`SpringApplicaton`的实例来运行一个SpringBoot应用，并进行定制化的配置，如：

```java
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
//		SpringApplication.run(Application.class, args);

		SpringApplication application = new SpringApplication(Application.class);
		application.setBannerMode(Banner.Mode.OFF);
		application.run(args);
	}
}
```

### 5. 流式创建者模式API

采用`SpringApplicationBuilder`的流式API可以更方便的创建具有上下文关系的SpringApplication实例，这里有待了解，简单的实例如下。

```java
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
//		SpringApplication.run(Application.class, args);

//		SpringApplication application = new SpringApplication(Application.class);
//		application.setBannerMode(Banner.Mode.OFF);
//		application.run(args);
		
		new SpringApplicationBuilder(Application.class)
			.bannerMode(Banner.Mode.CONSOLE)
			.run(args);
			
	}
}
```

### 6. 应用可用性

可用性表示当前的应用的状态可以是可以被其他应用感知。Spring的可用性分为两个方面：

- 存活状态：应用程序的“活动”状态告诉它的内部状态是否允许它正确工作，或者如果它当前出现故障，它是否可以自行恢复。

  损坏的“活动”状态意味着应用程序处于无法恢复的状态，基础设施应该重新启动应用程序。狭义地来看，在springboot应用中，`ApplicationContext`正确启动了，就可以认为应用是处于有效的状态。

- 就绪状态：应用程序的“准备就绪”状态告诉应用程序是否准备好处理通信。失败的“准备就绪”状态告诉平台，它暂时不应该将通信路由到应用程序。通常发生在启动期间，当`CommandLineRunner`和`ApplicationRunner`组件正在被处理时，或者在应用程序认为它太忙而无法处理额外流量时的任何时候。

具体的章节请参考官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-availability

### 7. 应用的事件和监听器

具体的章节请参考官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-web-environment

### 8. Web应用的环境

SpringApplicaton会基于当前应用的依赖，来创建对应的`ApplicationConext`：

1. 如果SpringMVC存在，则会使用`AnnotationConfigServletWebServerApplicationContext`;
2. 如果SpringMVC不存在单Spring WebFlux存在，则会使用`AnnotationConfigReactiveWebServerApplicationContext`;
3. 以上都不满足，则会使用`AnnotationConfigApplicationContext`。

如当我们使用了springMVC时

pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.2</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>org.ygy.study</groupId>
	<artifactId>springboot-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot-demo</name>
	<packaging>jar</packaging>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
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



![image-20210202153349557](C:\Users\Yegenyao\AppData\Roaming\Typora\typora-user-images\image-20210202153349557.png)

### 9. 访问应用的参数

