## Maven的入门使用

> 主要介绍maven的一些原理相关 

### 1. 创建MAVEN项目

在cmd中执行以下命令可以创建一个maven项目

```shell
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```



### 2. 编译MAVEN项目

执行项目的打包：

```shell
mvn package
```

### 3. 运行打包的项目

``` shell
java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
```

