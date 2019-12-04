---
title: JDK8源码学习： Integer 的自动装箱拆箱和缓存机制
category:
  - java相关
tags:
  - java
  - 源码
abbrlink: '90698191'
date: 2019-04-21 21:10:07
---
# 介绍
`Integer` 是 `int` 的包装类，`Integer` 中还有一个匿名内部类 `IntegerCache` 它是一个对 `Integer` 的对象缓存，但是它有一个范围，默认是 `-128——127`。首先来看看 `IntegerCache` 的实现。
```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            // 默认为127
            int h = 127;
            //获取JVM设置的参数值（+XX:AutoBoxCacheMax=？）
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    // 防止设置的参数大于了 Integer的最大值。
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                // 将-128 到 h的值放进数组里面
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
IntegerCache的功能就是将-128到127的整数缓存起来。并且可以通过jvm参数设置 IntegerCache的最大值，增加了灵活性。
# 属性
下面是一些Integer的基本参数介绍。
## MAX_VALUE 最大值
`MAX_VALUE`顾名思义，就是Integer的最大值，它是不可变的。它的值为 2^31-1 = 2147483647
``` java
@Native public static final int   MAX_VALUE = 0x7fffffff;
```
## MIN_VALUE 最小值
`MIN_VALUE`对应Integer的最小值，也是不可变的。值为： -2^31 = -2147483648
```java
@Native public static final int   MIN_VALUE = 0x80000000;
```
# 方法
介绍一下常用方法。
## toString 方法
`Integer#toString` 其实就是将Integer类型变成了String类型。
```java
public String toString() {
    return toString(value);
}

public static String toString(int i) {
    if (i == Integer.MIN_VALUE)
        // 如果为最小值 直接返回最小值
        return "-2147483648";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
    char[] buf = new char[size];
    getChars(i, size, buf);
    return new String(buf, true);
}
```

## valueOf(int) 方法
我们先看看下面这段代码：
```java
public class Demo {
    public static void main(String[] args) {
        Integer i = 10;
        System.out.println(i);
    }
}
```
10是 `int` 类型，为什么可以直接赋值给 `Integer` 类型呢？我们分别张歘`javac Demo.class` 和 `javap -c Demo.class`命令看看程序是怎么编译的。
```java
  public static void main(java.lang.String[]);
    Code:
       0: bipush        10
       2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1
       6: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       9: aload_1
      10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      13: return
```
<!-- ![bc7fca64c9f9284e995e4f618989bb02.png](evernotecid://99E24FA2-4BAA-475C-B10C-A7D31B5CD72F/appyinxiangcom/16310471/ENResource/p851) -->
上面是部分编译结果，可以看见`2: invokestatic  #2 ` 自动调用了`Integer#valueOf` 方法，这里就是程序自动将 `int` 变成了 `Integer` 类型。
现在我们来看`Integer#valueOf`的具体实现。
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
我们在文章开头已经说了一下 `IntegerCache` 类，我们可以看见 `valueOf` 的方法，首先判断i的值是否在 `IntegerCache` 的范围之内，如果在，直接从cache里面获取即可，不在范围之内，则new 一个Integer 返回。

## intValue() 方法
下面这段代码将 一个Integer类型的赋值给了int类型。为什么Integer可以赋值给int，这里就涉及到了Integer的自动拆箱。
``` java
Integer i = 10;
int i2 = i;
```
在执行 `int i2 = i;`的时候，i会先执行`Integer#intValue`再赋值给i2。下面是 `Integer#intValue`的实现代码。
```java
private final int value;

public int intValue() {
    return value;
}
```
就一行代码，直接返回了Integer的value。

# 结束
Integer的缓存机制，是面试中或者笔试中比较常见的，需要注意下。