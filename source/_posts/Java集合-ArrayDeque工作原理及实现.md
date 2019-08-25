---
title: Java集合-ArrayDeque工作原理及实现
date: 2018-06-26 10:01:51
toc: true
tags: 
 - Queue
 - 转载
categories: Java
---

> 【转载】[原文链接](https://yikun.github.io/2015/04/11/Java-ArrayDeque%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

<!--more-->

### 1. 概述

> Resizable-array implementation of the **Deque interface**. Array deques have no capacity restrictions; they **grow as necessary to support usage**. They are not thread-safe; in the absence of external synchronization, they do not support concurrent access by multiple threads. Null elements are prohibited. This class is likely to be faster than Stack when used as a stack, and faster than LinkedList when used as a queue.

![](http://cdn.briarbear.cn/201806261005_552.png)

> 以循环数组实现的双向Queue。大小是2的倍数，默认是16。
>
> 普通数组只能快速在末尾添加元素，为了支持FIFO，从数组头快速取出元素，就需要使用循环数组：有队头队尾两个下标：弹出元素时，队头下标递增；加入元素时，如果已到数组空间的末尾，则将元素循环赋值到数组[0](如果此时队头下标大于0，说明队头弹出过元素，有空位)，同时队尾下标指向0，再插入下一个元素则赋值到数组[1]，队尾下标指向1。如果队尾的下标追上队头，说明数组所有空间已用完，进行双倍的数组扩容。