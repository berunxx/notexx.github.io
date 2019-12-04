---
title: JDK8源码学习：ArrayList 源码解析
category: Java相关
tags:
  - java
abbrlink: 9e8769e6
date: 2018-02-11 00:00:00
---
# 概述
* 以数组实现。节约空间，但数组有容量限制。超出限制时会增加50%容量，用System.arraycopy（）复制到新的数组。因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。
* 按数组下标访问元素－get（i）、set（i,e） 的性能很高，这是数组的基本优势。

* 如果按下标插入元素、删除元素－add（i,e）、 remove（i）、remove（e），则要用System.arraycopy（）来复制移动部分受影响的元素，性能就变差了。

* 越是前面的元素，修改时要移动的元素越多。直接在数组末尾加入元素－常用的add（e），删除最后一个元素则无影响。

# 类结构
ArrayList是Java集合框架List接口的实现类，
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
       // 省略
}
```

通过代码我们看出，ArrayList继承AbstractList方法，并且在实现了List方法的同时，还实现了RandomAccess、Cloneable（克隆）、Serializable（序列化）；
ArrayList类图如下所示：
![](http://qxu1146470209.my3w.com/wordpress/wp-content/uploads/2018/02/49feecd1bc989fa98b6aad419ea8db21.png)

# 源码分析

## 变量定义
```java
	/**
	* 默认初始化容量大小为10
     */
    private static final int DEFAULT_CAPACITY = 10;
    /**
     * 共用的数组实例用于空的实例
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 默认大小的空实例所用的共用的数组实例。我们判断当一个元素被加入时数组实例该怎样扩大
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * elementData存储ArrayList内的元素
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * 存储在ArrayList内的容量大小
     *
     * @serial
     */
    private int size;
```
### 构造方法
```java
	/**
     * 构造一个空的集合，并规定初始化容量
     * @param  initialCapacity  该集合需要初始化的容量
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
			// 新建一个规定大小容量的集合赋值给 elementData
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
			// 空的集合赋给 elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
			// 传入变量 非法，抛出异常。
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     *  构造一个空的集合并初始化容量为10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定元素的集合，按集合的顺序返回使用迭代器。
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array. 替换为空
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
###  添加方法 add()
```java
	/**
     * 添加规定的根元素到集合的最后
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
在add之前会调用ensureCapacityInternal()方法，判断是否需要扩容
```java
	private void ensureCapacityInternal(int minCapacity) {
		// 如果elementData 为空 ，指定elementData的最小默认容量。如果为第一次添加，默认容量为10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
	private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code 如容量不足，进行扩容。
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
   /**
     * 增加容量，确保至少可以支持最小规定元素的数量
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
* 从grow方法中可以看出，ArrayList的elementData数组如遇到容量不足时，将会把新容量newCapacity设置为 oldCapacity + (oldCapacity >> 1)。二进制位操作>> 1等同于/2的效果，扩容导致的newCapacity也就设置为原先的1.5倍。
* 如果新的容量大于MAX_ARRAY_SIZE。将会调用hugeCapacity将int的最大值赋给newCapacity。不过这种情况一般不会用到，很少会用到这么大的ArrayList。
* 在确保有容量的情况下，会将元素添加至elementData数组中。

### 移除方法 remove()
```java
	/**
     * 删除列表中指定的元素 并将index 后面的元素移动位置
     */
    public E remove(int index) {
	//判断index 是否在 list大小范围之内
        rangeCheck(index);

        modCount++;
		// 根据index 取出需要移除的数据 赋值oldValue
        E oldValue = elementData(index);
		// 需要移动的大小
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
* 该方法是根据元素位置移除列表数据，首先判断index是否在列表元素的范围之内，将需要移除的数据，赋值给oldvalue存储，计算出该元素之后的列表数量，使用System.arraycopy()方法，该方法的作用是：从指定的源数组中复制一个数组，从指定的位置，到目标数组的指定位置。返回oldValue;
