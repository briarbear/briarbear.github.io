---
title: 设计模式-Java中三种代理模式
date: 2018-07-05 11:02:39
toc: true
tags: 代理模式
categories: 设计模式
---

> 代理模式是常用的结构性设计模式之一，它为对象的间接访问提供了一个解决方案，可以对对象的访问进行控制。可以在目标对象的基础上，增加额外的功能操作，即扩展目标对象的功能。
>
> 体现编程思想：不修改原模块代码或方法，通过代理的方式来扩展
>
> 完整代码见文末链接

<!--more-->

## 1. 静态代理

> 静态代理在使用时，需要定义接口或者父类，被代理对象与代理对象一起实现相同的接口或者是继承相同的父类

**关键：在编译期确定代理对象，在程序运行前代理类的.class文件就已经存在**

如：在代理对象中实例化被代理对象或者将被代理对象传入代理对象的构造方法

- 示例代码

```java
//对象接口
public interface Subject {
    public void doSomething();
}

//接口真实实现类（被代理对象）
public class SubjectImpl implements Subject{
    @Override
    public void doSomething() {
        System.out.println("do something.....");
    }
}

//接口代理实现类 （代理对象）
public class SubjectProxy implements Subject {

    Subject subjectImlp = new SubjectImpl();
    @Override
    public void doSomething() {
        subjectImlp.doSomething(); //调用真事实现类的方法
    }
}

//测试方法
public class StaticProxyTest {

    @Test
    public void test(){
        Subject sub = new SubjectProxy();  //实例化代理实现类对象
        sub.doSomething();
    }
}


```

**总结：**

1. 可以做到在不修改目标对象的功能前提下，对目标功能扩展
2. 缺点：
   1. 代理类与委托类实现相同的接口，同时要实现相同的方法，这样就出现了大量的代码重复。如果接口增加一个方法，除了实现类需要实现这个方法外，所有代理类也需要实现该方法。增加了代码维护的复杂度

----

## 2. 动态代理

> 在运行期，通过反射机制创建一个实现了一组接口的新类
>
> 是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)，因此也叫JDK代理
>
> 接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（`InvocationHandler.invoke`）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强

