---
layout: post
title: "开源世界旅行手册：软件管理"
date:  2019-12-30 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## 软件管理

Linux下安装软件，大致有两种方式：二进制和源代码。

二进制预编译软件包，可以快速安装部署你所需要的软件，但是你不能决定软件的特性。

使用源代码安装软件，虽然比较耗时，但可以满足你的特殊要求。

### 预编译软件包

预编译软件包可以满足大多数用户的需求，如果没有特殊要求，很少有人会选择编译软件

最早的软件包管理系统不支持网络，类似在Windows下安装软件，你必须获取一个安装包（比如通过网络下载）然后在本地安装它。使用这种方式安装软件的包管理系统主要有：dpkg（debian系）、rpm（redhat系）

由于Linux系统中，软件包拆分的比较细，安装一个软件可能要许多软件包。

而用户自己动手获取这些软件包，是一个十分繁琐的过程，因此产生了通过网络安装软件的方式：包管理系统自动分析你将安装的软件需要哪些软件包，通过网络下载、安装、配置它们。

debian系发行版通过apt系统实现网络支持，配合本地的dpkg实现这一过程；redhat系发行版则通过yum和rpm实现。而一些新派发行版，包管理系统兼具这两种能力，例如archlinux的pacman；而gentoo的emerge虽然基于源代码编译安装，但也具备这种能力

常见软件包管理系统的基本操作：

| 任务           | apt                        | zypp                             | yum                                 | pacman                | emerge                       |
| -------------- | -------------------------- | -------------------------------- | ----------------------------------- | --------------------- | ---------------------------- |
|                | Debian，Ubuntu             | openSUSE                         | Fedora, CenOS(redhat系)             | Archlinux             | Gentoo                       |
| 安装包         | apt-get install pkg        | zypper install pkg               | yum install pkg                     | pacman -S pkg         | emerge pkg                   |
| 移除包         | apt-get remove pkg         | zypper remove pkg                | yum erase pkg                       | pacman -R pkg         | emerge -C pkg                |
| 更新包列表     | apt-get update             | zypper refresh                   | yum check-update                    | pacman -Sy            | emerge --sync \| layman -S   |
| 更新系统       | apt-get upgrade            | zypper update                    | yum update                          | pacman -Su            | emerge -avuDN world          |
| 列出源         | cat /etc/apt/sources.list  | zypper repos                     | yum repolist                        | cat /etc/pacman.conf  | layman -L                    |
| 添加源         | edit /etc/apt/sources.list | zypper addrepo 仓库地址 仓库名称 | add 仓库 to /etc/yum.repos.d/       | edit /etc/pacman.conf | layman -a                    |
| 移除源         | edit /etc/apt/sources.list | zypper removerepo 仓库名称       | remove 仓库 from  /etc/yum.repos.d/ | edit /etc/pacman.conf | layman -d                    |
| 搜索包         | apt-cache search pkg       | zypper search pkg                | yum search pkg                      | pacman -Qs pkg        | emerge --search              |
| 列出已安装的包 | dpkg -l                    | rpm -qa                          | rpm -qa                             | pacman -Qii           | cat /var/lib/portage \| more |

把SUSE系和redhat系排在前面，不是因为它们的软件包管理系统比较先进，而是考虑到它们的用户多。


参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)