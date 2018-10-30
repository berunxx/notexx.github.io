---
title: Mac使用fish终端命令，不支持 `$`符号
date: 2018-10-27 07:37:13
category: "Mac" 
tags: [mac]
---
## 问题
在fish命令中使用以下命令
eg 1:
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
eg 2:
```shell
cd "$(brew --repo)"
```
报错提示：
> fish: $(...) is not supported. In fish, please use '(curl)'.
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

在Mac 的fish终端中使用了 "$"报错。
## 解决方法
去掉`$`即可工作

```shell
/usr/bin/ruby -e (curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)
```


