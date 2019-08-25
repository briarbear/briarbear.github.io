---
layout: java
title: Java集合-EnumMap工作原理及实现
date: 2018-06-25 10:02:46
toc: true
tags: 
 - Map
 - 转载
categories: Java
---

> 【转载】 [原文链接](http://yikun.github.io/2015/04/24/Java-EnumMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/) - （部分修改与补充）

----

### 1. 概述

> A specialized Map implementation for use with enum type keys. All of the keys in an enum map must come from a single enum type that is specified, explicitly or implicitly, when the map is created. Enum maps are represented internally as arrays. This representation is extremely compact and efficient.

`EnumMap`是是一种键为枚举类型的特殊的`Map`实现。所有的`Key`也必须是一种枚举类型，`EnumMap`是使用数组来实现的。（*两个数组，一个数组`keyUniverse`存储`key`，另一个数组`vals`存储val，两个数组通过下标对应起来*）

<!-- more -->

```java
EnumMap<Course, String> map = new EnumMap<Course, String>(Course.class);
map.put(Course.ONE, "语文");
map.put(Course.ONE, "政治");
map.put(Course.TWO, "数学");
map.put(Course.THREE, "英语");
for(Entry<Course, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

输出结果（*按照枚举类中的顺序遍历数组，按数组下标依次输出（跳过空）*）为：

```shell
ONE: 政治
TWO: 数学
THREE: 英语
```

其具体实现的结构如下图所示：

![](http://cdn.briarbear.cn/201806251030_826.png)

### 2. put和get方法

put方法通过key的ordinal将值存储到对应的地方，get方法则根据key的ordinal获取对应的值。

```java
public V put(K key, V value) {
    // 类型检查
    typeCheck(key);
    // 获取key的序号
    int index = key.ordinal();
    Object oldValue = vals[index];
    // 赋值
    vals[index] = maskNull(value);
    // 若之前的值为空，则size++
    if (oldValue == null)
        size++;
    return unmaskNull(oldValue);
}

public V get(Object key) {
    return (isValidKey(key) ?
            unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
}
```

### 3. 遍历

EnumMapIterator的迭代这样实现的：

```java
public boolean hasNext() {
    while (index < vals.length && vals[index] == null)
        index++;
    return index != vals.length;
}

public Map.Entry<K,V> next() {
    if (!hasNext())
        throw new NoSuchElementException();
    lastReturnedEntry = new Entry(index++);
    return lastReturnedEntry;
}
```

通过`hasNext`跳过空的数组，也就是说，保证了遍历顺序与`Enum`中`key`的先后顺序一致。

### 参考资料

[What is EnumMap in Java](http://javarevisited.blogspot.jp/2012/09/what-is-enummap-in-java-example-tutorial.html)