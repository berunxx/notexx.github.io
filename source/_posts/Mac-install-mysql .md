---
title: Mac安装MySQL
category: Mac
tags:
  - mac
  - mysql
abbrlink: ec3b3038
date: 2017-09-17 00:00:00
---
# Mac 安装mysql
1. 下载dmg文件
* 双击安装
* 点击安装完成之后会出现一个提示，上面默认是root用户 和密码。千万要记住。不要一下手快给关掉了。

* 如果不小心关掉了。就需要用终端。
  ```bash
  $ cd /usr/local/mysql/bin
  ```
* 切换到root权限 ，需要输入密码。

  ```bash
  $ sudo su
  ```
* 输入之后会看见如下信息：

  ```bash
  sh-3.2#
  ```
* 使用如下命令以安全模式运行mysql

  ```bash
  mysql> ./mysqld_safe --skip-grant-tables &
  ```
* 现在从开一个终端:输入:

	```bash
    $ mysql -uroot
   ```
就可以进入mysql
* 修改root密码，执行下面的命令。

	```bash
    mysql> FLUSH PRIVILEGES;
   ```
* 继续执行一下名利修改密码：

	```bash
	mysql> SET PASSWORD FOR root@'localhost' = PASSWORD('new password');
	```
* 到此为止。
