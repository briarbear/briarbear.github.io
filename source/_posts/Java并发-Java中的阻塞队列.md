---
title: Java并发-Java中的阻塞队列
date: 2018-07-04 09:57:54
toc: true
tags:
 - 并发
 - 阻塞队列
categories:
 - Java
---

> 本文摘自《Java并发编程的艺术-方腾飞》

> 本节将介绍什么是阻塞队列，以及Java中阻塞队列的4种四种处理方式，并介绍Java7（Java8相同）中提供的7种阻塞队列，稍后分析阻塞队列的一种实现方式

<!--more-->

## 1. 什么是阻塞队列

​    阻塞队列（`BlockingQueue`）是一个支持两个附加操作的队列。这两个附加的操作支持阻塞的插入和移除方法
  >1. *支持阻塞的插入方法：意思是当队列满时，队列会阻塞插入元素的线程，直到队列不满*
  >2. *支持阻塞的移除方法：意思是在队列为空时，获取元素的线程会等待队列变为非空*    

阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器

​    在阻塞队列不可用时，这两个附加操作提供了4种处理方式，如表1所示

- *插入和移除操作的 4 中处理方式*

| 方法 / 处理方式 | 抛出异常  | 返回特殊值 | 一直阻塞 |      超时退出      |
| :-------------: | :-------: | :--------: | :------: | :----------------: |
|    插入方法     |  add(e)   |  offer(e)  |  put(e)  | offer(e,time,unit) |
|    移除方法     | remove(e) |   poll()   |  take()  |  poll(time,unit)   |
|    检查方法     | element() |   peek()   |  不可用  |       不可用       |

-  **抛出异常**：当队列满时，如果再往队列里插入元素，会抛出 `IllegalStateException（"Queue full"）`异常。当队列空时，从队列里获取元素会抛出 `NoSuchElementException`异常
-  **返回特殊值**：当往队列插入元素时，会返回元素是否插入成功，成功返回 true。如果是移除方法，则是从队列里取出一个元素，如果没有则返回 `null`
-  **一直阻塞**：当阻塞队列满时，如果生产者线程往队列里 `put` 元素，队列会一直阻塞生产者线程，直到队列可用或者响应中断退出。当队列空时，如果消费者线程从队列里`take` 元素，队列会阻塞住消费者线程，直到队列不为空
-  **超时退出**：当阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生产者线程一段时间，如果超过了指定的时间，生产者线程就会退出

这两个附加操作的 4 种处理方式不方便记忆，所以我找了一下这几个方法的规律。`put`和 `take` 分别尾首含有字母 `t`，`offer` 和 `poll` 都含有字母 `o`

> 注意：如果是无界阻塞队列，队列不可能会出现满的情况，所以使用 `put` 或 `offer` 方法永远不会被阻塞，而且使用 `offer` 方法时，该方法永远返回 `true`

-----

## 2. Java里的阻塞队列

JDK7提供了7个阻塞队列，如下：

- `ArrayBlockingQueue`：一个由数组结构组成的有界阻塞队列
- `LinkedBlockingQueue`：一个由链表结构组成的有界阻塞队列
- `PriorityBlockingQueue`：一个支持优先级排序的无界阻塞队列
- `DelayQueue`：一个使用优先级队列实现的无界阻塞队列
- `SynchronousQueue`：一个不存储元素的阻塞队列
- `LinkedTransferQueue`：一个由链表结构组成的无界阻塞队列
- `LinkedBlockingDeque`：一个由链表结构组成的双向阻塞队列。

   ---

#### 1. **ArrayBlockingQueue**

- ​ `ArrayBlockingQueue` 是一个用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原则对元素进行排序
- 默认情况下不保证线程公平的访问队列，所谓公平访问队列是指阻塞的线程，可以按照阻塞的先后顺序访问队列，即先阻塞线程先访问队列。非公平性是对先等待的线程是非公平的，当队列可用时，阻塞的线程都可以争夺访问队列的资格，有可能先阻塞的线程最后才访问队列。为了保证公平性，通常会降低吞吐量。我们可以使用以下代码创建一个公平的阻塞队列

```java
ArrayBlockingQueue  fairQueue  =  new   ArrayBlockingQueue (1000. true) ;
```

-  访问者的公平性是使用可重入锁实现的，代码如下：

```java
public ArrayBlockingQueue (int capacity, boolean fair){
    if (capacity <= 0)
    throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty  =  lock.newCondition();
    notFull = lock.newCondition();
}
 
```


#### 2. **LinkedBlockingQueue**

