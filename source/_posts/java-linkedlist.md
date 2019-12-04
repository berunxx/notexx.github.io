---
title: JDK8源码学习：LinkedList
category:
  - Java
tags:
  - java
abbrlink: e3ffdb91
date: 2018-11-14 10:29:02
---
# 概述
LinkedList是允许元素为null的**双向链表**，LinkedList是**线程不安全**的，底层是基于链表的实现。
因为底层是链表的原因，说明，插入、删除比较方便，只需要移动对应的指针即可。但是在处理随机访问的数据，就会要比ArrayList慢了。
# 属性
LinkedList的属性相对比较少，只有下面的三个属性。
``` java
// 集合元素的数量
transient int size = 0;
// 链表的第一个指针元素
transient Node<E> first;
// 链表最后一个元素
transient Node<E> last;
```
# 构造方法
LinkedList只有两个构造方法，一个时默认的构造方法，另一个是将集合初始化到LinkedList。
``` java
/**
 * 初始化一个空的LinkedList
 */
public LinkedList() {
}

/**
 * 初始化一个包含集合元素的LinkedList
 */
public LinkedList(Collection<? extends E> c) {
    this();  // 调用默认构造方法
    addAll(c); // 将元素全部添加到LinkedList
}
```
在看 `addAll()`方法前，先看看Node节点结构
``` java
private static class Node<E> {
    E item; // 节点值
    Node<E> next; // 下一个节点
    Node<E> prev; // 前一个节点

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
看完了Node的结构，随便把`node(int)`方法给看了，该方法在文中会多次遇到。
``` java
Node<E> node(int index) {
    // assert isElementIndex(index);
    // size >> 1 等价于 size/2 。通过将LinkedList折半，增加查询效率。
    if (index < (size >> 1)) {
        Node<E> x = first; // 保存第一个节点，因为需要从第一个节点开始循环查找
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
`node(int index)`的实现是一个一个循环遍历得到的值，虽然说用到了缩小范围的折半方式优化查询。但是还是不可避免的导致获取效率低下。

在初始化有集合元素的构造方法是会调用到 `addAll()` 方法，跟着 `addAll()` 方法一直看下去，核心实现如下：
``` java
public boolean addAll(int index, Collection<? extends E> c) {
    // 检验index是否在范围内 [0, size]
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length; // 记录传递过来的集合元素个数
    if (numNew == 0)
        return false;

    Node<E> pred, succ; // pred: index前置节点, index的后置节点
    // 在链表尾部添加数据
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index); // 获取index节点的值，赋值给succ
        pred = succ.prev; // succ的前置节点赋值给 pred
    }

    // 遍历添加元素
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
         // 根据前置节点和当前元素之构建 node节点
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null) // 没有前置节点
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    // 判断添加元素是否在队尾添加
    if (succ == null) {
        last = pred; // 设置队尾节点
    } else { // 在链表中间添加的元素
        pred.next = succ; // 设置前置节点的下一个值
        succ.prev = pred; // 设置后置节点的前一个值
    }

    size += numNew; // 重新设置LinkedList的元素数量
    modCount++; // 记录一次结构更改的次数
    return true;
}
```
`addAll()`方法的实现过程还是容易看懂的。首先检查插入的位置是否在范围之内，接着判断是在链表中间插入数据，还是在链表尾部插入数据，设置根据succ和pred。然后循环遍历添加节点到LinkedList中。整个过程大致就是这样
# 常用方法
接下来看看LinkedList中的常用方法
## add 添加
LinkedList的添加方法，主要是 `add(E)`、 `add(int, E)`，首先来看看默认的`add(E)`方法的代码：
``` java
public boolean add(E e) {
    linkLast(e);
    return true;
}
```
上面代码没有具体逻辑，具体实现在`linkLast(e)`中，接着进入该方法中。
``` java
void linkLast(E e) {
    final Node<E> l = last; // 将末尾节点设置给l节点
    // 根据末尾节点做前置节点，得到一个新的节点
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode; // 更新末尾节点为新的节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```
`linkLast`方法实现过程是，首先得到末尾节点`l`，根据`l`做前置节点得到一个新的节点，最后对末尾节点重新赋值。我们可以看出，整个`add`方法在不指定添加位置的情况下，默认将值添加到末尾的。
看了默认的add方法实现，接着看看在规定位置添加节点的实现
`add(int, E)`方法的实现
``` java
public void add(int index, E element) {
    // 校验添加节点的位置是否在 [0, size] 之间
    checkPositionIndex(index);
    // 如果添加的位置刚好尾部节点，直接调用默认添加方法的实现。
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```
接着进入`linkBefore`方法看看是怎么实现的。e: 待添加元素值，succ: 带添加位置的节点
``` java
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev; // 保存待添加位置的前置节点
    final Node<E> newNode = new Node<>(pred, e, succ); // 根据前置节点、当前值、后置节点 得到一个新的节点
    succ.prev = newNode; // 设置添加位置的前置节点
    if (pred == null)
        first = newNode; // 如果前置节点为空，直接设置 第一个节点 （first代表第一个节点）
    else
        pred.next = newNode; // 将前面保存的前置节点的下一节点设置为新的节点
    size++; // 将LinkedList的容量加一
    modCount++; // 记录一下更改的次数
}
```
整个过程通过代码中的注释，大致都能看的明白。
在添加方法中还有`addLast`、`addFrist`方法。首先看看`addLast`方法
``` java
public void addLast(E e) {
    linkLast(e);
}
```
实现过程和默认的add方法一样，唯一的区别就是默认`add`方法有返回值，`addLast`方法没有返回值。
接下来看看`addFrist(E)`方法。
``` java
public void addFirst(E e) {
    linkFirst(e);
}
// 具体实现
private void linkFirst(E e) {
    final Node<E> f = first; // 保存原来的第一个节点
    final Node<E> newNode = new Node<>(null, e, f); // 得到一个新的节点，新节点的下一个节点就是原来的第一个节点
    first = newNode;
    if (f == null)
        last = newNode; // 说明之前为空的LinkedList，直接设置末尾节点为新节点
    else
        f.prev = newNode; // 设置原来第一个节点的前置节点为新节点
    size++; // 容量+1
    modCount++; // 修改次数 +1
}
```
##  remove 删除
删除有两个实现，其一是：`remove()`不需要传入指针的位置，默认删除第一个，其二是`remove(int)`方法，该方法是传入一个需要删除的位置，如果删除成功会返回被删除的值。
`remove()`方法的实现过程和`remove(int)`差不多，也可以理解为`remove()`默认传递了指针为0。这里就不单独介绍了。接下来就看看`remove(int)` 是怎么实现的。
代码如下：
``` java
public E remove(int index) {
    // 检验index是否在[0, size]之间
    checkElementIndex(index);
    // node(index)：待删除节点
    return unlink(node(index));
}
```
核心实现在unlink方法中，接下来看看 unlink的实现
``` java
E unlink(Node<E> x) {
    // assert x != null;
    // x 为待删除节点
    final E element = x.item; // 保存一下值，删除完成之后，需要返回给调用者。
    final Node<E> next = x.next; // 保存x的下一个节点
    final Node<E> prev = x.prev; // 保存x的前一个节点
    // 如果前置节点为空
    if (prev == null) {
        // 直接设置LinkedList的frist节点为 x的下一个节点
        first = next;
    } else {
        // 设置x的前置节点的下一个节点为 x的后置节点
        prev.next = next;
        x.prev = null; // 将x的前置节点设置为null，方便gc
    }

    if (next == null) {
        last = prev; // 如果x的后置节点为空，LinkedList的末尾节点设置为x的前置节点
    } else {
        next.prev = prev; // 设置x的下一个节点的前置节点为 x的前置节点
        x.next = null; // 将x的后置节点设置为null，方便gc
    }

    x.item = null; // 将x值设置为null，方便gc
    size--; // 容量 -1
    modCount++;
    return element;
}
```
删除方法的核心就是，将待删除节点的前置节点的下一个节点设置为待删除节点的下一个节点（读起来蛮拗口的，多读几遍就能理解了）。
## get
`get(int)`方法，根据位置获取值。
先贴上代码
``` java
public E get(int index) {
    // 检验index是否在[0, size]之间
    checkElementIndex(index);
    return node(index).item;
}
```
上面已经有对node(index)方法做过解释了，该方法就是得到节点，返回节点的值。

## set
`set(int, E)`方法是一个根据index替换其原来的值，最后返回原始值。
``` java
public E set(int index, E element) {
    // 检验index是否在[0, size]之间
    checkElementIndex(index);
    // 根据index获取需要设置的节点
    Node<E> x = node(index);
    // 保存旧值
    E oldVal = x.item;
    // 设置新值
    x.item = element;
    // 返回旧值
    return oldVal;
}
```
## poll
`poll()`方法是获取并删除第一个元素。其实就是删除第一个元素并返回删除元素的值和`remove()`方法相同。这里也不做过多解释了。
``` java
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

## peek
`peek()` 获取第一个节点的值。和`poll()`唯一不同的是，`peek`不需要删除第一个节点。
实现如下：
``` java
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

# 总结
LinkedList的分析到这里就结束了。
LinkedList是一个双向列表，在删除，添加元素方面有着比ArrayList更好的体验。但是在查询的时候需要一个一个遍历得到结果，虽说在遍历的时候将LinkedList分为了前一半和后一半遍历，但是和ArrayList的底层动态数组相比，还是差的太多。所以在项目具体使用过程中，还是得按需使用。
