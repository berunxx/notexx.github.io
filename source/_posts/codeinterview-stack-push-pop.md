---
title: 剑指offer——栈的压入、弹出序列（java版本）
date: 2018-10-11 16:19:28
category: "算法"
tags:
  -剑指offer
---
>栈的压入、弹出序列
[牛客网——栈的压入、弹出序列](https://www.nowcoder.com/practice/d77d11405cc7470d82554cb392585106?tpId=13&tqId=11174&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)
## 题目描述
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
## 思路
使用一个栈来模拟压入、弹出操作。依次将第一个数组pushA压入栈，压入之后比对第二个数组popA，判断是否需要弹出。如果pushA已经全部入栈，但是stack却没有弹出完。说明不是一个弹出序列。
## 实现

```java
public boolean IsPopOrder(int [] pushA,int [] popA) {
    if (pushA == null || pushA.length == 0) {
        return true;
    }
    Stack<Integer> stack = new Stack<>();
    int len = pushA.length;
    for (int i = 0, popIndex = 0; i < len; i++) {
        stack.push(pushA[i]); // 依次入栈
        // 栈顶元素和popA[popIndex]比较，判断是否需要弹出。
        while (i < len && !stack.isEmpty() && stack.peek() == popA[popIndex]) {
            stack.pop();
            popIndex++;
        }
    }
    return stack.isEmpty();
}
```