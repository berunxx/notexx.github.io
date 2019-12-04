---
title: Java8源码学习-String
category:
  - Java
tags:
  - java
  - 源码
abbrlink: e440274b
date: 2019-04-18 21:22:08
---

# 介绍
String是开发中比较常用的类了。String表示字符串，比如"abc"等。String实现了序列化（Serializable）、排序（Comparable）、字符串(CharSequence)等方法。 String是不可变的，String在定义的时候使用了final，也就是说一旦创建，就不可更改。
``` java
public final class String
extends Object
implements Serializable, Comparable<String>, CharSequence
```
# 属性
## value
它使用字符存储，私有变量，并且为不可变的char类型，定义如下：
``` java 
private final char value[];
```
## hash
缓存String的hash code值，默认为0
``` java
private int hash; // Default to 0
```

# 构造方法
下面介绍几个常用的构造方法
## 空初始化 
创建一个`""`字符串。
``` java
public String() {
    this.value = "".value;
}
```
## String类型初始化
在使用使用`new`关键字的时候，我们一般会使用 `new String("str")`来新建一个String。调用构造方法如下：
``` java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```
## 字符型（char）类型初始化
通过`Arrays.copyOf`方法（底层实现调用了 `System.arraycopy()`）得到一个char[]赋值给value
```
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
```
在使用String的使用中，`public String(char value[], int offset, int count)`方法在内部调用会常用到，比如：`subString`方法等。下面注意分析一下该方法的实现。
``` java
// offset：可以理解为偏移量，从什么位置开始。
// count：字符串的长度
public String(char value[], int offset, int count) {
    if (offset < 0) {
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        // 上面就是常规判断参数合法性
        // 到这里，说明count=0； 返回一个空字符串
        if (offset <= value.length) {
            this.value = "".value;
            return;
        }
    }
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
    // 调用 Arrays.copyOfRange方法 下面会说明
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}

```
``` java
// copyOfRange方法底层是调用了 `native`方法， `native`是一个原生函数，是由c++实现的。具体实现就不说了。
public static char[] copyOfRange(char[] original, int from, int to) {
    // 新字符串的长度
    int newLength = to - from;
    if (newLength < 0)
        throw new IllegalArgumentException(from + " > " + to);
    // 新建一个char数组
    char[] copy = new char[newLength];
    // System.arraycopy 是一个native方法
    // original： 原来的字符串， from： 复制的开始位置 
    // copy：新的存储字符串数组， 0：表示从索引0位置开始， Math.min(original.length - from, newLength)： 复制的元素数量
    System.arraycopy(original, from, copy, 0,
                     Math.min(original.length - from, newLength));
    return copy;
}
```
另外的 int、byte类型初始化不常用，就不说了。
## StringBuffer、StringBuilder类型初始化
一般不使用，需要注意一下，StringBuffer是线程安全的，如果在不考虑线程安全的情况下，拼接字符串不使用StringBuffer，原因是线程同步所带来的开销太大。不考虑线程安全，推荐使用StringBuilder。
``` java
public String(StringBuffer buffer) {
    synchronized(buffer) {
        this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
public String(StringBuilder builder) {
    this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```

# 常用方法

## equals 方法
String的equals方法是重写了Object的equals方法。
相关注释，写在代码之中，代码如下：
``` java
public boolean equals(Object anObject) {
    // 判断地址相等
    if (this == anObject) {
        return true;
    }
    // instanceof 关键字，用来判断传入参数类型
    // 判断如果不为String类型，直接返回false。
    if (anObject instanceof String) {
        // 向下转型
        String anotherString = (String)anObject;
        int n = value.length;
        // 判断anObject的长度和自身的长度是否相等
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 接着一个一个字符比较
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```
总结一下，上面的大概流程。
* 判断比较参数的地址是否相等，如果相等，直接返回true。
* 判断anObject的长度和自身的长度是否相等，如果相等，循环遍历两个char[], 一个个比较char字符是否相等。
使用例子：`"abc".equals("abc")`

