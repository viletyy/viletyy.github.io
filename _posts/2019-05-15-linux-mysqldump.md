---
layout: post
title: "Linux: mysqldump用法"
date:       2019-05-14 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## Mysql导出表结构及表数据 mysqldump用法

命令行下具体用法如下：  mysqldump -u用戶名 -p密码 -d 數據库名 表名 脚本名;

1、导出數據库為dbname的表结构（其中用戶名為root,密码為dbpasswd,生成的脚本名為db.sql）

```shell
mysqldump -uroot -pdbpasswd -d dbname >db.sql;
```

2、导出數據库為dbname某张表(test)结构

```shell
mysqldump -uroot -pdbpasswd -d dbname test>db.sql;
```

3、导出數據库為dbname所有表结构及表數據（不加-d）

```shell
mysqldump -uroot -pdbpasswd  dbname >db.sql;
```

4、导出數據库為dbname某张表(test)结构及表數據（不加-d）

```shell
mysqldump -uroot -pdbpasswd dbname test>db.sql;
```



