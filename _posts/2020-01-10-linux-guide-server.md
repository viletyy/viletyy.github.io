---
layout: post
title: "开源世界旅行手册：服务器"
date: 2020-01-10 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---



## 服务器

### LAMP

LAMP是一个缩写，它包括：Linux操作系统，Apache网络服务器，Mysql数据库，Perl、PHP或者Python语言

毫不夸张的说，LAMP说开源世界的“皇家海军”（或许Emacs工具链可以算是“太平洋舰队”），正是凭借LAMP的力量，开源应用得以在服务器市场称雄。

web服务器（Apache）是LAMP的核心，几乎所有远程访问都要通过web服务器进行。除了Apache，web服务器还有一些其他的开源实现，比如：Lighttpd，Tomcat，Zope等

Perl、PHP、Python是服务器端脚本语言，web服务器以扩展的形式支持这些语言作为后端处理器，以获得强大的可编程能力

数据库服务器（MySQL）提供数据存储、检索支持。除了MySQL，还有PostgreSQL等开源实现

理论上可以直接通过网络访问数据库服务器，不过出于安全需要，多数LAMP应用中，数据库服务器只能由web服务器本地访问

也就是说，访问web服务器，产生了数据处理请求，web服务器把数据请求发送到数据库服务器，然后将数据库服务器返回的处理结果发送给访问者

数据库服务器接收和返回数据处理请求，要通过SQL查询语言。web服务器本身不支持SQL，而服务器端脚本语言可以内嵌SQL语句，web服务器通过脚本语言处理访问者对数据库服务器端请求

#### 虚拟主机

一个机器上可能有多个网络接口：每一块网卡会分配一个IP地址；类似127.0.0.1这样的回环地址指向本地机器

网络接口就像港口，有很多泊位，称为端口。可以使用不同的协议连接（如http、ftp、ssh等）

服务器工作时，会舰艇某一个地址的固定端口，称为“绑定”。如果该端口中传来请求，则进行响应

很多服务器可以针对不同的网络接口设置不同的响应规则，例如：从内网访问接口192.168.1.2，使用/home/user作为根目录；从公网访问接口211.148.131.7使用/var/WWW作为根目录。

这就是虚拟主机。

### Lighttpd

**`Lighttpd`**是一个新兴的、轻量级的 web 服务器，它开始越来越多的应用在一些重要场合，如：YouTobe、Sourceforge、豆瓣……

Lighttpd 以安全、快速和内存消耗低著称，还专门为大型分布式连接环境做了优化，支持 FastCGI, CGI, Auth, 输出压缩(output compress), URL重写, Alias 等重要功能。

Lighttpd 已经进入大多数发行版的软件仓库

安装完成后，用启动脚本启动：**`/etc/init.d/lighttpd start`**

`/etc/lighttpd/lighttpd.conf`为 Lighttpd 服务器的配置文件（查看`/etc/init.d/lighttpd`文件，可以看到类似字句：
**`/usr/sbin/lighttpd -D -f /etc/lighttpd/lighttpd.conf`** **- f** 选项指定配置文件）：

```shell
## 网站根目录 映射在机器上的物理路径
server.document-root        = "/home/lighttpd/html/"

## 如果网站目录中出现以下文件名，不用指定文件名便可直接访问
index-file.names            = ( "index.php", "index.html",
                                "index.htm", "default.htm" )

## Lighttpd 进程的归属用户
server.username             = "nobody"

## Lighttpd 进程的归属群组
server.groupname            = "nobody"

## 绑定到端口 默认为 80
#server.port                = 81

## 绑定到地址 默认为 所有
#server.bind                = "127.0.0.1"

## 访问日志 路径
accesslog.filename          = "/var/log/lighttpd/access.log"

## 错误日志 路径
server.errorlog             = "/var/log/lighttpd/error.log"

## 禁止访问以下文件
url.access-deny             = ( "~", ".inc" )  

## 与目录列表相关的设置
#dir-listing.activate       = "enable"  
#dir-listing.encoding       = "utf8"
#dir-listing.show-readme       = "enable"  
```

配置文件中的**`server.modules`**字段决定Lighttpd使用哪些扩展模块：

```shell
server.modules = ("mod_access","mod_fastcgi","mod_accesslog" )  
```

- Lighttpd 通过 **`mod_fastcgi`** 模块支持 PHP
- **`mod_accesslog`** 模块为访问纪录

其实在 `/etc/lighttpd/lighttpd.conf` 文件中，这部分内容写在多行，方便用 **`#`** 作注释，禁用不需要的模块

