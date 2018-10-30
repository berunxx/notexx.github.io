---
title: jdk8源码学习：ConcurrentHashMap
date: 2018-10-10 10:43:50
category: "java"
tags:
  - java
  - jdk
---
# 概述
ConcurrentHashMap 是一个线程安全类，是为了解决HashMap的线程不安全衍生出的一个类。虽然说HashTable也是线程的安全，但是HashTable的同步机制颗粒度太粗（实现机制是将put、size等各种方法加上‘synchronized’，导致了所有并发操作需要竞争同一把锁），导致性能低下,现在已经很少被推荐使用。
在jdk8中，ConcurrentHashMap底层实现使用了数组+链表+红黑树的数据结构
# 常用变量
## LOAD_FACTOR 加载因子
LOAD_FACTOR的默认大小是`0.75f`。为何是`0.75`	而不是其他的值呢？
首先理解一下，什么是加载因子。
- 加载因子是表示Hsah表中元素的填满的程度。如果加载因子越大,填满的元素越多，所以空间利用率提高，但是冲突的机会加大了。反之亦然
- 冲突的机会越大，查找所需要的成本增加，查找时间也相应的增加 了。反之亦然。
- 结合前面两条，我们必须在 "冲突的机会"与"空间利用率"之间寻找一种平衡与折衷。这种平衡与折衷本质上是数据结构中有名的"时间复杂度-空间复杂度"矛盾的平衡与折衷。

## DEFAULT_CAPACITY 默认容量
DEFAULT_CAPACITY的默认值为16，可以通过构造方法定义该值，但是需要注意的是，DEFAULT_CAPACITY的值是2的幂次方。结合前面的加载因子，可以得出，ConcurrentHashMap的初始化容量为:`0.75*16=12`,当size大于12的时候，就会发生扩容。

# 构造方法
```java
// 使用默认初始化表大小（16）创建一个空的map
public ConcurrentHashMap() {
}
```

```java
// 创建一个新的空map，初始化表大小容纳指定数量的元素，不需要动态调转大小。
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}
```
该构造方法 就是设置 sizeCtl; 首先计算`initialCapacity + (initialCapacity >>> 1` （可以简写为： `1.5*initialCapacity + 1`）, 再调用`tableSizeFor` 方法，该方法的作用是：对`1.5*initialCapacity + 1 `向上取最近的2的n次方。比如：初始化大小为：10，计算结果为16。 初始化大小为：15，计算结果为32.

# 常用方法
## put 方法

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
```
该方法主要是调用了 `putVal()`,接下来是实现具体实现方法

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
     // key 和 value 都不能为空
    if (key == null || value == null) throw new NullPointerException();
     // 计算 hash
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
		// 如果数组为空，则初始化数组
        if (tab == null || (n = tab.length) == 0)
			// 初始化数组方法 后面单独介绍
            tab = initTable();
		// 通过hash值找到数组对应的节点f
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
			// f节点为空，调用casTabAt方法
			//casTabAt 具体执行过程是，底层通过sun.misc.Unsafe 调用了compareAndSwapObject。该方法是无锁算法，将值插入数组中。
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
		// MOVED : 转换节点的hash值
        else if ((fh = f.hash) == MOVED)
			// 帮助数据迁移。并发情况下。
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
					// 头结点的 hash 值大于 0，说明是链表。
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
							// 根据hash判断是否有相等的key， 如果有匹配成功，通过onlyIfAbsent 判断是否需要覆盖该key对应的值。然后break。
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
							// 到了这儿，说明没有匹配的key，如果该节点的下一个节点为空，说明已经到了链表的末端，将值给放在该链表的最末端即可，然后break。
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
					// 判断是否为红黑树
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
						// 调用红黑树的put方法，根据onlyIfAbsent判断是否覆盖该值。
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
			// 如果 binCount不等于0，则在进行链表操作。
            if (binCount != 0) {
				// TREEIFY_THRESHOLD 默认为8
				// 如果binCount的值大于等于8，则调用treeifyBin方法
                if (binCount >= TREEIFY_THRESHOLD)
					// 后面单独介绍该方法
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```
put方法的主要流程就是这样，接下来看看，put流程中遗留的几个问题。
## 初始化数组 initTable
```java
    /**
     * Initializes table, using the size recorded in sizeCtl.
	 * 使用sizeCtl中记录的大小初始化表。
     */
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
			// 初始化‘线程’竞争失败。等待。
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
			// 利用CAS操作， 并设置sizeCtl的值。
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
						// 三目运算，DEFAULT_CAPACITY默认为16
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
						// 如果n的值为16，则sc计算结果为12
                        sc = n - (n >>> 2);
                    }
                } finally {
					// 设置sizeCtl值
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

## 链表转红黑树 treeifyBin

```java
    private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
			// 如果数组的长度小于 MIN_TREEIFY_CAPACITY（64），对数组进行扩容
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
				// n值翻倍之后传入
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) { // 加锁
                    if (tabAt(tab, index) == b) {
                        TreeNode<K,V> hd = null, tl = null;
						// 遍历链表，建立红黑树。
                        for (Node<K,V> e = b; e != null; e = e.next) {
                            TreeNode<K,V> p =
                                new TreeNode<K,V>(e.hash, e.key, e.val,
                                                  null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
						// 将红黑树设置到数组的相应位置
                        setTabAt(tab, index, new TreeBin<K,V>(hd));
                    }
                }
            }
        }
    }
