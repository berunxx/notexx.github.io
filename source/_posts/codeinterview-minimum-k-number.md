---
title: 剑指offer——最小的K个数（java版本）
category: 算法
tags: '-剑指offer'
abbrlink: cb7b911e
date: 2018-10-11 16:16:58
---
>最小的K个数
[牛客网——最小的K个数](https://www.nowcoder.com/practice/6a296eb82cf844ca8539b57c23e6e9bf?tpId=13&tqId=11182&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)
## 题目描述
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。
## 解题思路
### 全排序
>先排序，接着找出最小的K个数。时间复制度为O(n*n)。

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
    if (input.length < k)
        return new ArrayList<>();
    int len = input.length;
    for (int i = 0; i < len; i++) {
        for (int j = 0; j < len-1; j ++){
            if (input[j] > input[j+1]) {
                int temp = input[j];
                input[j] = input[j+1];
                input[j+1] = temp;
            }
        }
    }
    ArrayList<Integer> result = new ArrayList<>();
    for (int i = 0; i < k; i++) {
        result.add(input[i]);
    }
    return result;
}
```
对于这种解法，是可以优化的，我只需K个数，说明我可以只对K个长度排序，不全排序。对于这种解法，可以叫做局部排序，时间复杂度O(n*k)。

### 堆
> 先用当前K个元素生存一个大顶堆，这个堆用于存储当前最小的k个元素。接着，从第k+1个元素开始扫描，和堆中最大的元素比较，如果被扫描的元素小于堆顶，则替换堆顶的元素，并调整堆，以保证堆内的k个元素，总是当前最小的k个元素。
> 时间复制度：O(n*lg(k))

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int[] nums, int k) {
    if (k > nums.length || k <= 0)
        return new ArrayList<>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>((o1, o2) -> o2 - o1);
    for (int num : nums) {
        maxHeap.add(num);
        if (maxHeap.size() > k)
            maxHeap.poll();
    }
    return new ArrayList<>(maxHeap);
}
```


