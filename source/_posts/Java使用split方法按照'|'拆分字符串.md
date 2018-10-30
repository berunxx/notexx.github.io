---
title: Java使用split 按照'|'（竖线）拆分字符串
date: 2017-11-20
category: "java" 
tags: 
  - java
---

在使用String类中的split方法时候，才开始使用了

```  java
"ab|cd".split("|"); 
```
 发现得到的结果是错误的，结果如下：
 
``` java
a
b
|
c
d
```
<!--more-->
很显然，这不是我所需要的结果。
原因是竖线 | 在正则中是特殊字符，需要转义，也就是

```
split("\|");
```
但实际在java中使用时，\又是java的特殊字符，需要转义，最终变成了

```
split("\\|");
```
正确使用如下：

``` java
"ab|cd".split("\\|");
```
结果如下：

```
ab
cd
```