```
链表转红黑树方法，在开始的时候回校验数组的长度，如果小于64，会进行扩容，反之，进行转红黑树过程。
## 扩容 tryPresize
将ConcurrentHashMap的容量扩展为原来的两倍。

```java
private final void tryPresize(int size) {
	// size 在传递过来的时候，已经翻倍了。 如果size大于等于 最大容量的二分之一，c的值就是默认最大容量
	// 反之，调用tableSizeFor方法，该方法就是对(size*1.5+1),在向上取2的n次方。
	int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
	tableSizeFor(size + (size >>> 1) + 1);
	int sc;
	while ((sc = sizeCtl) >= 0) {
		Node<K,V>[] tab = table; int n;
		if (tab == null || (n = tab.length) == 0) {
			n = (sc > c) ? sc : c;
			if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
				try {
					if (table == tab) {
						@SuppressWarnings("unchecked")
						Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
						table = nt;
						sc = n - (n >>> 2);
					}
				} finally {
					sizeCtl = sc;
				}
			}
		}
		else if (c <= sc || n >= MAXIMUM_CAPACITY)
			break;
		else if (tab == table) {
			int rs = resizeStamp(n);
			if (sc < 0) {
				Node<K,V>[] nt;
				if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
					sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
					transferIndex <= 0)
					break;
				if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
					transfer(tab, nt);
			}
			else if (U.compareAndSwapInt(this, SIZECTL, sc,
										 (rs << RESIZE_STAMP_SHIFT) + 2))
				transfer(tab, null);
		}
	}
}
```

## get(Object kye) 取值
接下来看看获取value的方法，get(key)规定了key不能为null，如果为null，则会抛出NPE。实现过程如下：

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 类似与HashMap中的hash方法
    int h = spread(key.hashCode());
    // table为当前对象中存储所有元素的数组，不能为空
    // tabAt是根据hash值，从table中找到hash为h的这个元素
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 判断元素的hash是否相等，接着判断key是否相等，如相等直接返回value
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        // 遍历链表，直到找到对应的值为止。
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```
get方法比较简单，没有加锁，只需要根据key直接取值就可以。类似`containsKey(Object key)`方法，其实就是调用了`get`方法，在判断get的返回值是否为空而已。

## remove(Object key) 删除
remove方法实现如下：

```java
public V remove(Object key) {
    return replaceNode(key, null, null);
}
```
具体实现在`replaceNode`方法中，接来下是该方法的具体实现。`replaceNode`是对`remove/replace`公共方法的实现

```java
final V replaceNode(Object key, V value, Object cv) {
    // 根据key的hashCode计算出hash值
    int hash = spread(key.hashCode());
    // 循环table
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // 如果tab为空或者通过tabAt得到的node节点为空，直接跳出循环
        if (tab == null || (n = tab.length) == 0 ||
            (f = tabAt(tab, i = (n - 1) & hash)) == null)
            break;
        //如果链表头节点的hash值为-1，说明table可能正在进行扩容，调用helpTransfer方法帮助扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            boolean validated = false;
            synchronized (f) { // 对链表头节点加锁
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        validated = true;
                        // 遍历链表查找key值
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) { // 判断节点e的key是否相等
                                V ev = e.val;
                                if (cv == null || cv == ev ||
                                    (ev != null && cv.equals(ev))) { // 如果给定的value为null，或者value与cv相等
                                    oldVal = ev;
                                    if (value != null) // 如果给定你的value不为null，替换节点e的value
                                        e.val = value;
                                    else if (pred != null) // 如果链表的前一个节点不为null，将节点e的下一个节点指向前一个节点的下一个节点。相当于删除当前节点
                                        pred.next = e.next;
                                    else
                                        setTabAt(tab, i, e.next); // 前一个节点为null，说明是链表头节点，插入头节点，next指向原头节点的next
                                }
                                break;
                            }
                            pred = e;
                            if ((e = e.next) == null) // 没有找到匹配的key，返回
                                break;
                        }
                    }
                    // 如果节点为树结构
                    else if (f instanceof TreeBin) {
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        // 调用findTreeNode方法根据key查找节点
                        if ((r = t.root) != null &&
                            (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            if (cv == null || cv == pv ||
                                (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                //调用树节点removeTreeNode方法删除节点，返回true说明节点太少，转化为链表结构
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    if (value == null)
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```

# 总结

* 在JDK8中使用了CAS方法保证操作的原子性
* 与JDK7不同的是，在JDK8中，当链表长度到达一定值时，会自动转为红黑树存储。
* 与JDK7不同的是，在JDK8中，ConcurrentHashMap不再使用segment结构，而是使用synchronized关键字对table中的链表头节点进行加锁，粒度更小，从而使同时操作的线程数量更多，效率更高。

## 参考
《Java并发编程实战》

[https://juejin.im/post/5a7844556fb9a06351725263](https://juejin.im/post/5a7844556fb9a06351725263)