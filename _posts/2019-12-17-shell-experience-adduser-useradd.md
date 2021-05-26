---
layout: post
title: "Shell命令：Linux中adduser和useradd的区别"
date:   2019-12-17 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Shell
  - Linux
---

### 本质

useradd是一个linux命令，但是它提供了很多参数在用户使用的时候根据自己的需要进行设置；

adduser是一个perl 脚本，在使用的时候会出现类似人机交互的界面，提供选项让用户填写和选择，这个命令比起useradd来说比较简单，也比较傻瓜。

### 输入后，产生不同的效果

**useradd**只是创建了一个用户名。

**adduser**会在/home目录下创建同名文件夹，然后利用（passwd+用户名）为指定的用户名设置密码。

**adduser**会用类似底层命令的更友好的前端，进行以下操作：

1. 建立一个新目录作为家目录
2. 建立同名新租
3. 把用户的主要组设为该组
4. 从/etc/SKEL目录下拷贝文件到家目录，完成初始化
5. 建立新用户的密码
6. 如果其存在的话，还会执行一个脚本