---
layout: post
title: "Git: 提交代码速度过慢解决方法(MacOS)"
date:   2018-04-11 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Git
  - Shell
  - MacOS
---

iterm2中ping下github.com和github.global.ssl.fastly.net

**ping github.com**

```shell
$ ping github.com
```

得到 

```shell
PING github.com (192.30.253.112): 56 data bytes
64 bytes from 192.30.253.112: icmp_seq=0 ttl=47 time=245.930 ms
64 bytes from 192.30.253.112: icmp_seq=1 ttl=47 time=245.910 ms
```

记录ip  192.30.253.112

**ping github.global.ssl.fastly.net**

```shell
$ ping github.global.ssl.fastly.net
```

记录ip 151.101.113.194

#### 解决方法

编辑hosts文件

```shell
$ vim /etc/hosts
```

加入这两行代码

```shell
192.30.253.112 github.com
151.101.113.194 github.global.ssl.fastly.net
```

最后更新DNS缓存

```shell
$ sudo dscacheutil -flushcache
```