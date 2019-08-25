---
title: Java集合-LinkedHashSet工作原理及实现
date: 2018-06-25 15:47:38
tags: 
 - Set
 - 转载
categories: Java
toc: true
---

> 【转载】[原文链接](http://yikun.github.io/2015/04/09/Java-LinkedHashSet%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

<!--more-->

### 1. 概述

> **Hash table and linked list implementation of the Set interface**, with predictable iteration order. This implementation differs from HashSet in that it maintains a doubly-linked list running through all of its entries. This linked list defines the iteration ordering, which is the order in which elements were inserted into the set (insertion-order). Note that insertion order is not affected if an element is re-inserted into the set. (An element e is reinserted into a set s if s.add(e) is invoked when s.contains(e) would return true immediately prior to the invocation.)

LinkedHashSet是基于HashMap和双向链表的实现。

``` java
LinkedHashSet<String> lset = new LinkedHashSet<String>();
lset.add("语文");
lset.add("数学");
lset.add("英语");
lset.add("历史");
lset.add("政治");
lset.add("地理");
lset.add("生物");
lset.add("化学");
for(String str : lset) {
    System.out.println(str);
}
```

利用链表来记录，保证了迭代输出的有序性。其具体结构如下所示：

![](http://cdn.briarbear.cn/201806251552_76.png)

可以看出，其实现基本和`LinkedHashMap`一样。

### 2. 关键实现

``` java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

从继承关系来看就知道`LinkedHashMap`的实现非常简单，就是集成`HashSet`的接口，并且在构造时调用的是：

``` java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

因此，结构也便是如`HashSet`于`HashMap`一样，`LinkedHashSet`也便如`LinkedHashMap`一样，只是将`Value`做了一个`dummy`的`object`。
### 参考资料

[LinkedHashSet](http://docs.oracle.com/javase/7/docs/api/java/util/LinkedHashSet.html)