- 与前面文章[Spring-AOP核心原理及手写实现](https://briarbear.github.io/2018/07/03/Spring-AOP%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86%E5%8F%8A%E6%89%8B%E5%86%99%E5%AE%9E%E7%8E%B0/)对应

**JDK中生成代理对象的API**

- 代理类所在的包：`java.lang.reflect.Proxy`
- JDK实现代理只需要使用`newProxyInstance`方法，该方法需要接受三个参数，完整的写法是

`static Object newProxyInstance(ClassLoader loader, Class [] interfaces, InvocationHandler handler)`

- 注意该方法在Proxy类中是静态方法，且接受的三个参数一次为：
  - `ClassLoader loader`:指定当前目标对象使用类加载器,用null表示默认类加载器
  - `Class [] interfaces`:需要实现的接口数组
  - `InvocationHandler handler`:调用处理器,执行目标对象的方法时,会触发调用处理器的方法,从而把当前执行目标对象的方法作为参数传入

`java.lang.reflect.InvocationHandler`：这是调用处理器接口，它自定义了一个 `invoke` 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。

```java
// 该方法负责集中处理动态代理类上的所有方法调用。第一个参数既是代理类实例，第二个参数是被调用的方法对象
// 第三个方法是调用参数。
Object invoke(Object proxy, Method method, Object[] args)

```

- 代码示例：

```java
//接口
public interface Subject {
    public void doSomething();
}
//////////////////////////////////////////////
//接口真实实现类
public class SubjectImpl implements Subject {

    @Override
    public void doSomething() {
        System.out.println("dynamic doing something .... ");
    }
}
//--------------------------------------------------
//代理处理类
public class ProxyHandler implements InvocationHandler {

    private Object target;

    /**
     * 绑定委托对象，并返回代理类
     */
    public Object bind(Object target){
        this.target = target;

        //绑定该类实现的所有接口，取得代理类
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),this);

    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        //这里就可以进行所谓的AOP编程
        //执行前方法
        System.out.println("before...");
        result = method.invoke(target,args);
        //执行后方法
        System.out.println("after...");

        return result;
    }
}

//------------------------------------------------------------------------------------------
//测试类
public class ProxyHandlerTest {

    @Test
    public void bind() {
        var proxy = new ProxyHandler();
        //绑定该类实现的所有接口
        Subject sub = (Subject) proxy.bind(new SubjectImpl());
        sub.doSomething();
    }
}
```

**总结**：

- 代理对象不需要实现接口，但目标对象一定要实现接口，否则不能用动态代理

-----

## 3. Cglib代理

> 上面的静态代理和动态代理模式都是需要要求目标对象就是一个接口或者多个接口，但有时候目标对象就是一个单独的对象，并没有实现任何接口，这个时候就可以使用`构建目标对象子类的方式`实现代理，这种方法就叫做： `Cglib代理`

`Cglib`代理，也叫作子类代理，它是在内存中构建一个子类对象从而实现对目标对象功能的扩展

- `Cglib`是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如`Spring AOP`和`synaop`,为他们提供方法的`interception`(拦截)

- `Cglib`包的底层是通过使用**字节码处理框架ASM**来转换字节码并生成新的子类.

- 代理的类不能为`final`,否则报错；目标对象的方法如果为`final/static`,那么就不会被拦截,即不会执行目标对象额外的业务方法.

- 示例代码：

  ```java
  //真实实现类 无继承接口
  public class SubjectImpl {
      public void doSomeThing(){
          System.out.println("cglib proxy doing something...");
      }
  }
  
  //--------------------------------------------------------------------------------------
  import org.springframework.cglib.proxy.Enhancer;
  import org.springframework.cglib.proxy.MethodInterceptor;
  import org.springframework.cglib.proxy.MethodProxy;
  
  import java.lang.reflect.Method;
  
  /**
   * Cglib子类代理工厂
   * 对SubjectImpl 在内存中工台构建一个子类对象
   * @author briarbear
   * @blog https://briarbear.github.io
   * @create 2018-07-05-15:37
   */
  public class ProxyFactory implements MethodInterceptor {
      //MethodInterceptor 需要引入cglib的jar文件,但是Spring的核心包中已经包括了Cglib功能,需要引入Spring core
  
      //维护目标对象
      private Object target;
  
      public ProxyFactory(Object target) {
          this.target = target;
      }
  
      //给目标对象创建一个代理对象
      public Object getProxyInstance(){
          //1. 工具类
          var en = new Enhancer();
          //2. 设置父类
          en.setSuperclass(target.getClass());
          //3. 设置回调函数
          en.setCallback(this);
          //4. 创建子类(代理对象)
          return en.create();
      }
  
  
      @Override
      public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
          System.out.println("before....");
          //执行目标对象的方法
          Object result = method.invoke(target,args);
          System.out.println("afer...");
          return result;
  
      }
  }
  
  //----------------------------------------------------------------------------------------
  //测试类
      @Test
      public void test(){
          //目标对象
          var target = new SubjectImpl();
  
          //代理对象
          var proxy = (SubjectImpl)new ProxyFactory(target).getProxyInstance();
  
          //执行代理对象的方法
          proxy.doSomeThing();
  
  
      }
  ```

  

**总结**

- 在`Spring AOP`编程中，如果加入容器的目标对象有实现接口,用JDK代理。如果目标对象没有实现接口,用`Cglib`代理

## 相关链接

- [完整示例代码](https://github.com/briarbear/java_core/tree/master/src/main/java/design_pattern/proxy/dynamic_proxy)
- [java的动态代理机制详解](https://www.cnblogs.com/xiaoluo501395377/p/3383130.html)
- [知乎-动态代理的作用是什么？-郭无心](https://www.zhihu.com/question/20794107)
- [java的三种代理模式](https://segmentfault.com/a/1190000009235245)
