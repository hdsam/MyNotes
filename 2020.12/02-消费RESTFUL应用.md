# 消费RESTful应用

创建一个消费RESTful服务的应用。

1. 创建一个SpringBoot项目，配置项目的POM	

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.1.7.RELEASE</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
       <groupId>com.example</groupId>
       <artifactId>consuming-rest</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <name>consuming-rest</name>
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

2. 定义两个绑定数据的实体类

   ```java
   package com.example.consumingrest;
   
   import com.fasterxml.jackson.annotation.JsonIgnore;
   import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
   
   @JsonIgnoreProperties(ignoreUnknown = true)
   public class Quote {
   
       private String type;
       private Value value;
   
       public Quote() {
       }
   
       public String getType() {
           return type;
       }
   
       public void setType(String type) {
           this.type = type;
       }
   
       public Value getValue() {
           return value;
       }
   
       public void setValue(Value value) {
           this.value = value;
       }
   
       @Override
       public String toString() {
           return "Quote{" +
                   "type='" + type + '\'' +
                   ", value=" + value +
                   '}';
       }
   }
   ```

   ```java
   package com.example.consumingrest;
   
   import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
   
   @JsonIgnoreProperties(ignoreUnknown = true)
   public class Value {
       private Long id;
       private String quote;
   
       public Value() {
       }
   
       public Long getId() {
           return id;
       }
   
       public void setId(Long id) {
           this.id = id;
       }
   
       public String getQuote() {
           return quote;
       }
   
       public void setQuote(String quote) {
           this.quote = quote;
       }
   
       @Override
       public String toString() {
           return "Value{" +
                   "id=" + id +
                   ", quote='" + quote + '\'' +
                   '}';
       }
   }
   ```

   `@JsonIgnoreProperties(ignoreUnknown = true)`：表明在序列化或者反序列化中，需要忽略掉一些无法绑定的属性。此外`@JsonProperty`能够处理JSON数据中心字段和类定义不一样的属性，如果字段名是一样的，这个属性也可以忽略掉。

3.创建Springboot启动类

```java
package com.example.consumingrest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumingRestApplication {

    private final static Logger log = LoggerFactory.getLogger(ConsumingRestApplication.class);

    public static void main(String[] args) {
        SpringApplication.run(ConsumingRestApplication.class, args);
    }

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder){
        return builder.build();
    }

    @Bean
    public CommandLineRunner run(RestTemplate restTemplate){
        return args -> {
            Quote quote = restTemplate.getForObject("https://gturnquist-quoters.cfapps.io/api/random",
                    Quote.class);
            log.info(quote.toString());
        };
    }
}

```



4. 要点
   - `RestTemplate`采用Jackson JSON库自动处理接收的数据。
   - `@SpringBootApplication`注解包含了`@Configuration`注解，所以`@Bean`注解能起作用，同时CommandLineRunner会在应用启动时运行。