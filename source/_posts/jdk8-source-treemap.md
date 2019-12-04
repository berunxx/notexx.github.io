---
title: 'JDK8源码学习:TreeMap'
category: Java
tags:
  - java
  - 源码
abbrlink: 3a2cad14
date: 2018-09-07 00:00:00
---
# 概述
本文是记录学习，文中有理解错误的地方，请指出共同探讨改正。
前面介绍了[HashMap](2da9164.html "HashMap")，因为HashMap是一种无序的存储集合，当某些时候需要特定的存储顺序的时候，就只能另寻他法了，在jdk中为我们提供了LinkedHashmap和TreeMap以供我们使用，本文先介绍TreeMap。
TreeMap和HashMap一样都是继承至AbstractMap，并且实现了NavigableMap()，TreeMap是在NavigableMap基础上基于红黑树的实现，他是一种顺序的存储结构。
TreeMap数据结构为Entry，Entry简单实现如下：
```java
static final class Entry<K,V> implements Map.Entry<K,V> {
	K key;
	V value;
	Entry<K,V> left; // 左节点
	Entry<K,V> right; // 右节点
	Entry<K,V> parent; // 父节点
	boolean color = BLACK;
	// ... 其他代码省略
}
```
# 常用变量
- root
root的定义为：`private transient Entry<K,V> root`，可以理解为一个短暂的 Entry
- size
size的定义为：`private transient int size = 0`， 记录树中的数量
- modCount
modCount的定义为：`private transient int modCount = 0`，记录结构性发生变化的次数，比如删除节点。
- comparator
comparator的定义为：`private final Comparator<? super K> comparator`,用于维护此树形图中的顺序，如果使用其键的自然顺序，则comparator为空。

# 构造方法
## 无参
- public TreeMap()
代码如下：
```java
public TreeMap() {
	comparator = null;
}
```
构造方法，没有指定comparator，所以使用它的自然顺序排序。

## 有参
- **public TreeMap(Comparator<? super K> comparator)**
代码如下：
```java
public TreeMap(Comparator<? super K> comparator) {
	this.comparator = comparator;
}
```
构造方法，使用给定的comparator规则排序

- **public TreeMap(Map<? extends K, ? extends V> m)**
代码如下：
```java
public TreeMap(Map<? extends K, ? extends V> m) {
	comparator = null;
	putAll(m);
}
```
构造一个新的 tree map，其中包含给定map的相同的映射，根据key的自然顺序进行排序，插入的新map的所有key必须实现 Comparable接口

- **public TreeMap(SortedMap<K, ? extends V> m)**
实现代码如下：
```java
public TreeMap(SortedMap<K, ? extends V> m) {
	// 将SoretdMap的排序方法赋给comparator
	comparator = m.comparator();
	try {
		// 构建 tree map
		buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
	} catch (java.io.IOException cannotHappen) {
	} catch (ClassNotFoundException cannotHappen) {
	}
}
```
构造一个新的 tree map，其中包含给定map的相同的映射，并且使用给定的 sorted map 的排序方式进行排序

# 常用方法
## put() 插入
插入方法是我们开发常用的方法，接下来看看，TreeMap的put方法具体是怎么实现的，为了方便阅读部分注释直接写在了代码中，代码如下：
```java
public V put(K key, V value) {
	Entry<K,V> t = root;
	if (t == null) {
		// 比较两个key值，使用此时正确的compare方法。
		compare(key, key); // type (and possibly null) check
		// new 一个 entry节点
		root = new Entry<>(key, value, null);
		size = 1;
		modCount++; // 增加结构变化的次数
		return null;
	}
	int cmp;
	Entry<K,V> parent;
	// split comparator and comparable paths
	Comparator<? super K> cpr = comparator;
	if (cpr != null) {
		// 设置value到特定的位置
		do {
			parent = t;
			cmp = cpr.compare(key, t.key); // 比较 key 和 t.key
			if (cmp < 0)
				t = t.left;
			else if (cmp > 0)
				t = t.right;
			else
				return t.setValue(value); // cmp=0，设置value
		} while (t != null);
	}
	else {
		if (key == null) // 不允许key为null
			throw new NullPointerException();
		@SuppressWarnings("unchecked")
			Comparable<? super K> k = (Comparable<? super K>) key;
		// 设置value到特定的位置
		do {
			parent = t;
			cmp = k.compareTo(t.key);
			if (cmp < 0)
				t = t.left;
			else if (cmp > 0)
				t = t.right;
			else
				return t.setValue(value);
		} while (t != null);
	}
	// 走到了这里， 说明了，cmp != 0，没有找到对应的key值，新建一个entry e，并将e放在相应的parent左右节点下面
	Entry<K,V> e = new Entry<>(key, value, parent);
	if (cmp < 0)
		parent.left = e;
	else
		parent.right = e;
	fixAfterInsertion(e);
	size++;
	modCount++;
	return null;
}
```
put方法还是比较容易能理解的，首先判断root是否为空，如果没空，直接new Entry即可。不为空，根据comparator的值，查找要设置value的位置。如果没有找到匹配的key，则新建一个Entry e，再根据cmp的值，将Entry e设置到对应的位置即可。
## get 获取值
根据key获取一个Entry，具体实现如下：
```java
final Entry<K,V> getEntry(Object key) {
	// Offload comparator-based version for sake of performance
	if (comparator != null)
		// 如果默认comparator不为空，调用getEntryUsingComparator方法
		return getEntryUsingComparator(key);
	if (key == null) // key不允许为null
		throw new NullPointerException();
	// 走到了这里，说明comparator为空，使用默认排序方法。
	@SuppressWarnings("unchecked")
		Comparable<? super K> k = (Comparable<? super K>) key;
	// 将当前root赋值给p
	Entry<K,V> p = root;
	// 循环遍历p
	while (p != null) {
		// 通过compareTo方法比较key与p的key
		int cmp = k.compareTo(p.key);
		if (cmp < 0)
			p = p.left; // 将p.left赋值给p
		else if (cmp > 0)
			p = p.right; // 将p.right赋值给p
		else
			return p; // 说明key与p.key相等，返回当前p节点
	}
	// 如果p节点中，没有找到对应的key，返回null
	return null;
}
```
获取Entry的方法的过程大致为：根据comparator获取Entry，其实就是遍历root，查找比较key，有匹配的返回对应的entry即可。注意key值不允许为空，会抛出空指针异常。
在获取Entry的方法中，如果comparator不为空，则使用getEntryUsingComparator方法获取。实现如下：
```java
final Entry<K,V> getEntryUsingComparator(Object key) {
	@SuppressWarnings("unchecked")
	K k = (K) key;
	Comparator<? super K> cpr = comparator;
	if (cpr != null) {
		Entry<K,V> p = root;
		while (p != null) {
			// 通过自定义的compare方法比较key与p的key
			int cmp = cpr.compare(k, p.key);
			if (cmp < 0)
				p = p.left; // 将p.left赋值给p
			else if (cmp > 0)
				p = p.right; // 将p.right赋值给p
			else
				return p;// 说明key与p.key相等，返回当前p节点
		}
	}
	// 如果p节点中，没有找到对应的key，返回null
	return null;
}
```
实现过程和上面差不多，只是比较key值的方法换了而已。

