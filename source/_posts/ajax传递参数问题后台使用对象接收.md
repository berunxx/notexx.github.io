---
title: 使用ajax传递参数，后台使用对象接收。报400错误。
date: 2017-11-22
category: "java"
tags:
  - java
  - SpringBoot
  - ajax
---

* 在ajax的传递Json格式的数据到后台，后台使用SpringBoot。

	**前台Json格式如下:**

    ``` java
    [
    {
        "age": 1,
        "name": "test",
        "gender": null,
        "order": 1
    }
    ]
    ```
   **后台接收对象格式如下:**

    ``` java
    public class User{
    	private Integer age;
        private String name;
        private Gender gender; // Gender 为枚举类型
        private Integer order;

	}
    ```
    因为gender为枚举类型，所以有个Gender Enum 类:
    ``` java
    public Enum Gender{
    	MALE,
        FEMALE;
    }
    ```
    **注意**：

    前台Json格式中 ‘gender’ 为 null，后台使用对象接收，因为 接收对象中属性Gender为枚举类型。所以前台会报‘400’ 错误，后台因为看见枚举值为 null，或者枚举值不存在，就会报错。
* 所以在使用有枚举类型对象接收Json格式参数的时候，应该保证传输的值和枚举中对应的属性值一样。
