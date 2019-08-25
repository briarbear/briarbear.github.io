---
title: Java集合-TreeSet工作原理及实现
date: 2018-06-25 15:57:27
toc: true
tags:
 - Set
 - 转载
categories: Java
---

> 【转载】 [原文链接](http://yikun.github.io/2015/04/10/Java-TreeSet%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

### 1.概述

<!--more-->

> A NavigableSet implementation based on a **TreeMap**. The elements are ordered using their natural ordering, or by a Comparator provided at set creation time, depending on which constructor is used.This implementation provides guaranteed **log(n)** time cost for the basic operations (add, remove and contains).

TreeSet是基于TreeMap实现的，也非常简单，同样的只是用key及其操作，然后把value置为dummy的object。

``` java
TreeSet<String> tset = new TreeSet<String>();
tset.add("1语文");
tset.add("3英语");
tset.add("2数学");
tset.add("4政治");
tset.add("5历史");
tset.add("6地理");
tset.add("7生物");
tset.add("8化学");
for(String str : tset) {
    System.out.println(str);
}
```

其具体的结构是：

![](http://cdn.briarbear.cn/201806251559_135.png)

利用`TreeMap`的特性，实现了set的有序性(通过红黑树实现)。
### 参考资料

[TreeSet(Java Platform SE 8)](http://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html)