## firstKey 
获取第一个key
```java
public K firstKey() {
	// getFirstEntry得到第一个Entry，调用key(),得到key值。
	return key(getFirstEntry());
}
该方法比较简单，就不细说了。
```
## lastKey 
获取最后一个key
```java
public K lastKey() {
	// getLastEntry得到最后一个Entry，调用key(),得到key值。
	return key(getLastEntry());
}
```
lastKey的具体实现过程在getLastEntry中，实现如下：
```java
final Entry<K,V> getLastEntry() {
	Entry<K,V> p = root;
	if (p != null)
		while (p.right != null)
			p = p.right;
	return p;
}
```
实现方法和firstKey差不多。一直找到最右节点为止。

## remove 删除
具体代码实现如下：
```java
// 根据key删除entry节点，返回value
public V remove(Object key) {
	// 根据key得到对应entry节点
	Entry<K,V> p = getEntry(key);
	if (p == null)
		return null;

	V oldValue = p.value;
	// 删除entity，后面会介绍
	deleteEntry(p);
	return oldValue;
}
```
首先根据key查找Entry，如果不为空，则调用deleteEntry方法删除。
deleteEntry方法是实现如下：
```java
/**
 * Delete node p, and then rebalance the tree.
 * 删除节点p，从新平衡树
 */
private void deleteEntry(Entry<K,V> p) {
	// 增加一次结构发生变化的次数
	modCount++;
	// 此TreeMap节点数量减一
	size--;

	// If strictly internal, copy successor's element to p and then make p
	// point to successor.
	// 被删除节点的左子树和右子树都不为空，那么就用 p节点的中序后继节点代替 p 节点
	if (p.left != null && p.right != null) {
		// successor: 得到p后面的节点
		Entry<K,V> s = successor(p);
		p.key = s.key;
		p.value = s.value;
		p = s;
	} // p has 2 children

	// Start fixup at replacement node, if it exists.
	// replacement为替代节点，如果p的左节点不为空，则为p的左节点，反之为p的右节点
	Entry<K,V> replacement = (p.left != null ? p.left : p.right);
	// 如果replacement不为空
	if (replacement != null) {
		// Link replacement to parent
		replacement.parent = p.parent;
		if (p.parent == null) 
			// 如p没有父节点，则根root直接变为替代节点
			root = replacement;
		else if (p == p.parent.left) //如果P为左节点，则用replacement来替代为左节点
			p.parent.left  = replacement;
		else
			p.parent.right = replacement; //如果P为右节点，则用replacement来替代为右节点

		// Null out links so they are OK to use by fixAfterDeletion.
		p.left = p.right = p.parent = null; //去除p节点

		// Fix replacement
		// 根据节点的颜色，来删除。红色：直接删除，黑色：删除之后，需要平衡树，调整位置。
		if (p.color == BLACK)
			fixAfterDeletion(replacement);
	} else if (p.parent == null) { // return if we are the only node.
		// 说明是唯一的节点，当前root直接返回 null 即可
		root = null;
	} else { //  No children. Use self as phantom replacement and unlink.
		if (p.color == BLACK)
			fixAfterDeletion(p);
		// 删除p节点
		if (p.parent != null) {
			if (p == p.parent.left)
				p.parent.left = null;
			else if (p == p.parent.right)
				p.parent.right = null;
			p.parent = null;
		}
	}
}
```

# 最后
本文是对TreeMap做了一个简单的介绍，没有对删除节点之后，红黑树怎么自动平衡做讲解，后面会专门写一篇文章对红黑树做一个学习。
