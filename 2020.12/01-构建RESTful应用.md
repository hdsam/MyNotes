## 构建RESTful web应用

使用idea spring initializr创建springboot项目,如创建时无法联网，可以直接关闭系统防火墙即可

1. 配置POM文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>rest-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>rest-service</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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



2. 创建一个描述资源的实体类

```java
package com.example.restservice;

public class Greeting {
    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}

```

框架使用Jackson JSON将实体类对象转化为JSON数据，Jackson jar包包含在web starter中.

3. 创建资源的控制器Controller
 ```java
   package com.example.restservice;
   
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.concurrent.atomic.AtomicInteger;
   
   @RestController
   public class GreetingController {
   
       private static final String template = "Hello, %s";
       private final AtomicInteger counter = new AtomicInteger();
   
       @GetMapping("/greeting")
       public Greeting greeting(@RequestParam(value = "name", defaultValue = "World")String name){
           return new Greeting(counter.incrementAndGet(), String.format(template, name));
       }
   }
   
 ```

4. 启动项目成功后，并浏览器访问测试

![image-20201222121735029](https://raw.githubusercontent.com/hdsam/MyImages/master/MyNotes/20210305192616.png)

5. 重点总结

   - JSON数据渲染 ：实体Greeting实例被能自动转化被为JSON返回给浏览器，是由于Spring的HTTP message converter的支持，Jackson2就在应用的类路径中，对应起作用的转换类是`MappingJackson2HttpMessageConverter`.

   - `@SpringBootApplication` 注解等同于以下几个注解的组合：

     `@Configuration` : 标记一个类作为应用上下文的一个配置类

     `@EnableAutoConfiguration` : 告诉Spring Boot添加开始类路径下的bean、其他的bean、及环境配置。

     `@ComponentScan` : 告诉Spring扫描`com.example`包下的其他的组件，扫描的是启动类的当前级别的文件和下级的包。