 # Spring框架的设计理念

[TOC]

## 1. Spring的设计理念

**Spring的核心骨架**是Spring context 、Spring bean、Spring core，而其他如AOP、Transaction、Orm、Web等则是建立在这些之上的扩展。bean是核心骨架中的最重要的部分，

Spring通过将对象的关系用配置文件来建立，关系的建立则是由Ioc容器来管理。

Spring将对象包装成bean，context则是bean的集合，而core则是让这些bean之间建立其了联系。



## 2. Spring的核心组件详解

### 2.1 Bean组件

Bean组件位于`org.springframework.beans`包下。这个包主要负责了Bean的定义、Bean的创建和对Bean的解析。对使用者来说，需要关注的是Bean的创建。

`BeanFactory`负责了Bean的创建

`BeanDefinition`对应配置文件中`<bean/>`标签及包括的子标签，是对Bean的定义。



### 2.2 Context组件

Context组件位于`org.springframework.context`包下。Context的作用就是给Bean提供了一个运行时的环境。

`ApplicationContext`的子类主要包括两方面：

- `ConfigurableApplicationContext`：表示该Context是可修改的，用户可以动态添加或修改已有的配置信息，其子类中最常用的是`AbstractRefreshableApplicationContext`，这个类表示是一种可以更新的Context。
- `WebApplicationContext`：用于web环境的Context，可以访问`ServletContext`。

Context的作用：

* 标识一个应用环境
* 利用BeanFactory来创建Bean对象
* 保存对象关系
* 捕获各种事件

下面取出了一些ApplicationContext的相关的类来描述了继承关系，并不全面，仅供参考：

![image-20210325015422695](C:\Users\Yegenyao\AppData\Roaming\Typora\typora-user-images\image-20210325015422695.png)



### 2.3 Core组件

定义了资源的访问方式。如其中的`Resource`接口定义了

