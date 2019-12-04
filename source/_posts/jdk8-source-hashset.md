---
title: JDK8源码学习：HashSet
category:
  - Java
tags:
  - java
  - 源码
abbrlink: 8cf8f8a0
date: 2019-04-23 17:35:01
---
# 介绍
HashSet实现Set接口，由哈希表（实际上是HashMap实例）支持。它不能保证集合的迭代顺序，HashSet允许元素为null。并且该类不是线程安全的。它能保证元素唯一性，一般可以用于去重。
如果要线程安全，可以通过以下方式实现：
```java
Set s = Collections.synchronizedSet(new HashSet(...));
```

# 变量

# 构造方法
## public HashSet()
默认构造方法，是新建一个空的set,由HashMap支持，默认初始容量（16）和加载因子（0.75），对加载因子的解释在[HashMap源码](https://www.waynezw.cn/archives/2da9164.html)这篇文章中有解释。
```java
public HashSet() {
    map = new HashMap<>();
}
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}
```
`HashSet` 的构造方法都是调用 `HashMap` 的构造方法完成的。

# 方法
## public boolean add (E e)
添加元素，如果指定的元素尚不存在，则将其添加到此集合中，返回true，如果不存在，返回false。实现代码如下：
```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```
这里直接调用了` Map#put` 方法，方法有一个 `PRESENT`,它是一个虚拟的`object`值，实际上我们在添加元素的时候往 map里面put值的时候是没有value的，但是map的put方法是需要接收一个value值，这里就用到了`PRESENT`，一个默认的虚拟值。
因为 `HashSet#add` 方法实际上调用了` Map#put`方法，将元素 `e`当做了map的key值，这样就保证了`HashSet`元素的唯一性。该方法的具体实现可以看这篇文章[HashMap源码](https://www.waynezw.cn/archives/2da9164.html)。
我们通过下面的列子来印证。
```java 
Set<String> set = new HashSet<>();
boolean bool = set.add("hash set");
System.out.println(bool + ""); //  output: true
boolean bool2 = set.add("hash set");
System.out.println(bool2 + ""); // output: false
Iterator<String> iterator = set.iterator();
while (iterator.hasNext()) { // 3
    System.out.println(iterator.next()); 
}
```
通过上面的代码我们可以清楚的看见 步骤`3` 只输出了一次 `hash set`，证实了HashSet元素的唯一性。

## public boolean contains (Object o)
如果此set包含指定的元素，则返回true。
```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```
`contains`调用`HashMap#containsKey`方法，其实就是调用了`HashMap#getNode`， 该方法的具体实现可以看这篇文章[HashMap源码](https://www.waynezw.cn/archives/2da9164.html)。

## boolean remove(Object o)
删除元素，存在元素，并且删除成功，返回true，反之false；实现也是基于HashMap的方法。

## 遍历
HashSet推荐遍历的方式；
```java
public static void main(String[] args) {
    Set<Integer> set = new HashSet<>();
    for (int i = 0; i < 10; i++) {
        set.add(i);
    }
    forHashSet(set);
    whileHashSet(set);
}

private static void whileHashSet(Set<Integer> set) {
    Iterator<Integer> iterator = set.iterator();
    while (iterator.hasNext()) {
        System.out.println(iterator.next());
    }
}

private static void forHashSet(Set<Integer> set) {
    for (Iterator iterator = set.iterator();
         iterator.hasNext(); ) {
        System.out.println(iterator.next());
    }
}

```
`whileHashSet`和`forHashSet`本质上都是调用了 `iterator`，只是分别使用 for循环或者while来实现而已。

# 总结

* 基于HashMap的支持
* 元素唯一性
* 不保证元素添加的顺序
* 线程不安全类