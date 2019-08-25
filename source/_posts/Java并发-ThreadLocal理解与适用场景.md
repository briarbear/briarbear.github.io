---
title: Java并发-ThreadLocal理解与适用场景
date: 2018-06-26 16:41:53
toc: true
tags:
 - 并发
 - ThreadLocal
categories:
 - Java
---

---

> Java中的`ThreadLocal`类允许我们创建只能被同一个线程读写的变量。因此，如果一段代码含有一个`ThreadLocal`变量的引用，即使两个线程同时执行这段代码，它们也无法访问到对方的`ThreadLocal`变量

- `ThreadLocal`并不是解决多线程共享变量的问题，既然变量不共享，更谈不上同步到问题
- `ThreadLocal` 提供了线程本地变量的实例。它与普通变量的区别在于，每个使用该变量的线程都会初始化一个完全独立的实例副本。`ThreadLocal` 变量通常被`private static`修饰。当一个线程结束时，它所使用的所有 `ThreadLocal` 相对的实例副本都可被回收

---

 <!--more-->

## 1. ThreadLocal的使用

#### 1. 创建`ThreadLocal`变量：

```java
private ThreadLocal<String> myThreadLocal = new ThreadLocal<String>();  // 制定泛型类型
```

- 代码实例化一个`ThreadLocal`对象，只需要实例化一次，并且也不需要知道它是被哪个线程实例化。虽然所有的线程都能访问到这个`ThreadLocal`变量实例，但是每个线程只能访问到自己通过调用`ThreadLocal`的`set()`方法设置的值，即使在两个不同的线程在同一个`ThreadLocal`对象上设置了不同的值，他们仍然无法访问到对象的值

#### 2. 访问`ThreadLocal`变量

```java
myThreadLocal.set("A thread local value");   // 设置变量值
String threadLocalValue = myThreadLocal.get() //获取变量值
```
#### 3. 初始化`ThreadLocal`变量的值

- 由于在`ThreadLocal`对象中设置的值只能被设置这个值的线程访问到，线程无法在`ThreadLocal`对象上使用`set()`方法保存一个初始值，并且这个初始值能被所有线程访问到。
- 但是我们可以通过创建一个`ThreadLocal`的子类并且重写`initialValue()`方法，来为一个`ThreadLocal`对象指定一个初始值

```java
private ThreadLocal myThreadLocal = new ThreadLocal<String>() {
    @Override protected String initialValue() {
        return "This is the initial value";
    }
};
```
#### 4. 完整的`ThreadLocal`使用实例

```java
package concurrent.thread_local;
public class ThreadLocalExample {
    public static class MyRunnable implements Runnable{
        //ThreadLocal变量
        private ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();
        public void run() {
            threadLocal.set((int)(Math.random() * 1000));
            try {
                Thread.sleep(2000); //休眠2秒
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
        }
 
    }
 
    public static void main(String[] args) throws InterruptedException {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread1 = new Thread(myRunnable);
        Thread thread2 = new Thread(myRunnable);
 
        thread1.start();
        thread2.start();
 
        thread1.join();  //等待线程终止
        thread2.join();
    }
}
 
```

- 上面的例子创建了一个`MyRunnable`实例，并将该实例作为参数传递给两个线程。两个线程分别执行`run()`方法，并且都在`ThreadLocal`实例上保存了不同的值。如果它们访问的不是`ThreadLocal`对象并且调用的`set()`方法被同步了，则第二个线程会覆盖掉第一个线程设置的值。但是，由于它们访问的是一个`ThreadLocal`对象，因此这两个线程都无法看到对方保存的值。也就是说，它们存取的是两个不同的值。

---

## 2. 关于Inheritable ThreadLocal

> `InheritableThreadLocal`类是`ThreadLocal`类的子类。`ThreadLocal`中每个线程拥有它自己的值，与`ThreadLocal`不同的是，`InheritableThreadLocal`允许一个线程以及该线程创建的所有子线程都可以访问它保存的值

---

## 3. ThreadLocal原理

>  既然每个访问的ThreadLocal变量的线程都有自己的一个“本地”实例副本，每一个线程的`Thread`对象中都有一个`ThreadLocalMap`对象，该Map中，键是`ThreadLocal. threadLocalHashCode`，本地线程变量为值。 `ThreadLocal`对象就是当前线程的`ThreadLocalMap` 的访问入口，每一个`ThreadLocal` 对象都包含了一个独一无二的`threadLocalHashCode` 值，使用这个值就可以在线程K-V 值对中找回对应的本地线程变量。

---

## 4. ThreadLocalMap与内存泄漏问题

> 由于每个线程访问某 ThreadLocal 变量后，都会在自己的 Map 内维护该 ThreadLocal 变量与具体实例的映射，如果不删除这些引用（映射），则这些 ThreadLocal 不能被回收，可能会造成内存泄漏

- **解决方案**
  - 对于已经不再被使用且已被回收的 `ThreadLocal` 对象，它在每个线程内对应的实例由于被线程的 `ThreadLocalMap` 的 `Entry` 强引用，无法被回收，可能会造成内存泄漏
  - 针对该问题，`ThreadLocalMap` 的 set 方法中，通过 `replaceStaleEntry` 方法将所有键为 `null` 的 `Entry` 的值设置为 `null`，从而使得该值可被回收。另外，会在 rehash 方法中通过 `expungeStaleEntry` 方法将键和值为 `null` 的 Entry 设置为 `null` 从而使得该 `Entry` 可被回收。通过这种方式，`ThreadLocal` 可防止内存泄漏

---

## 5. 适用场景

如上文所述，`ThreadLocal` 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

对于第一点，每个线程拥有自己实例，实现它的方式很多。例如可以在线程内部构建一个单独的实例。`ThreadLocal` 可以以非常方便的形式满足该需求。

对于第二点，可以在满足第一点（每个线程有自己的实例）的条件下，通过方法间引用传递的形式实现。`ThreadLocal` 使得代码耦合度更低，且实现更优雅

---

## 6. 案例

>  对于 Java Web 应用而言，Session 保存了很多信息。很多时候需要通过 Session 获取信息，有些时候又需要修改 Session 的信息。一方面，需要保证每个线程有自己单独的 Session 实例。另一方面，由于很多地方都需要操作 Session，存在多方法共享 Session 的需求。如果不使用 ThreadLocal，可以在每个线程内构建一个 Session实例，并将该实例在多个方法间传递,耦合度太高。

> 使用 `ThreadLocal` 改造后的代码，不再需要在各个方法间传递 `Session` 对象，并且也非常轻松的保证了每个线程拥有自己独立的实例

---

## 7. 总结

- `ThreadLocal` 并不解决线程间共享数据的问题
- `ThreadLocal` 通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题
- 每个线程持有一个 Map 并维护了 `ThreadLocal` 对象与具体实例的映射，该 `Map` 由于只被持有它的线程访问，故不存在线程安全以及锁的问题
- `ThreadLocalMap` 的 `Entry` 对 `ThreadLocal` 的引用为弱引用，避免了 `ThreadLocal` 对象无法被回收的问题
- `ThreadLocalMap` 的 set 方法通过调用 `replaceStaleEntry` 方法回收键为 `null` 的 `Entry` 对象的值（即为具体实例）以及 `Entry` 对象本身从而防止内存泄漏
- `ThreadLocal` 适用于变量在线程间隔离且在方法间共享的场景

---

# 相关链接

[Java进阶（七）正确理解Thread Local的原理与适用场景](http://www.jasongj.com/java/threadlocal/)