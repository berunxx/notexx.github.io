---
title: 使用hexo 同步code到GitHub
tags:
  - hexo
category: Hexo
abbrlink: 5aa02c5
date: 2017-09-20 00:00:00
---

因为 我们第一次同步的代码到master分支，只属于编译之后的代码（md文件解析成了HTML文件）。当我们需要更换电脑的时候，同步master分支的代码就无法自由更新hexo blog。所以我们就需要把本地的code也同步上传到GitHub上面。步骤如下：

``` bash
$ git init  //初始化本地仓库
$ git add source //将必要的文件依次添加，有些文件夹如npm $install产生的node_modules由于路径过长不好处理，所以这里没有用`git add .`命令了，而是依次添加必要文件，如下图所示
$ git commit -m "code commit"
$ git branch hexo  //新建hexo分支
$ git checkout hexo  //切换到hexo分支上
$ git remote add origin git@github.com:yourname/yourname.github.io.git  //将本地与Github项目对接
$ git push origin hexo  //push到Github项目的hexo分支上 
```
 