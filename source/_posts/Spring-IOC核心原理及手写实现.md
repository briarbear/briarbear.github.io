---
title: Spring-IOC核心原理及手写实现
date: 2018-07-03 11:54:24
toc: true
tags: IOC
categories: Spring
---

## 0. IOC基本原理

> 建议不要硬着头皮看Spring代码，本身代码800多m，就是不上班开始看，也不知道什么时候看完，如果想学学IOC，控制反转这些建议看看jodd项目，比较简练，但是我仍然不建议过多看这些框架的代码，因为这些代码要完成任务需要很多琐碎的类实现，比如读取某个包下的所有类，解析class的头文件，反射各种信息，再加上封装。很有可能读源码的过程中掉在各种细节出不来，所以读这种源码不必事无巨细，理解原理即可。

<!--more-->

​    基本原理其实就是通过反射解析类及其类的各种信息，包括构造器，方法及其参数，属性。然后将其封装成`bean`定义信息类、`constructor`信息类、`method`信息类、`property`信息类，最终放在一个`map`中，也就是所谓的`container`，池等等，其实就是个`map`。

1. 当你写好配置文件，启动项目后，框架会按照你的配置文件找到那个要scan的包，
2. 然后解析包里面的所有类，找到所有`@bean`、`@service`等注解的类，利用反射解析它们，包括解析构造器，方法，属性等，然后封装成各种信息类到一个map里
3. 每当你需要一个`bean`的时候，框架就会从`container`找到是不是有这个类的定义。如果找到就通过构造器new出来（这就是控制反转，不用你new，框架帮你new）
4. 再在这个类找是不是有要注入的属性或者方法，比如标有`@autowrite`的属性，如果有则还是到`container`找对应的解析类，new出对象，并通过之前解析出来的信息类找到`setter`方法，然后用该方法注入对象（这就是依赖注入）。
5. 如果其中有一个类`container`里没找到，则抛出异常，比如常见的`spring`无法找到该类定义，无法wire的异常。
6. 还有就是嵌套bean则用了一下递归，`container`会放到`servletcontext`里面，每次`reQuest`从`servletcontext`找这个`container`即可，不用多次解析类定义。
7. 如果`bean`的`scope`是`singleton`，则会重用这个bean不会再重新创建，将这个`bean`放到一个map里，每次用都先从这个map里面去找。
8. 如果`scope`是`session`，则该`bean`会放到`session`里面，仅此而已。

----

## 1. 说明

> IOC是Spring中核心功能之一，基本原理就是通过反射解析类及其各种信息，然后封装成对象，存放在map中，即bean容器。可以根据其基本原理简单实现一个Spring IOC模块,完整代码见文末链接。
>
> 模拟同学会面的打招呼场景，定义 `Greeting`接口，包含 `hello`方法，实现对某人打招呼`Hello`类实现该 `Greeting` 接口，实现`hello`方法。然后定义人类`Person`的抽象类，具备属性 `name`，`Student`为该类的继承类。同时还包括`BeanFactory`接口，以及负责xml文件解析与`bean`创建等

-----

## 2. 基本类

```java
/**
 * 打招呼
 */
public interface Greeting {
    /**
     * 对某个打招呼
     * @param p
     */
    public void hello(Person p);
}

public class Hello implements Greeting {
    @Override
    public void hello(Person p) {
        System.out.println("hello to "+p.name);
    }
}

/**
 * Person抽象类
 */
public abstract class Person {
    public String name; //姓名
}

/**
 * 学生类
 */
public class Student extends Person{

}

/**
 * 学生服务类 
 */
public class StudentService {

    private Greeting greeting;

    public Greeting getGreeting() {
        return greeting;
    }

    public void setGreeting(Greeting greeting) {
        this.greeting = greeting;
    }

    /**
     * 对某人打招呼
     * @param p
     */
    public void helloService(Person p){
        this.greeting.hello(p);
    }
}



```

- 上述代码中，学生服务类中的 `helloService`方法，表示调用 `greeting`的方法，完成对某个打招呼的功能，这里依赖于另外一个对象`Person`类，这样的依赖注入，通过IOC来实现

----

## 3. 配置解析与容器类

```java
/**
 * BeanFactory接口
 */
public interface BeanFactroy {
    public Object getBean(String name);
}

/**
 * IOC 容器类
 */
public class ClassPathXmlApplicationContext implements BeanFactroy {
    private Map<String,Object> beans = new HashMap<String, Object>(); //IOC容器

    /**
     * 构造函数，完成xml文件的解析
     */
    public ClassPathXmlApplicationContext() throws JDOMException, IOException, ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        SAXBuilder builder = new SAXBuilder();
        Document document  = builder.build(this.getClass().getClassLoader().getResource("iocbeans.xml"));
        Element root = document.getRootElement();
        //获得bean元素集合
        List<Element> list = root.getChildren("bean");

        for (int i = 0; i < list.size(); i++) {
            Element element = list.get(i);
            String id = element.getAttributeValue("id");
            String clazz = element.getAttributeValue("class");

            System.out.println(id + " : " + clazz);

            //通过反射构建对象
//            Object obj = Class.forName(clazz).newInstance()
            Object obj = Class.forName(clazz).getDeclaredConstructor().newInstance();

            //添加到map容器中
            beans.put(id,obj);

            //注入依赖
            for (Element property : element.getChildren("property")) {
                String name = property.getAttributeValue("name");
                String injectBean = property.getAttributeValue("bean");
                Object propertyObj = beans.get(injectBean);

                //拼接setter方法
                String methodNmae = "set" + name.substring(0,1).toUpperCase() + name.substring(1);
                System.out.println("method name = " + methodNmae);

                Method m = obj.getClass().getMethod(methodNmae,propertyObj.getClass().getInterfaces());

                //注入
                m.invoke(obj,propertyObj);

            }

        }

    }

    @Override
    public Object getBean(String name) {
        return beans.get(name);
    }
}

```

---

## 4. xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans>

    <bean id = "hello" class = "ioc.Hello"/>
    <bean id = "student" class="ioc.Student"/>
    <bean id = "service" class="ioc.StudentService">
        <property name = "greeting" bean = "hello"/>
    </bean>
</beans>

```

---

## 5. 功能测试

```java
public class StudentServiceTest {

    @Test
    public void helloService() throws NoSuchMethodException, IllegalAccessException, IOException, InstantiationException, JDOMException, InvocationTargetException, ClassNotFoundException {
        //var jdk10语法
        var factory = new ClassPathXmlApplicationContext();
	    var service = (StudentService)factory.getBean("service"); //获得service对象实例
        var student = (Student)factory.getBean("student"); //获得student对象实例
        student.name = "xiongzp08";
        service.helloService(student); 
    }
}
```

----

## 6. 运行结果

```shel
hello : ioc.Hello
student : ioc.Student
service : ioc.StudentService
method name = setGreeting
hello to xiongzp08

Process finished with exit code 0

```

----

## 相关链接

- [完整代码](https://github.com/briarbear/SpringDemo)
- [Spring Ioc实现机制——简析](https://blog.csdn.net/Anger_Coder/article/details/12706277)
- [知乎-王奕然](https://www.zhihu.com/question/21346206)

  