- `LinkedBlockingQueue`是一个用链表实现的有界阻塞队列。此队列的默认和最大长度为`Integer.MAX VALUE`。此队列按照先进先出的原则对元素进行排序。

#### 3. **PriorityBlockingQueue**

- `    PriorityBlockingQueue`是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序升序排列。也可以自定义类实现`compareTo()`方法来指定元素排序规则，或者初始化`PriorityBlockingQueue`时，指定构造参数`Comparator`来对元素进行排序。需要注意的是不能保证同优先级元素的顺序。

#### 4. **DelayQueue**

- `DelayQueue`是一个支持延时获取元素的无界阻塞队列。队列使用`PriorityQueue`来实现。队列中的元素必须实现`Delayed`接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素
- `DelayQueue`非常有用，可以将`DelayQueue`运用在以下应用场景:
  - 缓存系统的设计：可以用`DelayQueue`保存缓存元素的有效期，使用一个线程循环查询`DelayQueue`，一旦能从`DelayQueue`中获取元素时，表示缓存有效期到了
  - 定时任务调度：使用`DelayQueue`保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，比如`TimerQueue`就是使用`DelayQueue`实现的

#### 5. **SynchronousQueue**

- `SynchronousQueue`是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素
- 它支持公平访问队列。默认情况下线程采用非公平性策略访问队列。使用以下构造方法可以创建公平性访问的`SynchronousQueue`，如果设置为`true`，则等待的线程会采用先进先出的顺序访问队列

```java
public SynchronousQueue(boolean fair){
    transferer = fair?new TransferQueue():new TransferStack();
}
```

- `SynchronousQueue`可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合传递性场景。`SynchronousQueue`的吞吐量高于`LinkedBlockingQueue`和`ArrayBlockingQueue`

#### 6. **LinkedTransferQueue**

- `LinkedTransferQueue`是一个由链表结构组成的无界阻塞`TransferQueue`队列。相对于其他阻塞队列，`LinkedTransferQueue`多了`tryTransfer`和`transfer`方法。

  - `transfer`方法

    如果当前有消费者正在等待接收元素（消费者使用`take0`方法或带时间限制的`poll0`方法时），`transfer`方法可以把生产者传人的元素立刻`transfer`（传输）给消费者。如果没有消费者在等待接收元素，`transfer`方法会将元素存放在队列的`tail`节点，并等到该元素被消费者消费了才返回。`transfer`方法的关键代码如下

    ```java
    Node pred = tryAppend(S,haveData);
    return awaitMatch(s,pred,e,(how = TIMED),nanos);
    ```

    第一行代码是试图把存放当前元素的s节点作为tail节点。第二行代码是让CPU自旋等待消费者消费元素。因为自旋会消耗CPU，所以自旋一定的次数后使用Thread.yield0方法来暂停当前正在执行的线程，并执行其他线程

  - `tryTransfer`方法

    - `tryTransfer`方法是用来试探生产者传人的元素是否能直接传给消费者。如果没有消费者等待接收元素，则返回`false`。和`transfer`方法的区别是`tryTransfer`方法无论消费者是否接收，方法立即返回，而`transfer`方法是必须等到消费者消费了才返回。
    - 对于带有时间限制的`tryTransfer(E e，long timeout，TimeUnit unit)`方法，试图把生产者传人的元素直接传给消费者，但是如果没有消费者消费该元素则等待指定的时间再返回，如果超时还没消费元素，则返回`false`，如果在超时时间内消费了元素，则返回`true`。

#### 7. **LinkedBlockingDeque**

- `LinkedBlockingDeque`是一个由链表结构组成的双向阻塞队列。所谓双向队列指的是可以从队列的两端插入和移出元素。双向队列因为多了一个操作队列的入口，在多线程同时人队时，也就减少了一半的竞争。相比其他的阻塞队列，`LinkedBlockingDeque`多了`addFirst`、`addLast`、`offerFirst`、`offerLast`、`peekFirst`和`peekLast`等方法，以`First`单词结尾的方法，表示插入、获取( peek)或移除双端队列的第一个元素。以`Last`单词结尾的方法，表示插入、获取或移除双端队列的最后一个元素。另外，插入方法`add`等同于`addLast`，移除方法`remove`等效于`removeFirst`。但是`take`方法却等同于`takeFirst`，不知道是不是`JDK`的`bug`，使用时还是用带有`First`和`Last`后缀的方法更清楚。

- 在初始化`LinkedBlockingDeque`时可以设置容量防止其过度膨胀。另外，双向阻塞队列可以运用在“工作窃取”模式中。