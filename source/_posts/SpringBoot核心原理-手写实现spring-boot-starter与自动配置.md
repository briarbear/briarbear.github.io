---
title: SpringBoot核心原理-手写实现spring-boot-starter与自动配置
date: 2018-07-24 19:51:55
toc: true
tags: SpringBoot
categories: Spring
---

> 主要内容摘自《Java EE的颠覆者 SpringBoot实战-汪云飞》，以下为学习笔记与实现代码，仅供交流学习

## 1. SpringBoot实现原理

&nbsp;&nbsp;`Springboot`只所以能实现快速开发，其中最重要的特征就是`spring-boot-starter`与自动配置，前者将常用的依赖分组进行整合，将其合并到一个依赖中；后者则是利用了`Spring4.x+`对条件化配置的支持，合理地推测应用所需的`bean`并自动配置他们，这些则符合`springboot`的设计理念-**习惯优于配置**，从而大大减少繁多的配置，提高开发效率

<!--more-->

## 2. 实现步骤

### 1. IDEA新建maven项目

![](http://cdn.briarbear.cn/201807241814_743.png)

![](http://cdn.briarbear.cn/201807241820_966.png)

### 2. 修改pom.xml文件

```xml
<groupId>com.xzp</groupId>
    <artifactId>spring-boot-starter-hello</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>


    <name>spring-boot-starter-hello</name>
    <url>http://maven.apache.org</url>


    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.0.3.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

### 3. 新建配置属性类

- maven创建项目时，如果相关的目录，则可以手动建立
- `HelloServiceProperties.java`

```java
@ConfigurationProperties(prefix = "hello")
public class HelloServiceProperties {

    /**
     * 默认的msg是world
     * 可以通过在application.properties中hello.msg= 来设置
     */
    private static final String MSG = "world";
    private String msg = MSG;

    public static String getMSG() {
        return MSG;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

```

### 4. 判断服务类

- 该示例中通过判断此类的存在与否来创建这个类的Bean，这个类可以是第三方类库的类

- `HelloService.java`

```java
public class HelloService {
    private String msg;

    public String sayHello(){
        return "Hello " + msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

### 5. 自动配置类

- `HelloServiceAutoConfiguration.java`
- 根据 HelloServiceProperties 提供的参数，并通过＠ConditionalOnClass 判断 HelloService 这 个类在类路径中是否存在，且当容器中没有这个 Bean 的情况下自动配置这个 Bean.    

```java
@Configuration
@EnableConfigurationProperties(HelloServiceProperties.class)  // 1. 开启属性注入
@ConditionalOnClass(HelloService.class)   // 2. 当HelloService在类路径下
@ConditionalOnProperty(prefix = "hello",value = "enabled",matchIfMissing = true)  //3. 配置的前缀: hello 当设置hello=enabled的情况下，如果没有设置，则默认true，即条件符合
public class HelloServiceAutoConfiguration {

    @Autowired
    private HelloServiceProperties helloServiceProperties;

    @Bean  // 4. 使用Java config配置
    @ConditionalOnMissingBean(HelloService.class)   //判断HelloService这个类在类路径中是否存在，且当容器中没有这个Bean的情况下自动配置这个Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        helloService.setMsg(helloServiceProperties.getMsg());
        return helloService;
    }
}
```

### 6. 注册配置

- 若想配置生效，需要注册自动配置类。在`src/main/resources`下新建`META-INF/spring.factories,`，项目目录结构如下所示：

  ![](http://cdn.briarbear.cn/201807241913_970.png)

- `spring.factories`内容如下：

```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.xzp.spring_boot_starter_hello.HelloServiceAutoConfiguration
```

### 7. 安装到本地库

- 执行命令：`mvn install`安装到本地库，或者将这个jar包发布到Maven私服上

```shell
tarter-hello-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.500 s
[INFO] Finished at: 2018-07-24T19:16:56+08:00
[INFO] ------------------------------------------------------------------------

```

### 8. 新建测试项目 

- 新建SpringBoot项目，本文示例为`ch6_5`项目，对应于书中的章节，
- 新建测试的`Controller`，

```java
@RestController
public class HelloController {

    @Autowired
    HelloService helloService;

    @RequestMapping("/")
    public String index(){
        return helloService.sayHello();
    }

}

```

- 目录结果如下：

![](http://cdn.briarbear.cn/201807251917_722.png)

- 启动`SpringBoot`，此时`application.properties`中未添加任何内容,浏览器中输入 `localhost:8080`,结果如下：

![](http://cdn.briarbear.cn/201807241924_515.png)

- 上述结果表明默认的配置生效了，测试我们修改application.properties内容如下，再重新启动

  ```shell
  hello.msg=xiongzp08
  ```

- 则运行结果如下：

![](http://cdn.briarbear.cn/201807251916_569.png)

- 可以看到相应的结果发生变化了

- 如果在`application.properties`中添加`debuge=true`，还可以在控制台看到我们的配置生效的结果

  ![](http://cdn.briarbear.cn/201807241930_197.png)

---

## 3. 相关连接

- 本示例完整代码 
  - [码云](https://gitee.com/briarbear/springboot_in_action_wyf/blob/master/springbootstarterhello)
  - [Github](https://github.com/briarbear/springboot_in_action_wyf/blob/master/springbootstarterhello)