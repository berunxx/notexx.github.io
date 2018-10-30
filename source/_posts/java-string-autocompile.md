---
title: java 字符串位数不足自动补全
date: 2018-04-27
category: "java" 
tags: 
  - java
---
## java 字符串位数不足自动补全
* 使用`org.apache.commons.lang`包下的`StringUtils`的leftPad函数，可以直接得到想要的结果。


```java
public class Demo{
	public static void main(String[] args) {
		//第一个参数：补位之前的值，第二个参数：总共的位数，第三个参数：补位用什么字符来填充。
		final String s = org.apache.commons.lang.StringUtils.leftPad("10110", 8, "0");
		System.out.println(s); // output: 00010110
	}
}
```