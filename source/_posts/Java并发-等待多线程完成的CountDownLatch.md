---
title: Java并发-等待多线程完成的CountDownLatch
date: 2018-07-16 17:10:57
toc: true
tags:
 - 并发
 - CountDownLatch
 - 转载
categories:
 - Java
---

> 【转载】-本文摘自《Java并发编程的艺术-方腾飞》

`CountDownLatch`允许一个或多个线程等待其他线程完成操作。
假如有这样一个需求：我们需要解析一个Excel里多个sheet的数据，此时可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成sheet的解析操作，最简单的做法是使用join()方法，如下所示。

<!--more-->

```java
public class JoinCountDownLatchTest {

    public static void main(String[] args) throws InterruptedException {

        Thread parser1 = new Thread(()-> System.out.println("parser1 finish"));
        Thread parser2 = new Thread(() -> System.out.println("parser2 finish"));

        parser1.start();
        parser2.start();
        parser1.join();
        parser2.join();

        System.out.println("all parser finish");
    }
}
```

`join`用于让当前执行线程等待`join`线程执行结束。其实现原理是不停检查join线程是否存活，如果`join`线程存活则让当前线程永远等待。其中，`wait(0)`表示永远等待下去，代码片段如下:

```java
while(isAlive){
    wait(0);
}
```

直到join线程中止后，线程的`this.notifyAll()`方法会被调用，调用`notifyAll()`方法是在JVM里实现的，所以在JDK里看不到，大家可以查看JVM源码

在JDK l.5之后的并发包中提供的`CountDownLatch`也可以实现join的功能，并且比`join`的功能更多，如代码清单所示

```java
public class CountDownLatchTest {

    static CountDownLatch c = new CountDownLatch(2);
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            System.out.println(1);
            c.countDown();
            System.out.println(2);
            c.countDown();
        }).start();

        c.await();
        System.out.println("3");
    }
}
```

&nbsp;&nbsp;`CountDownLacth`的构造函数接受一个`int`类型的参数作为计数器,如果你想等待N个点完成,这里传入N
&nbsp;&nbsp;当我们调用`CountDownLacth`的`countDown`方法时,N就会减1,`CountDownLatch`的`await`方法会阻塞当前线程,直到N变成0
&nbsp;&nbsp;由于`countDown`方法可以用在任何其他地方,所以这里说的n个点,可以是N个线程,也可以是一个线程里面的N个步骤,用在多线程时,只需要把这个`CountDownLatch`的引用传递到线程里即可
&nbsp;&nbsp;如果一个线程执行过慢,还可以使用另外一个带指定时间的`await`方法,`await(long time,TimeUint unit)`



- 更多`CountDownLatch`的应用示例，可参考文末的完整代码链接

## 相关连接

- [Github-完整代码](https://github.com/briarbear/java_core/blob/master/src/main/java/concurrent/countdownlatch/art_code)