## hashcode 方法
通过hashcode方法会返回一个int类型的数值。
实现如下：
``` java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            // val[i] 是获取 ASCII 码 比如 a ->97 , b ->98
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
hashcode方法比较简单，首先获取value的长度，然后根据长度遍历，反复计算h的值。比如：`"abc".hashCode()`的返回值为：96354，
h默认为0。字符对应的ASCII码，可以自行Google。
第一次遍历： h = 31 * 0 + 97 ，所以 h = 97 
第二次遍历： h = 31 * 97 + 98, h = 3105
第三次遍历： h = 31 * 3105 + 99, h = 96,354
还是比较容易理解的。

## indexOf 方法
`indexOf`可以接收`int`、`String`、`char`类型的参数, 该方法的作用是判断字符串是否包含传入参数。下面只介绍String的使用。
`indexOf(Strinf str)` 方法如下：
``` java
public int indexOf(String str) {
    return indexOf(str, 0);
}
public int indexOf(String str, int fromIndex) {
    return indexOf(value, 0, value.length,
            str.value, 0, str.value.length, fromIndex);
}
// 主要实现逻辑在这里。
// source: 字符串本身 、 sourceOffset：字符串从什么位置比较,、 target: 待比较的字符串、 targetOffset: 待比较的字符串比较起始位置、 fromIndex： 从什么位置开始，默认为0
static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
    if (fromIndex >= sourceCount) {
        // 开始比较的位置大于等于了本身字符串的长度
        // 待比较值为空，直接返回本身字符串长度，反之返回-1
        return (targetCount == 0 ? sourceCount : -1);
    }
    if (fromIndex < 0) {
        // 防止设置参数错误
        fromIndex = 0;
    }
    if (targetCount == 0) {
        // 相当于传递了一个空字符串， 直接返回 fromIndex
        return fromIndex;
    }
    // 获取第一个待比较值的value , 比如 ”abc“, first = 'a'
    char first = target[targetOffset];
    // 计算循环结束的位置 比如： "abc".indexOf("b") max为 0+(3-1) = 2;
    int max = sourceOffset + (sourceCount - targetCount);
    
    for (int i = sourceOffset + fromIndex; i <= max; i++) {
        /* Look for first character. */
        // 找出第一个值与本身值相等的位置 i
        if (source[i] != first) {
            while (++i <= max && source[i] != first);
        }

        /* Found first character, now look at the rest of v2 */
        // 前面主要是找出了第一个相等字符的位置，接着需要匹配后面的字符
        if (i <= max) {
            // j: 待比较第二个后面字符的开始位置
            int j = i + 1;
            // end：待比较字符的最后一个位置+1
            int end = j + targetCount - 1;
            // 依次比较后面字符 是否相等
            for (int k = targetOffset + 1; j < end && source[j]
                    == target[k]; j++, k++);
            // 如果j 不等于 end ，说明 target 的其中一个字符不在source中。
            if (j == end) {
                /* Found whole string. */
                // 返回第一个字符相等的位置
                return i - sourceOffset;
            }
        }
    }
    return -1;
}
```
indexOf方法，相对而言长一点，但是整体来说，还是不难理解。相应的注释已经写得差不多了，这里大概总结一下。
* 首先比较第一个字符，如果第一个字符都不等，直接返回-1，表示没有找到。
* 接着比较后面的字符，如果依次全等。返回第一个字符相等的位置。

## substring 方法
`substring`的功能是截取字符串，使用方式是指定一个起始位置`beginIndex`和结束位置`endIndex`(endIndex 可以不指定，默认为字符串最后一位)，代码实现如下：
``` java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        // 传入参数不合格，抛出字符串索引越界异常
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    // 需要截取的长度
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        // 小于0，说明传入beginIndex大于了字符串本身的长度，抛出异常
        throw new StringIndexOutOfBoundsException(subLen);
    }
    // 因为String是不可变的，所以返回一个新的字符串, new String()的具体实现，在构造方法里面已经说过了。
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

