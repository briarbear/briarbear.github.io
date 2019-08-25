---
title: Java并发-线程池的原理及使用
date: 2018-09-03 13:26:29
toc: true
tags:
 - 并发
 - 线程池
 - 转载
categories:
 - Java
---

-----

> 【转载】- 本文摘自《Java并发编程的艺术-方腾飞》

&nbsp;&nbsp;Java中的线程池是运用场景最多的并发框架，几乎所有的异步或并发执行任务的程序都可以使用线程池。在开发中，合理地使用线程池能够带来3个好处

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用线程池，必须对其实现原理了如指掌。

<!--more-->

## 1. 线程池的原理

当向线程池提交一个任务之后，线程池是如何处理这个任务的呢？本节来看一下线程池的主要处理流程，处理流程图如下图所示

- **线程池的主要处理流程**

![](http://cdn.briarbear.cn/201808031011_271.png)

1. 线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
2. 线程池判断工作队列是否已经满。如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程
3. 线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务

- `ThreadPoolExecutor`执行`execute()`方法的示意图

![](http://cdn.briarbear.cn/201808031019_955.png)

`ThreadPooIExecutor`执行`execute`方法分下面4种情况。

1. 如果当前运行的线程少于`corePooISize`，则创建新线程来执行任务（注意，执行这一步骤需要获取全局锁）。
2. 如果运行的线程等于或多于`corePooISize`，则将任务加入`BlockingQueue`。
3. 如果无法将任务加入`BlockingQueue`（队列已满），则创建新的线程来处理任务（注意，执行这一步骤需要获取全局锁）。
4. 如果创建新线程将使当前运行的线程超出`maximumPooISize`，任务将被拒绝，并调用`RejectedExecutionHandler.rejectedExecution()`方法。

`ThreadPooIExecutor`采取上述步骤的总体设计思路，是为了在执行`execute()`方法时，尽可能地避免获取全局锁（那将会是一个严重的可伸缩瓶颈）。在`ThreadPooIExecutor`完成预热之后（当前运行的线程数大于等于`corePooISize`），几乎所有的`execute()`方法调用都是执行步骤2，而步骤2不需要获取全局锁。

**源码分析：**上面的流程分析让我们很直观地了解了线程池的工作原理，让我们再通过源代码来看看是如何实现的，线程池执行任务的方法如下。

```java
public void execute (Runnable command){
  if(command == null)
    throw new NullPointerException;
    //如果线程数小于基本线程数，则创建线程并执行当前任务
    if(poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)){
        //如线程数大于等于基本线程数或线程创建失败，则将当前任务放到工作队列中。
        if(runState == RUNNING && workQueue.offer(command)){
            if(runState != RUNNING || poolSize == 0)
                ensureQueuedTaskHandled(command);
        }
        // 如果线程池不处于运行中或任务无法放入队列，并且当前线程数量小于最大允许的线程数量，
        // 则创建一个线程执行任务
        else if(!addIfUnderMaximumPoolSize(command))
        //抛出 Rej ectedExecutionException异常
        reject(command);
    }
}
```

**工作线程**：线程池创建线程时，会将线程封装成工作线程Worker，Worker在执行完任务后，还会循环获取工作队列里的任务来执行。我们可以从Worker类的run()方法里看到这点

```java
public void run(){
    try{
        Runnable task = firstTask;
        firstTask = null;
        while(task != null || (task = getTask()) != null){
            runTask(task);
            task = null;
        }
    }finally{
        workerDone(this);
    }
}
```

`ThreadPoolExecutor`中线程执行任务的示意图如下所示：

![](http://cdn.briarbear.cn/201808031106_940.png)

线程池中的线程执行任务分两种情况，如下

1. 在execute()方法中创建一个线程时，会让这个线程执行当前任务。
2. 这个线程执行完上图中1的任务后，会反复从`BlockingQueue`获取任务来执行

----

## 2. 线程池的使用

### 2.1 线程池的创建

我们可以通过`ThreadPoolExecutor`来创建一个线程池

```java
new ThreadPoolExecutor(corePoolSize, maximumPoolSize,keepAliveTime,TimeUnit,                        BlockingQueue<Runnable>,ThreadFactory,RejectedExecutionHandler)
```

创建线程池时需要输入几个参数，如下：
#### 1. `corePooISize`(线程池的基本大小）：

当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的`prestartAIICoreThreads`()方法，线程池会提前创建并启动所有基本线程。

#### 2. `maximumPooISize`(线程池最大数量）：

线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数就没什么效果。

#### 3. `keepAliveTime`(线程活动保持时间)：

线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个执行任务的时间比较短，可以调大时间，提高线程的利用率

#### 4. `TimeUnit`(线程活动保持时间的单位)：

可选的单位有天（DAYS）、小时（HOURS）、分钟( MNUTES)、毫秒(MIIIISECONDS)、微秒（MICROSECONDS，千分之一毫秒）和纳秒(NANOSECONDS，千分之一微秒)

#### 5. `BlockingQueue`（任务队列）：

用来保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列

- `ArrayBlockingQueue`: 是一个基于数组结构的有界阻塞队列，此队列按FIFO(先进先出)原则对元素进行排序
- `LinkedBlockingQueue`：一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通常要高于`ArrayBlockingQueue`。静态工厂方法 `Executots.newFixedThreadPool()`使用了这个队列。
- `SynchronousQueue`：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移出操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于`LinkedBlockingQueue`，静态工厂方法`Executor.newCachedThreadPool`使用了这个队列。
- `PriorityBlockingQueue`：一个具有优先级的无限阻塞队列

#### 6. `ThreadFactory`：(用于设置创建线程的工厂)

可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架`guava`提供的`ThreadFactoryBuilder`可以快速给线程池里的线程设置有意义的名字，代码如下：

```java
new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
```

#### 7. `RejectedExecutionHandler`（饱和策略）：

当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是`AbortPolicy`，表示无法处理新任务时抛出异常。在JDK1.5中Java线程池框架提供了4种策略
1.`AbortPolicy`：直接抛出异常。
2.`CallerRunsPolicy`：只用调用者所在线程来运行任务。
3.`DiscardOldestPolicy`：丢弃队列里最近的一个任务，并执行当前任务。
4.`DiscardPolicy`：不处理，丢弃掉。

-----

### 2.2 向线程池提交任务

- 可以使用两个方法向线程池提交任务，分别是`execute()`和`submit()`方法

  1. `execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功，可以通过以下代码可知`execute()`方法输入的是一个`Runnable`类的实例

     ```java
     threadsPool.execute(new Runnable() {
                           @Override
                             public void run() {
                                 // TODO Auto-generated method stub
                             }
                         });
     ```

  2. `sublime()`方法用于提交需要返回值的任务。线程池会返回一个`future`类型的对象，通过这个`future`对象可以判断任务是否执行成功，并且可以通过`future`的`get()`方法来获取返回值，`get()`方法会阻塞当前线程知道任务完成，而使用`get(long timeout, TimeUnit unit)`方法则会一直阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

  -------

### 2.3 线程池的关闭

- 我们可以通过调用线程池的`shutdown`或`shutdownNow`方法来关闭线程池，但是它们的实现原理不同;
  - `shutdown`的原理是只是将线程池的状态设置成`SHUTDOWN`状态，然后中断所有没有正在执行任务的线程。
  - `shutdownNow`的原理是遍历线程池中的工作线程，然后逐个调用线程的`interrupt`方法来中断线程，所以无法响应中断的任务可能永远无法终止。`shutdownNow`会首先将线程池的状态设置成`STOP`，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
- 只要调用了这两个关闭方法的其中一个，`isShutdown`方法就会返回`true`。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用`isTerminaed`方法会返回`true`。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用`shutdown`来关闭线程池，如果任务不一定要执行完，则可以调用`shutdokwnNow`

----

## 其他链接

- [聊聊并发(三) Java线程池的分析与使用](http://ifeve.com/java-threadpool/)