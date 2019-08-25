---
title: Spring-AOP核心原理及手写实现
date: 2018-07-03 18:43:21
toc: true
tags: AOP
categories: Spring
---

## 1. 基本原理

> Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于方法拦截（AspectJ静态代理框架能够实现更为高级的功能，本文不做介绍）
>
> Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法

Spring AOP中的动态代理主要有两种方式 :

1. **JDK动态代理**：JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是`InvocationHandler`接口和`Proxy`类
2. **CGLIB动态代理**： 如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的
3. 下面将手动实现一个AOP功能（完整代码见文末）

<!--more-->

---

## 2. JDK动态代理

### 1. 主要代码

```java
//1. 服务接口类
public interface HelloService {
    public void sayHello(String name);
}

//2. 服务实现类
public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello(String name) {
        System.err.println("hello "+name);
    }
}

//3. 代理类
/**
 * 代理类 实现InvocationHandler接口
 */
public class HelloServiceProxy implements InvocationHandler {

    /**
     * 真实服务对象
     */
    private Object target;

    /**
     * 绑定委托对象，并返回一个代理类
     */
    public Object bind(Object target){
        this.target = target;
        //取得代理对象
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
    }

    /**
     *  通过代理对象调用方法首先进入这个方法
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.err.println("###########我的JDK动态代理############");
        Object result = null;
        //反射方法前调用
        System.err.println("我准备说hello");
        // 执行方法,相当于调用HelloServiceImpl类的sayHello方法.
        result = method.invoke(target, args);
        // 反射方法后调用.
        System.err.println("我说过hello了.");
        return result;
    }
}

```

### 2. 测试结果

```java
public class HelloServiceMain {
    public static void main(String[] args) {
        HelloServiceProxy helloHandler = new HelloServiceProxy();
        HelloService proxy = (HelloService) helloHandler.bind(new HelloServiceImpl());
        proxy.sayHello("xiongzp08");
        System.err.println(proxy.getClass());
    }
}
```

- 运行结果

```shell
我准备说hello
hello xiongzp08
我说过hello了.
class com.sun.proxy.$Proxy0
```

- 可以看到是为JDK代理(com.sun.proxy.$Proxy*)

## 3. CGLIB动态代理

> 下面代码演示的是CGLIB实现动态代理，这里要用到CGLIB框架，注意导入依赖，pom.xml文件增加内容如下

```xml
        <!-- https://mvnrepository.com/artifact/cglib/cglib -->
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.2.7</version>
        </dependency>
```

### 1. 主要代码

```java
//1. CGLIB代理类
//这里面用到了上一节中部分代码，未列出，完整代码见文末连接
// 实现MethodInterceptor接口
public class HelloServiceCglib implements MethodInterceptor {

    private Object target;

    /**
     * 创建代理对象，并返回
     */
    public Object getInstance(Object target) {
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        //回调方法
        enhancer.setCallback(this);
        // 创建代理对象
        return enhancer.create();
    }

    /**
     * 需实现intercept接口
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.err.println("我是CGLIB动态代理");
        Object result;
        // 反射方法前调用.
        System.err.println("我准备说hello.");
        // 执行方法,相当于调用HelloServiceImpl类的sayHello方法.
        result = methodProxy.invokeSuper(o, objects);
        // 反射方法后调用.
        System.err.println("我说过hello了.");
        return result;
    }
}

```

### 2. 测试结果

```java
public class HelloServiceMain {

    public static void main(String[] args) {
        HelloServiceCglib cglib = new HelloServiceCglib();
        HelloService service = (HelloService) cglib.getInstance(new HelloServiceImpl());

        service.sayHello("xiongzp08");
       System.err.println(service.getClass());
    }
}

```

- 运行结果

```shell
我准备说hello.
hello xiongzp08
我说过hello了.
class com.xzp.dynamic_proxy.HelloServiceImpl$$EnhancerByCGLIB$$af1808f4
```

----

## 相关链接

- [完整示例代码-GitHub](https://github.com/briarbear/mybatis-learning-books/tree/master/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAMybatis%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86/chapter6)