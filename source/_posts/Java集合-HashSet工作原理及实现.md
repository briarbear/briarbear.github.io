---
title: Java集合-HashSet工作原理及实现
date: 2018-06-25 15:36:38
toc: true
tags:
 - Set
 - 转载
categories: Java
---

> 【转载】 [原文链接](http://yikun.github.io/2015/04/08/Java-HashSet%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)



<!--more-->

### 1. 概述

> This class implements the Set interface, backed by a hash table (actually a **HashMap** instance). It makes no guarantees as to the iteration order of the set; in particular, it does not guarantee that the order will remain constant over time. This class permits the null element.

HashSet是基于HashMap来实现的，操作很简单，更像是对HashMap做了一次“封装”，而且只使用了HashMap的key来实现各种特性，我们先来感性的认识一下这个结构：

``` java
HashSet<String> set = new HashSet<String>();
set.add("语文");
set.add("数学");
set.add("英语");
set.add("历史");
set.add("政治");
set.add("地理");
set.add("生物");
set.add("化学");
```

其大致的结构是这样的：

![](http://cdn.briarbear.cn/201806251540_236.png)

``` java
private transient HashMap<E,Object> map;
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```

`map`是整个HashSet的核心，而`PRESENT`则是用来造一个假的`value`来用的。
### 2. 基本操作

``` java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

public boolean contains(Object o) {
    return map.containsKey(o);
}

public int size() {
    return map.size();
}
```

基本操作也非常简单，就是调用`HashMap`的相关方法，其中`value`就是之前那个`dummy`的`Object`。所以，只要了解 `HashMap`的实现就可以了。
### 参考资料

[HashSet(Java Platform 8)](http://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html)