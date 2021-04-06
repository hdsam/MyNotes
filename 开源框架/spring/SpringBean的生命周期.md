# Spring中Bean的生命周期



下面以一个传统的spring项目为例，总结下Spring的生命周期：

1. `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-ioc</artifactId>
        <groupId>org.ygy.springioc</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>chapter02</artifactId>

    <name>chapter02</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <!--用来支持@PostConstruct和@PreDestroy注解-->
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.1.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.3.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-project-info-reports-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>

```



2.  创建Bean实体类，并实现生命周期相关的接口

   ```java
   package org.ygy.springioc;
   
   import org.springframework.beans.BeansException;
   import org.springframework.beans.factory.*;
   import org.springframework.beans.factory.config.BeanPostProcessor;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.ApplicationContextAware;
   
   import javax.annotation.PostConstruct;
   import javax.annotation.PreDestroy;
   
   /**
    * the bean is instantiated (if it is not a pre-instantiated singleton),
    * its dependencies are set, and the relevant lifecycle methods
    * (such as a configured init method or the InitializingBean callback method) are invoked.
    *
    * @Author Yegenyao
    * @Date 2021/1/29 23:38
    */
   public class Bean implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, BeanPostProcessor, InitializingBean, DisposableBean {
   
       private String name;
   
       private int value;
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public int getValue() {
           return value;
       }
   
       public void setValue(int value) {
           this.value = value;
       }
   
       public Bean(String name, int value) {
           this.name = name;
           this.value = value;
           System.out.println("Bean.new");
       }
   
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("Bean.BeanPostProcessor.postProcessBeforeInitialization");
           return bean;
       }
   
       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("Bean.BeanPostProcessor.postProcessAfterInitialization");
           return bean;
       }
   
       @Override
       public void setBeanName(String s) {
           System.out.println("Bean.BeanNameAware.setBeanName");
       }
   
       @Override
       public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
           System.out.println("Bean.BeanFactoryAware.setBeanFactory");
       }
   
       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           System.out.println("Bean.ApplicationContextAware.setApplicationContext");
       }
   
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("Bean.InitializingBean.afterPropertiesSet");
       }
   
       @Override
       public void destroy() throws Exception {
           System.out.println("Bean.DisposableBean.destroy");
       }
   
       public void myInit() {
           System.out.println("Bean.myInit");
       }
   
       public void myDestroy() {
           System.out.println("Bean.myDestroy");
       }
   
       @PreDestroy
       public void preDestroy() {
           System.out.println("Bean.preDestroy");
       }
   
       @PostConstruct
       public void postConstruct() {
           System.out.println("Bean.postConstruct");
       }
   }
   
   ```

3. 将bean加入ioc容器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    

    <bean class="org.ygy.springioc.Bean" id="bean" init-method="myInit" destroy-method="myDestroy">
        <constructor-arg name="name" value="abc"></constructor-arg>
        <constructor-arg name="value" value="123"></constructor-arg>
    </bean>


</beans>
```

4. 编写测试类：

```java
package org.ygy.springioc;

import org.junit.Test;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Unit test for simple App.
 */
public class AppTest {
    /**
     * Rigorous Test :-)
     */
    @Test
    public void testBeanLifeCycle() {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        Bean bean = applicationContext.getBean("bean",Bean.class);
        System.out.println(bean.getName());
        applicationContext.close();
    }
}

```

5. 执行观察结果

```java
Bean.new
Bean.BeanNameAware.setBeanName
Bean.BeanFactoryAware.setBeanFactory
Bean.ApplicationContextAware.setApplicationContext
Bean.InitializingBean.afterPropertiesSet
Bean.myInit
abc
Bean.DisposableBean.destroy
Bean.myDestroy
```

6. 以流程图呈现：

   ![image-20210323011610633](https://raw.githubusercontent.com/hdsam/MyImages/master/MyNotes/20210323143919.png)