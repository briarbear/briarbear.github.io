---
title: Java并发-阻塞队列实现生产者消费者
date: 2018-07-04 10:50:44
toc: true
tags:
 - 并发
 - 阻塞队列
categories:
 - Java
---

## 1. 阻塞队列

> 前文中讲到过：阻塞队列（`BlockingQueue`）是一个支持两个附加操作的队列。阻塞队列常用于生产者和消费者的场景，生产者是向队列中添加元素的线程，消费者是从队列取元素的线程。阻塞队列就是生产者用来存放元素，消费者用来获取元素的容器

- 完整代码见文末

<!--more-->

---

## 2. 消费者

```java
/**
 * 消费者
 */
public class Consumer implements Runnable {
    protected BlockingQueue<Object> queue;

    public Consumer(BlockingQueue<Object> queue) {
        this.queue = queue;

    }

    @Override
    public void run() {
        while (true){
            try {
                Object object = queue.take(); //从队列中取
                System.out.println("consumed resource... the queue size is" + queue.size());
                take(object);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
	// 消费对象
    void take(Object obj){
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("comsuming object " + obj);

    }
}
```

---

## 3. 生产者

```java
/**
 * 生产者
 */
public class Producer implements Runnable {

    protected BlockingQueue<Object> queue;

    public Producer(BlockingQueue<Object> queue) {
        this.queue = queue;
    }

    //生产者创建一个资源（Object对象）并将它存入到队列中，如果队列已满，它将会等待到消费者线程从队列中取走资源，所以队列长度永远不会超过20
    @Override
    public void run() {
        while (true){
            Object object = getResource();
            try {
                queue.put(object);
                System.out.println("Produced resource...the queue size is " + queue.size());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    //生产对象
    Object getResource(){
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return new Object();
    }
}
```

---

## 4. 测试启动

```java
/**
 * 测试启动类
 * 分别启动4个生成者线程，3个消费者线程
 * 生产者不断添加对象到队列中，消费者不断从队列中取出
 * 运行10s后，程序退出
 */
public class ProducerConsumerExample {
    public static void main(String[] args) throws InterruptedException {
        int numProducers = 4; //生产者数量
        int numConsumers = 3; //消费者数量
        //使用LinkedBlockingDeque - 一个由链表结构组成的双向阻塞队列
        //BlockingQueue<Object> queue = new LinkedBlockingDeque<>(20);
        BlockingQueue<Object> queue = new LinkedBlockingQueue<>(20); //也可以使用链表阻塞队列
        //分别启动生产者、消费者
        for (int i = 0; i < numProducers; i++) {
            new Thread(new Producer(queue)).start();
        }

        for (int i = 0; i < numConsumers; i++) {
            new Thread(new Consumer(queue)).start();
        }

        Thread.sleep(10*1000);
        Thread.yield();
        System.exit(0);
    }
}
```

运行结果：

```shell
consumed resource... the queue size is1
Produced resource...the queue size is 3
Produced resource...the queue size is 2
consumed resource... the queue size is2
consumed resource... the queue size is3
Produced resource...the queue size is 3
Produced resource...the queue size is 4
Produced resource...the queue size is 2
Produced resource...the queue size is 3
Produced resource...the queue size is 4
Produced resource...the queue size is 5
comsuming object java.lang.Object@619f2f27
......
```

---

## 相关链接

- [完整代码](https://github.com/briarbear/java_core/tree/master/src/main/java/concurrent/blockqueue)