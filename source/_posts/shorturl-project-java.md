---
title: Java 短链接项目
category: Java
tags:
  - java
  - spring
  - springboot
abbrlink: 8913fd1b
date: 2018-07-31 00:00:00
---
# 需求
将很长的http请求的地址，转为短链接。如下所示：
![](http://www.myluffy.com/wp-content/uploads/2018/07/112a80391435658790d15308b377aba3.png)
为什么要转短链，短链接可以转发给第三方的时候，能轻易识别，我在开发中有遇到过一次，别人分享给我的地址里面包含了一些特殊字符（$、&）等，我在我的微信里面，直接点击地址，只识别到特殊字符的前半部分，没有整个链接识别完成。
## 短链服务的作用：
- 将长链接变为短链接，当然是越短越好
- 用户点击短链接的时候，实现自动跳转到原来的长链接

## 原理解析
- 在游览器地址栏输入短链接地址：如 http://domain/shUESS
- 游览器会将domain解析为ip地址，得到IP地址之后，会向这个IP发送请求。
- 后台接收到shUESS字符之后，去对应的存储中查询出原网址。
- 通过重定向调转到原网址。

# 实现
demo使用springboot完成，相关依赖如下：
```
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

		<!-- MD5加密 -->
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.5</version>
        </dependency>
```

## 长链接转短链接
将一长串的链接，转为4至6个字符，我是使用MySQL做存储，用来存储，转换之后的字符 short_key和原来的链接originalUrl；
表结构简单版本如下：
- id: 主键ID，自动递增
- short_key: 转换之后的字符，正式环境数据量很大，可以建立索引
- originalUrl: 原来的网址

本文采用MD5加密算法
对传入网址进行MD5加密，并且将加密之后的字符串，分为4部分分别于0x3FFFFFFF 进行位与运算，生成4个字符串，4个字符串都可以使用。
加密核心代码如下：
```java
public static String[] shortUrl(String url) {
        // 可以自定义生成 MD5 加密字符传前的混合 KEY
        String key = "";
        // 要使用生成 URL 的字符
        String[] chars = new String[]{"a", "b", "c", "d", "e", "f", "g", "h",
                "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t",
                "u", "v", "w", "x", "y", "z", "0", "1", "2", "3", "4", "5",
                "6", "7", "8", "9", "A", "B", "C", "D", "E", "F", "G", "H",
                "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T",
                "U", "V", "W", "X", "Y", "Z"};

        // 对传入网址进行 MD5 加密
        String sMD5EncryptResult = DigestUtils.md5Hex(key + url);
        String hex = sMD5EncryptResult;
        String[] resUrl = new String[4];
        for (int i = 0; i < 4; i++) {
            // 把加密字符按照 8 位一组 16 进制与 0x3FFFFFFF 进行位与运算
            String sTempSubString = hex.substring(i * 8, i * 8 + 8);
            // 这里需要使用 long 型来转换，因为 Inteper .parseInt() 只能处理 31 位 , 首位为符号位 , 如果不用
            // long ，则会越界
            long lHexLong = 0x3FFFFFFF & Long.parseLong(sTempSubString, 16);
            String outChars = "";
            for (int j = 0; j < 6; j++) {
                // 把得到的值与 0x0000003D 进行位与运算，取得字符数组 chars 索引
                long index = 0x0000003D & lHexLong;
                // 把取得的字符相加
                outChars += chars[(int) index];
                // 每次循环按位右移 5 位
                lHexLong = lHexLong >> 5;
            }

            // 把字符串存入对应索引的输出数组
            resUrl[i] = outChars;
        }
        return resUrl;
    }
```

## 短链接转长链接
有了长链接转短链接之后，这一步就简单多了。
通过请求的地址，得到short_key,去查询出对应的原网址，再重定向到原网址即可。
代码如下：
```java
@RequestMapping("/{key}")
public String find(@PathVariable String key) {
	if (StringUtils.isEmpty(key)) {
		return "error";
	}
	ShortUrl byShortKey = repository.getByShortKey(key);
	if (byShortKey == null) {
		return "error";
	}

	return "redirect:" + byShortKey.getOriginalUrl();
}
```

完整项目已上传到[github](https://github.com/waynecoder/springboot-example/tree/master/short-url "short_url")

# 参考
[https://blog.csdn.net/yushouling/article/details/55096992](https://blog.csdn.net/yushouling/article/details/55096992 "https://blog.csdn.net/yushouling/article/details/55096992")
