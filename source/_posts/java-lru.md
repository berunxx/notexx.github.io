---
title: Java实现LRU算法
category:
  - Java
tags:
  - 算法
abbrlink: 2d1b3bf6
date: 2019-07-18 12:22:19
---
# 介绍
LRU 的全称是 Least Recently Used ，它是一种最少使用算法，当存储的空间已满时，将最久未使用到的数据淘汰。
# 实现
使用HashMap和双向链表实现。因为HashMap的get()方法时间复杂度为O(1)，双向链表使节点添加/删除操作O(1)。
* get(key) 获取缓存中的值
* put(key, value) 添加值，当缓存容量满了时，则删除最久未访问的数据。

建立一个Node类
```java
/**
 * 双向链表
 */
class Node {
    String key;
    String value;
    Node next;
    Node prev;

    public Node(String key, String value) {
        this.key = key;
        this.value = value;
    }
}
```
Node类里面包含了key、value 和前置节点、后置节点。
LRUCache实现类如下：
```java

import java.util.HashMap;
import java.util.Map;

public class LRUCache {

    /**
     * 尾节点
     */
    Node tail;
    /**
     * 头结点
     */
    Node head;
    Map<String, Node> map = null;
    private int capacity = 0; // 容量
    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>();
    }

    public String get(String key) {
        Node node = map.get(key);
        if (node == null) {
            return "";
        }
        // 移除末尾节点
        removeNode(node);
        // 移动 node 到头节点
        moveToHead(node);
        // 返回
        return node.value;
    }

    public void put(String key, String value) {

        if (map.containsKey(key)) {
            // 已经存在该 key 将key 移动到头结点
            Node node = map.get(key);
            // 移除末尾节点
            removeNode(node);
            // 移动 node 到头节点
            moveToHead(node);
        } else {
            // 容量判断
            if (map.size() >= capacity) {
                // 删除 尾节点
                map.remove(tail.key);
                removeNode(tail);
            }
            // 插入节点 到 头节点
            Node node = new Node(key, value);
            moveToHead(node);
            map.put(key, node);

        }
    }

    /**
     * 移除节点
     * @param n
     */
    private void removeNode(Node n) {
        if (n.prev != null) {
            n.prev.next = n.next;
        } else {
            head = n.next;
        }
        if (n.next != null) {
            n.next.prev = n.prev;
        } else {
            tail = n.prev;
        }
    }

    /**
     * 添加节点到头节点
     * @param n
     */
    private void moveToHead(Node n) {
        final Node f = head;
        head = n;
        if (f != null) {
            f.prev = n;
            head.next = f;
        }
        if (tail == null) {
            tail = f;
        }
    }
}
```
# 结果
建立一个测试类
```java

public class LRUCacheTest {
    public static void main(String[] args) {
        // 设置容量为10 的缓存
        LRUCache cache = new LRUCache(10);
        for (int i = 0; i < 12; i++) {
            cache.put("" + i, "" + i);
        }
        Node head = cache.head;
        // 输出缓存中的元素
        // output: 11 10 9 8 7 6 5 4 3 2  
        while (head != null) {
            System.out.print(head.value + " ");
            head = head.next;
        }
        System.out.println();
        System.out.println(cache.get("0")); // "0"没有在缓存中 output: ""
        System.out.println(cache.get("3")); // output: "3"
        System.out.println(cache.get("4")); // output: "4"

        // 因为上面 获取了 '3'、'4'这两个元素，所以缓存中的元素顺序发生了变化
        Node head2 = cache.head;
        // output: 4 3 11 10 9 8 7 6 5 2
        while (head2 != null) {
            System.out.print(head2.value + " ");
            head2 = head2.next;
        }
    }
}
```
通过上面的测试 LRU 缓存是将最新使用的元素更新到链表的前面，当超出容量时，则移除链表末尾的元素。
![](https://ws1.sinaimg.cn/large/64202e18ly1g53xf21frlj20fw06wjri.jpg)
