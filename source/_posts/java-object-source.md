---
title: Java源码学习：Object
date: 2018-04-03
category: "java" 
tags: 
  - java
---
# 简介
Object类是所有类根，即所有的类都隐式继承了Object.
# 常用方法
## hashCode()
hashCode方法返回对象的散列码，相等对象必须返回相等的hashCode，不同对象的hashCode尽可能不相等；Obejct中定义的hashCode方法为：
```
public native int hashCode();
```
## equals()
Object中定义的equals()方法：
```java
public boolean equals(Object obj) {
	return (this == obj);
}
```
equals()方法，顾名思义就是判断是否相等，可以看见Object中equals()方法是用 ‘==’来判断地址;由此可见，我们常用String类的肯定是重写了该方法的，查看String的源码；
```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
当我们重写equals时总要重写hashCode：
* 重写equals不重写hashCode，会导致“不相等对象拥有相同的hashCode”，导致集合类HashMap，HashSet和Hashtable无法工作；极端情况下，在散列表中使所有对象的hashCode都相等，所有对象都被映射到同一个桶中，散列表退化成链表；
* 当两个对象调用equal返回true，则两个对象各自调用hashCode()返回相同hashCode；
* 当两个对象调用equal返回false， 两个对象各自调用hashCode()返回的hashCode可以相同（散列冲突不能完全避免）

## toString()
Object类中toString方法，输出对象的“对象类名@散列码”；
```java
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

## finalize()
Obejct类中finalize()方法
```java
protected void finalize() throws Throwable { }
```
finalize()会在对象被垃圾回收时由垃圾回收器调用，垃圾对象是指没有引用指向的对象
* JVM的垃圾回收是"最少回收"方式，只有当内存不够的时候才会进行垃圾回收
* 如果调用System.gc() 这个方法，只是告诉JVM 希望这里进行垃圾回收，但是具体什么时候回收还需要看JVM的运行状态，且System.gc()对资源还是有一定消耗，如果盲目的运用System.gc()这个方法，反而效率还会下降，看场景适用；

# Reference
https://blog.csdn.net/xu511739113/article/details/52328727
https://segmentfault.com/a/1190000009057426