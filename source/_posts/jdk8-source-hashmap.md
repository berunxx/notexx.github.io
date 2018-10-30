---
title: JDK8源码学习:HashMap
date: 2018-09-03
category: "java" 
tags: 
  - java
---
# 概述
HashMap是Java的一个集合类，是我们在开发中经常使用的。本文记录个人阅读源码的一些步骤和理解。阅读步骤大致为：变量-->构造方法-->常用方法。
在JDK7中，HashMap的底层数据结构为：数组+链表的形式。
![](http://www.myluffy.com/blog/wp-content/uploads/2018/09/eb52ee72faf1954da936f71a5b43c232.png)
在JDK8中，HashMap的底层数据结构为：数组+链表+红黑树（TreeNode），增加红黑树的结构。
![](http://www.myluffy.com/blog/wp-content/uploads/2018/09/38ce4ff1cf9ece2b706242424fb6ad2c.png)
# 变量
- loadFactor 加载因子
默认加载因子为`0.75f`,为何是`0.75` 而不是其他的值呢？
首先理解一下，什么是加载因子。
加载因子是表示Hsah表中元素的填满的程度。如果加载因子越大,填满的元素越多，所以空间利用率提高，但是冲突的机会加大了。反之亦然
冲突的机会越大，查找所需要的成本增加，查找时间也相应的增加 了。反之亦然。
结合前面两条，我们必须在 "冲突的机会"与"空间利用率"之间寻找一种平衡与折衷。这种平衡与折衷本质上是数据结构中有名的"时间复杂度-空间复杂度"矛盾的平衡与折衷。
加载因子的值是可以大于1的。
- threshold
threshold表示当HashMap的size大于threshold时会执行resize操作。
- size
记录数组的长度
- modCount
modCount是记录HashMap发送结构性变化的次数，比如扩容、rehash。

另外大概了解一下HashMap的最大容量 `1<<30`也就是2的30次方，初始化容量为16。

# 构造方法
- HashMap(int, float)
为了方便阅读，注释直接写在了代码里面。
```java
public HashMap(int initialCapacity, float loadFactor) {
	if (initialCapacity < 0)
		throw new IllegalArgumentException("Illegal initial capacity: " +
										   initialCapacity);
	// 初始化容量必须在 1<<30 以内
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	if (loadFactor <= 0 || Float.isNaN(loadFactor))
		throw new IllegalArgumentException("Illegal load factor: " +
										   loadFactor);
	this.loadFactor = loadFactor;
		// 调用了tableSizeFor方法，该方法是返回给定的`initialCapacity`值的向上取最近的2的幂。比如传递的值为12，返回16。
	this.threshold = tableSizeFor(initialCapacity);
}
```
- HashMap(int)
```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
该构造方法，只是指定了初始化容量，使用默认的加载因子，调用`HashMap(int, float)`方法。
```
# 常用方法
## put 插入
put方法主要的实现过程如下，为了方便阅读，将注释写在了代码中。
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
	Node<K,V>[] tab; Node<K,V> p; int n, i;
	if ((tab = table) == null || (n = tab.length) == 0)
		// 如果table为空，调用resize()方法，初始化一个table。
		n = (tab = resize()).length;
	if ((p = tab[i = (n - 1) & hash]) == null)
		// 该节点不存在，新建节点
		tab[i] = newNode(hash, key, value, null);
	else {
		Node<K,V> e; K k;
		if (p.hash == hash &&
			((k = p.key) == key || (key != null && key.equals(k))))
			e = p;
		else if (p instanceof TreeNode) // 如果是p节点是红黑树节点，调用红黑树的put方法。
			e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
		else {
			for (int binCount = 0; ; ++binCount) {
				if ((e = p.next) == null) { // 找到链表的最后一个节点，插入新的节点
					p.next = newNode(hash, key, value, null);
					// 如果binCount的大小大于等于TREEIFY_THRESHOLD-1（TREEIFY_THRESHOLD默认为8），调用treeifyBin方法，后面单独介绍该方法。
					if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
						treeifyBin(tab, hash);
					break;
				}
				// 链表中存在该节点，跳出循环。
				if (e.hash == hash &&
					((k = e.key) == key || (key != null && key.equals(k))))
					break;
				p = e;
			}
		}
		if (e != null) { // existing mapping for key
			V oldValue = e.value;
			// 根据oblyIfAbsent是否更新值
			if (!onlyIfAbsent || oldValue == null)
				e.value = value;
			afterNodeAccess(e);
			return oldValue;
		}
	}
	// 修改 modCount
	++modCount;
	// 如果table大小大于了阈值，则需要扩容。
	if (++size > threshold)
		resize();
	afterNodeInsertion(evict);
	return null;
}
```
上面已经介绍了put方法的主流程，接下来分析一下该方法中留下的几个问题。
- treeifyBin方法

```java
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
			// 判断是否需要扩容，MIN_TREEIFY_CAPACITY的值：64,也就是说在大小为16、 32 的时候，不要进行结构转换
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
				// 将节点转为树节点
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p; // 将p树节点指向hd树节点
                else {
                    p.prev = tl; // 当前p树节点指向 p树节点的前一树节点
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```
treeifyBin方法主要是把容器里的元素变成树结构。当HashMap的内部元素数组中某个位置上存在多个hash值相同的键值对，这些Node已经形成了一个链表，当该链表的长度大于等于7的时候，会调用该方法来进行一个特殊处理。

## get 取值
源码中get方法代码如下，为了方便阅读，在代码中会写上相应的注释。

```java
public V get(Object key) {
	Node<K,V> e;
	return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```
这段代码比较简单，调用了getNode()方法，并传入`hash(key)`和`key`，所以说取值的过程在`getNode`中，下面看看该方法的具体实现：
```java
final Node<K,V> getNode(int hash, Object key) {
	// 定义一个新的table数组、首节点
	Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
	// 判断table数组是否为空，并且根据hash值算出 tab[(n - 1) & hash]是否为空，其中一个条件为空，说明key没有对应的value值。
	if ((tab = table) != null && (n = tab.length) > 0 &&
		(first = tab[(n - 1) & hash]) != null) {
		// 判断首节点的hash和key是否都相等，如果都等，直接返回首节点
		if (first.hash == hash && // always check first node
			((k = first.key) == key || (key != null && key.equals(k))))
			return first;
		// 走到这儿，说明是一个链表了或者红黑树了
		if ((e = first.next) != null) {
			// 判断是否为红黑树
			if (first instanceof TreeNode)
				// 调用 红黑树的getTreeNode方法
				return ((TreeNode<K,V>)first).getTreeNode(hash, key);
			// 遍历链表一直找到匹配的值为止
			do {
				if (e.hash == hash &&
					((k = e.key) == key || (key != null && key.equals(k))))
					return e;
			} while ((e = e.next) != null);
		}
	}
	return null;
}
```
`getNode`方法的过程相对而言是比较简单的，上面注释基本上比较易懂的，在整个流程中，红黑树的取值方法，没有说到，接下来看看`getTreeNode`方法中主要流程。
```java
final TreeNode<K,V> getTreeNode(int h, Object k) {
	return ((parent != null) ? root() : this).find(h, k, null);
}
```
`getTreeNode`方法的实现过程如下：
```java
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
	TreeNode<K,V> p = this;
	do {
		// ph: p节点的hash值，pk:p节点的key值
		int ph, dir; K pk;
		// pl: p节点的左节点， pr：p节点的右节点
		TreeNode<K,V> pl = p.left, pr = p.right, q;
		if ((ph = p.hash) > h)
			// p节点的hash值大于了 h（h是待get值的hash值），将左节点赋值给p节点
			p = pl;
		else if (ph < h)
			// p节点的hash值小于了 h（h是待get值的hash值），将右节点赋值给p节点
			p = pr;
		else if ((pk = p.key) == k || (k != null && k.equals(pk)))
			// 走到了这儿，说明p.hash==h,只需要匹配key是否相等就好了。
			return p;
		else if (pl == null)
			// 左节点为空，将右节点赋值给p节点
			p = pr;
		else if (pr == null)
			// 右节点为空，将左节点赋值给p节点
			p = pl;
		// kc参数在首次使用比较键时缓存equivalentClassFor（key）。
		// comparableClassFor方法：只有当传入对象的运行时类型符合“class C implements Cormparable <C>”，则返回k的Class，否则返回null。
		// compareComparables方法: 如果pk匹配kc（k的筛选可比类），则返回k.compareTo（pk），否则返回0。
		else if ((kc != null ||
				  (kc = comparableClassFor(k)) != null) &&
				 (dir = compareComparables(kc, k, pk)) != 0)
			p = (dir < 0) ? pl : pr;
		else if ((q = pr.find(h, k, kc)) != null)
			return q;
		else
			p = pl;
	} while (p != null);
	return null;
}
```
在本文中，主要是对常用的方法get、put做了一个学习了解。