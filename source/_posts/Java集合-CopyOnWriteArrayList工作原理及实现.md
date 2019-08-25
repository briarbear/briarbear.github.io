---
title: Java集合-CopyOnWriteArrayList工作原理及实现
date: 2018-06-26 09:57:40
toc: true
tags: 
 - List
 - 转载
categories: Java
---

> 【转载】[原文链接](https://yikun.github.io/2015/04/28/Java-CopyOnWriteArrayList%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

> 并发优化的`ArrayList`。用`CopyOnWrite`策略，在修改时先复制一个快照来修改，改完再让内部指针指向新数组。
>
> 因为对快照的修改对读操作来说不可见，所以只有写锁没有读锁，加上复制的昂贵成本，典型的适合读多写少的场景。如果更新频率较高，或数组较大时，还是`Collections.synchronizedList(list)`，对所有操作用同一把锁来保证线程安全更好。
>
> 增加了`addIfAbsent(e)`方法，会遍历数组来检查元素是否已存在，性能可想像的不会太好。