```shell
server.modules              = (
## 基础模块
    				"mod_access",
## 访问纪录
    				"mod_accesslog" )   
## fastcgi 支持
    				"mod_fastcgi",
## cgi 支持
#                               "mod_cgi",
## 路径绑定
#                               "mod_alias",
##  代理 (转发页面)
#                               "mod_proxy",
## 虚拟主机
#                               "mod_evhost",
## 输出压缩
#                               "mod_compress",
## 网址重写
#                               "mod_rewrite",
## 用户认证
#                               "mod_auth",
#                               "mod_redirect",
#                               "mod_cml",
#                               "mod_trigger_b4_dl",
#                               "mod_status",
#                               "mod_setenv",
#                               "mod_simple_vhost",
#                               "mod_userdir",
#                               "mod_ssi",
#                               "mod_usertrack",
#                               "mod_expire",
#                               "mod_secdownload",
#                               "mod_rrdtool", 
```

#### fastcgi配置

在配置文件的**`server.modules`**字段中启用**`mod_fastcgi`**模块，然后检查以下内容：

```shell
### fastcgi 脚本扩展名
static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )      

### fastcgi 服务器设置      
fastcgi.server             = ( ".php" =>
                               ( "localhost" =>
                                 (
# TCP/IP 接口 (“套接字”)
                                   "socket" => "/tmp/php-fastcgi.socket",
# PHP cgi 模式的可执行文件(PHP 有 cli 和 cgi 两种模式)
                                   "bin-path" => "/usr/bin/php-cgi"
                                 )
                               )
                            )
      
```

上面例子的第二部分，使用 Lighttpd 转发规则。大意为： **`.php`**文件按以下方式处理 => 从**`localhost`**(本地)，发送到**`/tmp/php-fastcgi.socket`**接口，使用**`/usr/bin/php-cgi`**处理。写成一行比较直观：

```shell
    fastcgi.server  = ( ".php" => ( "localhost" => ( "socket" => "/tmp/php-fastcgi.socket", "bin-path" => "/usr/bin/php-cgi" )))
```

如果想要 fastcgi 和 PHP 协同工作，还需要对 PHP 作一些设置

#### proxy

该模块可以将文件转发到其它服务器进行处理，例如将`.jsp`文件转发到Tomcat服务器

```shell
### 首先启用 mod_proxy 模块
# += 表示在原来设置上增加
servers.modules +=( "mod_proxy")

### 设置 proxy 服务器转发规则    
proxy.server               = ( ".jsp" =>
                               ( "localhost" =>
                                 (
# 将 .jsp 文件发送到 地址“127.0.0.1”的“8080”端口(也就是本机的 Tomcat 服务器)
                                   "host" => "127.0.0.1",
                                   "port" => 8080
                                 )
                               )
                             )    

```

#### CGI

Lighttpd 可以支持 cgi

```shell
### 启用 mod_cgi 模块
server.modules              += ("mod_cgi")

### 设置 cgi 解释器
cgi.assign                 = ( ".pl"  => "/usr/bin/perl",
                               ".cgi" => "/usr/bin/perl",
                               ".py"  => "/usr/bin/python" )    
```

#### 路径绑定

将一个路径，映射到网站目录中

```shell
## 启用 mod_alias 模块
servers.modules +=( "mod_alias")    

##  将 /home/lighttpd/html/man 映射到 http://host/docs
alias.url += ( "/docs" => "/home/lighttpd/html/man" )    
```

#### 虚拟主机

Lighttpd 可以建立多个虚拟主机，绑定在不同的网络接口

```shell
### 启用 mod_evhost 模块
servers.modules +=( "mod_evhost")

### 虚拟主机绑定的网络接口
$HTTP["host"] == "192.168.1.2" 
{
### 虚拟主机可以使用独立的选项
dir-listing.activate       = "enable"
dir-listing.encoding       = "utf8"
dir-listing.show-readme       = "enable"
### 虚拟主机根目录
server.document-root = "/home/user/html"
### 虚拟主机路径绑定
alias.url = ( "/download/" => "/home/user/downloads/" )
alias.url += ( "/pictures/" => "/home/user/pictures/" )
}    
```

### PHP&MySQL

PHP的配置文件为/etc/php/php.ini，如果与Lighttpd配合使用，请检查下面语句

```shell
cgi.fix_pathinfo=1
```

在web服务器根目录下新建一个index.php文件，内容如下：

```shell
<?php phpinfo(); ?>
```

与Lighttpd和PHP一样，MySQL也进入了大多数发行版的软件仓库，使用包管理系统安装。

MySQL安装后，可能需要重设密码

```shell
## 停止MySQL服务器
/etc/init.d/mysqld stop
## 使用单用户维护模式运行
mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
## 使用root身份，进入名为mysql的数据库
mysql -u root mysql 
## 更新表user中，USER项 值为‘root’的行
## 设定这一行Password项的值，PASSWORD()函数用来给密码加密
mysql > UPDATE user SET Password=PASSWORD('这里设置密码') where USER='root';
mysql > FLUSH PRIVILEGES;
mysql > quit 
## 重启MySQL服务器
/etc/init.d/mysqld restart
```

### PureFTPD

相对于ProFTPD、WuFTPD等老牌 ftp 服务器，PureFTPD、vsftpd这些轻量级 ftp 服务器更加实用

