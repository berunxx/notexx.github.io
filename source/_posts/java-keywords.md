---
title: Java关键字：instanceof、transient、 final
category:
  - Java
tags:
  - java
abbrlink: 384d8ae2
date: 2019-04-25 13:44:31
---
# instanceof
`instanceof` 是 Java 的一个二元操作符，类似于 ==，>，< 等操作符。
`instanceof` 是 Java 的保留关键字。用于测试对象是否是指定类型（类或子类或接口）的实例，返回Boolean类型。
eg:
``` java
public class Demo {
    public static void main(String[] args) {
        List arrayList = new ArrayList();
        Object obj = new Object();
        System.out.println(arrayList instanceof List); // output: true
        System.out.println(arrayList instanceof ArrayList); //output:  true
        System.out.println(arrayList instanceof LinkedList); //output:  false
    }
}
```

# transient
Java中的transient关键字用于指示字段不应该是序列化（这意味着保存，比如文件）过程的一部分。
[https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.3](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.3)
> 变量可以标记为瞬态，以指示它们不是对象的持久状态的一部分。
比如下面的列子，不需要将密码保存起来。
eg：
```java
import java.io.*;
public class TransientDemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User();
        user.setName("luffy");
        // password 字段为 transient
        user.setPassword("123456");
        writeFile(user);
        readFile();
    }

    private static void writeFile(User user) throws IOException {
        File file = new File("/Users/luffy/work/temp/user.txt");
        FileOutputStream os = new FileOutputStream(file);
        ObjectOutputStream oos = new ObjectOutputStream(os);
        oos.writeObject(user);
    }

    private static void readFile() throws IOException, ClassNotFoundException {
        File file = new File("/Users/luffy/work/temp/user.txt");
        FileInputStream os = new FileInputStream(file);
        ObjectInputStream ois = new ObjectInputStream(os);
        User user = (User) ois.readObject();
        System.out.println("name:" + user.getName()); // output name:wayne
        System.out.println("password:" + user.getPassword()); // output password:null
    }

}

import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionUID = -8855687778988171741L;

    private String name;
    private transient String password;
    // getter setter 省略
}
```

# final
final是一个非访问修饰符，仅适用于变量，方法或类。
> final Variable: 创建常量变量
> final Method: 禁止方法被覆盖
> final Class: 禁止类被继承

如果一个变量是用final关键字声明的，那么它的值不能被修改，本质上是一个常量。这也意味着必须初始化最终变量。
1. 可以在声明它时初始化最终变量。这种方法是最常见的。如果在声明时未初始化，则最终变量称为空白最终变量。
2. 空白的最终变量可以在实例初始化块或内部构造函数中初始化。如果您的类中有多个构造函数，则必须在所有构造函数中初始化它，否则将抛出编译时错误。
3. 可以在静态块内初始化空白的最终静态变量。

让我们通过一个例子看到上面初始化最终变量的不同方法。

```java

public class FinalDemo {
    // 一个final变量, 直接初始化
    final int A = 5;

    // 一个空白变量, 后面通过初始化实例块 初始化变量
    final int B;

    {
        B = 10;
    }

    // 另一个空白的 final 变量,通过构造方法初始化
    final int C;

    public FinalDemo() {
        C = 15;
    }

    // 一个静态类型的 final 变量, 直接初始化
    static final double D = 20;

    // 一个空白的 静态类型final 变量,通过静态初始化块
    static final double E;
    static {
        E = 25;
    }
}
```
