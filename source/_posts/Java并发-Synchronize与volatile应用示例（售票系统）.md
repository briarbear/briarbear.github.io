---
title: Java并发-Synchronize与volatile应用示例（售票系统）
date: 2018-07-04 15:14:56
toc: true
tags:
 - 并发
 - Synchronize
categories:
 - Java
---

> 场景描述：假设现在我们总共有1000张票要进行出售，共有10个出售点，那么当售票到最后只有一张票时每个售票点如何去处理这唯一的一张票？或者对于某一张票而言，假设它正在售票站1售票的程序执行过程中，但是还没有出售，那么此时，其他售票站改如何去处理这张票呢？

<!--more-->

---

> 这是典型的并发问题，涉及线程同步，在Java总可以采用锁的机制实现，本文将例举`synchronize`与`volatile`的原子变量类来解决,


> 每个方案都是详细的代码解释，完整代码可见文末链接

## 1. synchronize解决方案

```java
public class SynchronizeExample {
    public static void main(String[] args) {
        int n  = 10;
        Thread[] threads = new Thread[n];//10个窗口售卖，开10个进程
        for (int i = 0; i < n; i++) {
            threads[i] = new Thread(new ticket());
            threads[i].setName(i+1+"号窗口");
            threads[i].start();
        }
    }
}
 
class ticket implements Runnable{
    private static int count = 1000;
    private static String key = "key"; //通过key这个互斥变量来控制对count的访问
    @Override
    public void run() {
        while (count > 0){
            synchronized (key){
                if (count > 0){
                    System.out.println("第"+Thread.currentThread().getName()+"在卖第"+(1001-count)+"张票");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    count--;
                }else {
                    break;
                }
            }
        }
        System.out.println("第"+Thread.currentThread().getName()+"的票已经卖完");
    }
}
```

----

## 2. volatile+AtomicInteger解决方案

```java
/**
* 同样是买票问题 尝试使用原子类 + volatile 解决
*/
public class SynchronizeExample2 {
    public static void main(String[] args) {
        int windows = 10;
        for (int i = 0; i < windows; i++) {
            Thread thread = new Thread(new Sale());
            thread.setName(i+1+"号窗口");
            thread.start();
        }
    }
}
 
class Sale implements Runnable{
    //保证count的自增操作的原子性，使用volatile保证共享变量的可见性
    private static volatile AtomicInteger count = new AtomicInteger(0);
    @Override
    public void run() {
        int temp =count.get();
        while (temp <= 1000){
            temp = count.getAndIncrement();
            if (temp < 1000){
                System.out.println("第"+Thread.currentThread().getName()+"在卖第"+(temp+1)+"张票");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
        System.out.println("第"+Thread.currentThread().getName()+"的票已经卖完");
    }
}
```

----

## 相关链接

- [示例完整源码-github](https://github.com/briarbear/java_core/tree/master/src/main/java/concurrent/synchronize)