PureFTPD 的配置文件通常为 `/etc/pure-ftpd.conf`（查看`/etc/init.d/pure-ftpd`文件，可以看到类似字句：
**`/usr/sbin/pure-config.pl /etc/pure-ftpd.conf`** 启用配置文件）

```shell
## 绑定的地址和端口(默认为所有 IP地址 的 21端口)
# Bind                      127.0.0.1,21 

## 将所有用户限制在主目录中 (不能跳出)
ChrootEveryone              yes   

## 如果前一个设置为 no，下面组的成员(GID)可以跳出主目录；其他用户仍然受限
## 如果想取消所有限制，注释掉 ChrootEveryone 和 TrustedGID 
# TrustedGID                    100 

## 如果用户主目录不存在，自动创建
# CreateHomeDir               yes 

## 新建目录及文件的权限掩码(禁止的权限)。文件掩码:目录掩码 
Umask                       133:022 

## 被动模式响应的端口范围
# PassivePortRange          30000 50000

## 强制一个 IP地址 使用被动模式
# ForcePassiveIP                192.168.0.1 

## 仅允许认证用户进行 FXP 传输 (服务器=>服务器)
AllowUserFXP                yes

## 允许匿名 FXP 传输
AllowAnonymousFXP           no 

## 兼容 ie 等山寨 ftp 客户端
BrokenClientsCompatibility  no 

## 服务器允许的最大连接数
MaxClientsNumber            50 

## 同一 IP 允许的最大连接数
MaxClientsPerIP             8 

## 匿名用户最大带宽（KB/s）
# AnonymousBandwidth            8

## 所有用户最大带宽（KB/s），包括匿名用户
# UserBandwidth             8

# 匿名用户的上传/下载的比率
# AnonymousRatio                1 10

# 所有用户的上传/下载的比率
# UserRatio                 1 10 

## 客户端的最大空闲时间（缺省15分钟，无动作将会被踢出）
MaxIdleTime                 15 

## 不允许认证用户 (仅作为一个公共的匿名 FTP)
AnonymousOnly               no

## 不允许匿名连接，仅允许认证用户使用 (设置为 no 允许匿名连接)
NoAnonymous                 no   

## 受信地址允许认证用户，其它地址只能匿名连接
#TrustedIP                  10.1.1.1 

## 禁止匿名上传( no = 允许上传)
AnonymousCantUpload         no 

## 是否允许匿名用户创建目录
AnonymousCanCreateDirs      no 

## 'ls' 命令的递归限制。(文件的最大数目 最大子目录深度)
LimitRecursion              2000 8 

## 保留所有文件(禁止删除)
# KeepAllFiles                yes 

## 如果上传的文件已经存在，自动重命名
AutoRename                  no

## 禁止重命名
# NoRename 		      yes 

## 禁止更改文件权限
# NoChmod                     yes 

## 禁止读取隐藏文件（如 .history, .ssh 等)
ProhibitDotFilesRead        no 

## 禁止下载所有者为 "ftp" 的文件 (匿名用户上传后未被本地管理员验证的文件)
AntiWarez                   yes 

## 指定文件内容作为欢迎信息
# FortunesFile              /usr/share/fortune/zippy 

## 启用磁盘限额。第一个数字为最大文件数，第二个数字为存储空间大小(单位：Mb)
## 1000:10 限制每一个用户只能使用 1000 个文件，共 10Mb
# Quota                       1000:10 

## 最大可用空间，保证日志文件不会被覆盖 (默认为 99%)
MaxDiskUsage               99  
```

PureFTPD 允许同时使用多种用户认证方式。以下为关于用户认证的配置：

```shell
## LDAP 配置文件 
# LDAPConfigFile                /etc/pureftpd-ldap.conf

## MySQL 配置文件
# MySQLConfigFile               /etc/pureftpd-mysql.conf

## Postgres 配置文件 
# PGSQLConfigFile               /etc/pureftpd-pgsql.conf

## PureDB 用户数据库 
# PureDB                        /etc/pureftpd.pdb

## pure-authd 套接路径
# ExtAuth                       /var/run/ftpd.sock

## 启用 PAM 认证
# PAMAuthentication             yes

# 启用 Unix 系统认证 (/etc/passwd)
# UnixAuthentication            yes   
```

PureFTPD可以使用syslog生成日志，以下为关于日志的配置：

```shell
## Syslog 日志 。默认为 "ftp"， "none" 禁用日志
SyslogFacility              ftp 

## 在日志文件中不解析主机名
DontResolve                 yes   

## 在日志中添加 PID  
# LogPID                     yes

## 使用 Apache 格式创建额外日志
# AltLog                     clf:/var/log/pureftpd.log

## 使用优化格式创建额外日志
# AltLog                     stats:/var/log/pureftpd.log

## 使用 W3C 格式创建额外日志 
# AltLog                     w3c:/var/log/pureftpd.log   
```


参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)
