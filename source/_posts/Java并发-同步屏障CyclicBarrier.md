---
title: Java并发-同步屏障CyclicBarrier
date: 2018-07-16 16:20:58
toc: true
tags:
 - 并发
 - CyclicBarrier
 - 转载
categories:
 - Java
---

>   【转载】- 本文内容摘自《Java并发编程的艺术-方腾飞》

> `CyclicBarrier`的字面意思是可循环使用(`Cyclic`)的屏障(`Barrier`)。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

<!--more-->

## 1. CyclicBarrier简介

`CyclicBarrier`默认的构造方法是`CyclicBarrier( int parties)`，其参数表示屏障拦截的线程数量，每个线程调用`await`方法告诉`CyclicBarrier`我已经到达了屏障，然后当前线程被阻塞

- 示例代码 `CyclicBarrierTest.java`

```java
public class CyclicBarrierTest {

    /**
     * CyclicBarrier同步屏障 它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，
     * 直到最后一个线程到达屏障时，屏障才会开门，所有别屏蔽的线程才会继续运行
     */

    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args) {
        new Thread(() -> {
            try {
                c.await();
            } catch (Exception e) {

            }
            System.out.println(1);
        }).start();

        try {
            c.await();
        } catch (Exception e) {

        }
        System.out.println(2);
    }
}
```

因为主线程和子线程的调度是由CPU决定的，两个线程都有可能先执行，所以会产生两种输出，第一种可能输出如下:

```shell
1
2
#或者
2
1
```

`CyclicBarrier`还提供一个更高级的构造函数`CyclicBarrier(int parties，Runnable barrier-Action)`，用于在线程到达屏障时，优先执行`barrierAction`，方便处理更复杂的业务场景，如下所示。

- 示例代码:`CyclicBarrierTest2.java`

```java
public class CyclicBarrierTest2 {

    static CyclicBarrier c = new CyclicBarrier(2, new A());

    public static void main(String[] args) {
        new Thread(() -> {
            try {
                c.await();
            } catch (Exception e) {
            }
            System.out.println(1);
        }).start();

        try {
            c.await();
        } catch (Exception e) {

        }
        System.out.println(2);
    }

    static class A implements Runnable {
        @Override
        public void run() {
            System.out.println(3);
        }

    }
}
```

因为`CyclicBarrier`设置了拦截线程的数量是2，所以必须等代码中的第一个线程和线程A都执行完之后，才会继续执行主线程，然后输出2，所以代码执行后的输出如下。

```shell
3
1
2
```

---

## 2. CyclicBarrier的应用场景

`CyclicBarrier`可以用于多线程计算数据，最后合并计算结果的场景。例如，用一个Excel保存了用户所有银行流水，每个Sheet保存一个账户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用`barrierAction`用这些线程的计算结果，计算出整个Excel的日均银行流水，如下所示

- `BankWaterService.java` 代码过长,完整见文末连接

-----

## 3. CyclicBarrier和CountDownLatch的区别

`CountDownLatch`的计数器只能使用一次，而`CyclicBarrier`的计数器可以使用`reset0`方法重置。所以`CyclicBarrier`能处理更为复杂的业务场景。例如，如果计算发生错误，可以重置计数器，并让线程重新执行一次。

`CyclicBarrier`还提供其他有用的方法，比如`getNumberWaiting`方法可以获得`Cyclic-Barrier`阻塞的线程数量。`isBroken0`方法用来了解阻塞的线程是否被中断。代码清单8-5执行完之后会返回true，其中`isBroken`的使用代码如代码所示

```java
public class CyclicBarrierTest3 {
    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args){
        Thread thread = new Thread(() -> {
            try {
                c.await();
            } catch (Exception e) {
            }
        });
        thread.start();
        thread.interrupt();  //中断thread
        try {
            c.await();
        } catch (Exception e) {
            System.out.println(c.isBroken());
        }
    }
}
```

运行结果:

```shell
true
```

----

## 相关连接

- [完整代码-github](https://github.com/briarbear/java_core/blob/master/src/main/java/concurrent/cyclicbarrie)