## replace 方法
`replace()`方法的功能是替换字符串，比如 `"hello world !".replace(" ", "-")` 变成了 `hello-world-!`
```java
// target：被替换的字符，replacement：替换的字符
public String replace(CharSequence target, CharSequence replacement) {
    return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
            this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
}
```
`replaceFirst()`方法功能是替换第一个匹配的字符串，比如 `"hello world !".replaceFrist(" ", "-")` 变成了 `hello-world !`
``` java
public String replaceFirst(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
}
```

`replaceAll()`方法功能是替换第一个匹配的字符串，比如 `"hello world !".replaceAll(" ", "-")` 变成了 `hello-world-!`
```java
public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}
```
这三个方法主要区别是 `replaceFirst()`调用了`Matcher#replaceFirst`， `replace()`和`replaceAll()`调用了`Matcher#replaceAll`方法。
Matcher的`replaceAll()`和`replaceFirst()`方法区别在对于`appendReplacement(sb, replacement);`的调用次数，`replaceFirst()`只调用一次，表示只替换第一个匹配的值
```java
public String replaceAll(String replacement) {
    reset();
    boolean result = find();
    if (result) {
        StringBuffer sb = new StringBuffer();
        do {
            // 实现太复杂，不讨论了。
            appendReplacement(sb, replacement);
            result = find();
        } while (result);
        appendTail(sb);
        return sb.toString();
    }
    return text.toString();
}
```

## join 方法
`join()`是1.8之后才有的新方法。功能是：将多个字符或者集合元素连接起来。例如：
```java
String message = String.join("-", "Java", "is", "cool");
// message returned is: "Java-is-cool"

List<String> strings = List.of("Java", "is", "cool");
String message = String.join(" ", strings);
//message returned is: "Java is cool"

Set<String> strings =
 new LinkedHashSet<>(List.of("Java", "is", "very", "cool"));
String message = String.join("-", strings);
//message returned is: "Java-is-very-cool"
```
上面的例子表明了`String#join`方法可以接收 可变数组格式的字符、List、Set格式参数。下面来看看join的实现。
```java
public static String join(CharSequence delimiter, CharSequence... elements) {
    // 非空验证
    Objects.requireNonNull(delimiter);
    Objects.requireNonNull(elements);
    // Number of elements not likely worth Arrays.stream overhead.
    StringJoiner joiner = new StringJoiner(delimiter);
    for (CharSequence cs: elements) {
        joiner.add(cs);
    }
    return joiner.toString();
}
```
上面的方法我们会看见有一个新的class出现了，`StringJoiner`也是1.8之后的才有的。主要参数有：`delimiter`：分隔符，`value`：定义类型是`StringBuilder`，由此可以看出来具体拼接还是通过`StringBuilder`实现的，另外还有`prefix`、`suffix`。
``` java
public StringJoiner add(CharSequence newElement) {
    prepareBuilder().append(newElement);
    return this;
}
// 主要实现在prepareBuilder()这里
private StringBuilder prepareBuilder() {
    if (value != null) {
        // 添加 连接字符
        value.append(delimiter);
    } else {
        // 新建一个StringBulider, 如果有前缀也加上。
        value = new StringBuilder().append(prefix);
    }
    return value;
}
```
`prepareBuilder()`执行完之后，才会去添加新的元素。

# 结束
到了这里也差不多了， `String`中还有很多方法，就不去一一解析了，当然我们需要知道怎么使用，具体功能是什么，下面做个功能总结：

* `trim` 去掉首尾的空格
* `toLowerCase` 将字符串大写字母转为小写
* `toUpperCase` 将字符串小写字母转为大写
* `split` 根据特定字符拆分成一个数组
* `toString` 这个方法使用频率很高，需要注意它和`String#valueOf`的区别。如果`null`调用了`toString`会抛出`NullException`，而`null`调用了`valueOf`,会返回`"null"`字符。

