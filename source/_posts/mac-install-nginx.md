---
title: Mac对nginx 启动、关闭、重启
category: MacOS
tags:
  - mac
  - nginx
abbrlink: c21378c4
date: 2018-10-22 00:00:00
---

## 问题
在终端中执行：
`brew install nginx`
安装完成之后，nginx配置文件路径在`/usr/local/etc/nginx`下
## 常用命令
查看nginx版本
`nginx -v`
![](/images/nginx-v.jpg)
启动nginx：`brew services start nginx`
![](/images/nginx-start.jpg)
关闭nginx：`brew services stop nginx`
![](/images/nginx-stop.jpg)

也可以使用如下快捷指令
重新启动nginx：
`nginx -s reload`
停止nginx：
`nginx -s stop`
