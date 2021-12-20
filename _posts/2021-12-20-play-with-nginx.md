---
layout: post
title: "玩转Nginx"
date:  2021-12-20 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 服务器
  - Shell
  - Linux
---
## 玩转Nginx

### 概述

Nginx（发音同“engine x”）是一个高性能的反向代理和Web服务器软件，因其系统资源消耗低，运行稳定且具有高性能的并发处理能力等特性，Nginx在互联网企业中得到广泛应用。

#### 特点

- 高性能、高并发
- 扩展性好
- 异步非阻塞的事件驱动模型

### 安装&使用

#### 安装

1. Windows
   - 下载官方稳定版： https://nginx.org/download
   - 解压到本地，直接运行nginx.exe即可
2. Linux
   - sudo apt-get install nginx
   - yum install nginx 
   - brew install nginx

#### 使用

| 命令参数        | 含义                             |
| --------------- | -------------------------------- |
| nginx           | 启动                             |
| nginx -V        | 查看当前版本及编译配置信息       |
| nginx -t        | 检查配置文件语法是否正确         |
| nginx -s stop   | 直接关闭woker子进程              |
| nginx -s quit   | 等待worker子进程正确处理完后关闭 |
| nginx -s reload | 重读配置文件                     |

### 实用场景

#### 虚拟主机

```shell
server {
	# 1. 基于多ip的虚拟主机：listen监听不同网卡的ip，端口可相同
	listen 8000；
	server_name 172.17.1.1;
	
	listen 8000;
	server_name 172.17.1.2;
	
	# 2. 基于多端口的虚拟主机：listen监听不同端口
	listen 8001;
	server_name localhost;
	
	listen 8002;
	server_name localhost;
	
	# 3. 基于域名的虚拟主机：端口可相同，server_name为不同的域名
	listen 8003;
	server_name www.test1.com;
	
	listen 8003;
	server_name www.test2.com;
}
```

#### 静态站点

为了加快网站解析速度，可以将动态资源交给后端服务器，纯前端的静态页面放在系统目录下，交给Nginx来解析。

![image-20210626210446827](/img/in-post/2021-12-20-play-with-nginx-01.png)

```shell
server {
		listen 80;
		server_name localhost;
		
		location / {
				root /opt/nginx/html;
				index index.html index.htm;
		}
}
```

#### 反向代理

反向代理是用户客户端访问代理服务器后，被反向代理服务器按照一定的规则从一个或多个被代理服务器中获取响应资源并返回给客户端的代理模式，客户端只知道代理服务器的IP，并不知道后端服务器的IP，原因是代理服务器隐藏了被代理服务器的信息。

##### 七层反向代理

在配置文件nginx.conf中的http段中，写入如下格式的配置，即可将本地8088端口代理到百度。

```shell
server {
		listen 8088;
		server_name localhost;
		
		location / {
				proxy_pass https://www.baidu.com;
		}
}
```

##### 四层反向代理

Nginx除了可以代理HTTP七层流量，还可以代理TCP/UDP四层流量，核心模块stream需要在编译配置时增加“--with-stream”参数进行编译

配置文件如下（需写在main段中）

```shell
stream{
		server {
				listen 3306;
				# 访问本机的3306，就被转发到了远程的3306
				proxy_pass 172.17.0.1:3306;
		}
}
```

#### 负载均衡

当出现高并发大流量的业务场景时，单台后端服务器已无法支撑业务正常运行，需要将请求流量按照一定的规则分发到多台服务节点上，即使某个节点宕机，系统依然能够对外正常提供服务，以此来提高系统的性能和稳定性。

![image-20210626210446828](/img/in-post/2021-12-20-play-with-nginx-02.png)

支持协议图

![image-20210626210446829](/img/in-post/2021-12-20-play-with-nginx-03.png)

#### HTTPS加密传输

HTTPS通过加密通道保护客户端与服务端之间的数据传输，已成为当前网站部署的必选配置。在部署有Nginx代理集群的HTTPS站点，通常会把SSL证书部署在Nginx的服务器上，然后把请求代理到后端的上游服务器。这种部署方式由Nginx服务器负责SSL请求的运算，相对减轻了后端上游服务器的CPU运算量。

- 生成自签名HTTPS证书

```shell
# 配置https签名证书
# 1. 创建https证书存放目录：
cd /usr/local/nginx/conf/
mkdir ssl 
# 2. 创建私钥：
openssl genrsa -des3 -out https.key 1024
# 3. 创建签名请求证书：
openssl req -new -key https.key -out https.csr 
# 4. 在加载SSL支持的Nginx并使用上述私钥时除去必须的口令：
cp https.key https.key.org
openssl rsa -in https.key.org -out https.key 
# 5. 最后标记证书使用上述私钥和CSR和有效期
openssl x509 -req -days 365 -in https.csr -signkey https.key -out https.crt
```

- server配置

```shell
server {
		listen 443 ssl;
		server_name localhost;
		
		# 证书部分
		ssl_certificate /usr/local/nginx/conf/ssl/https.crt; #RSA证书
		ssl_certificate_key /usr/local/nginx/conf/ssl/https.key; #RSA密钥
		
		# TLS握手优化
		# 会话缓存的存储大小为1MB
		ssl_session_cache shared:SSL:1m;
		# 会话缓存的超时时间为5分钟
		ssl_session_timeout 5m;
		keepalive_timeout 75s;
		keepalive_requests 100;
		location / {
				root html;
				index index.html index.htm;
		}
}
```

#### 文件服务器

要归档一些数据或资料，那么文件服务器必不可少。使用Nginx可以非常快速便捷的搭建一个简易的文件服务。

- 配置

```shell
server {
		listen 8004;
		server_name localhost;
		
		# 正常显示中文，windows服务器下中文目录无法下钻，目前无解
		charset gbk,utf-8;
		
		# 打开autoindex功能，以/结尾的请求
		autoindex on;
		
		# 显示文件的大小
		# on：以字节显示
		# off：人性化显示，文件过大会显示为mb或者gb
		autoindex_exact_size off;
		
		# 以哪种格式返回：html｜xml｜json｜jsonp
		# 默认值：autoindex_format html
		autoindex_format html;
		
		# 显示时间格式
		# on: 12-Jul-2019 10:11 (当前时区)
		# off: 12-Jul-2019 02.11 (0时区，GMT)
		autoindex_localtime on;
		
		location / {
				root /data/files/;
				# 如果a.html文件存在，则会返回a.html内容，否则才会返回目录内容
				index a.html;
		}
}
```

#### 限流

- limit_conn
  - 用于限制客户端并发连接数
  - 使用共享内存，对所有的worker子进程生效（需要保存客户端连接数）
- limit_req
  - 用于限制客户端处理请求的平均速率
  - 使用共享内存，对所有的woker子进程生效
  - 限流算法：leaky_bucket(漏桶)
    - 暂时拦截住上方水的向下流动，等待桶中的一部分水漏走后，再放行上方水
    - 溢出的上方水直接抛弃。

```shell
http {
		include mime.types;
		default_type application/json;
		
		# limit_conn_zone key zone=name:size
		# key: 用于定义客户端的唯一标识来限速，如remote_addr
		# name: 任意名称
		# size: 共享内存大小空间，m为单位
		# binary_remote_addr 使用4个字节空间，高效； remote_addr使用7-15个字节空间
		limit_conn_zone $binary_remote_addr zone=limit_addr:10m;
		
		# limit_req_zone key zone=name:size rate=rate;
		# 上下文:http
		# rate:表示允许相同标识的客户端的访问频次，12r/m的即限制每5秒访问一次，每5秒才处理一个请求
		limit_req_zone $binary_remote_addr zone=limit_req:15m rate=12r/m;
		
		server {
				listen 80;
				server_name localhost;
				
				location / {
						root html;
						index index.html index.htm;
						
						# 触发限速后，返回状态码，默认503
						# 上下文：http, server, location
						limit_conn_status 503;
						
						# 当触发限速后，错误日志出记录一条日志， 这里用于定义日志等级
            # info|notice|warn|error
            # 上下文：http, server, location
            # 默认值：error
						limit_conn_log_level warn;
						
						# limit_conn zone number
						# zone: 用limit_conn_zone 中定义的zone名称
						# number: 以zone为标识的客户端被允许的同时最大连接数
						limit_conn limit_addr 2;
						
						# 定义响应数据的传输速度，bytes/s
						# 本指令属于ngx_http_core_module, 不属于ngx_http_limit_conn_module
						limit_rate 50;
						
						# limit_req_status code(http的状态码)
						# 默认值: 503
						# 上下文: http, server, location
						limit_req_status 504;
						
						# 触发限制访问后，日志记录的等级
						# info|notice|warn|error
						# 默认值：error
						# 上下文: http, server, location
						limit_req_log_level notice;
						
						# limit_req zone=name [burst=number] [nodelay | delay=number];
						# burst: 桶大小,设置一个大小为x的缓冲区，当有大量请求过来时，超过了访问频次限制的请求可以先放到这个缓冲区内等待，但是这个等待区里的位置只有5个，超过的请求会直接报503错误然后返回
						# nodelay: 如果设置，会在瞬时提供处理(burst+rate)个请求的能力，请求超过（burst+rage）的时候就会直接返回503，永远不存在请求需要等待的情况
						# 上下文: http, server, location
						# limit_req zone=limit_req burst=7 nodelay;
						limit_req zone=limit_req;
				}
		}
}
```

#### 黑白名单

- access	
  - 限制特定IP或网段访问
  - allow
  - deny

```shell
server {
		listen 80;
		server_name localhost;
		location / {
				# allow address | CIDR | UNIX | all
				# 默认值
				# 上下文：http，server，location，limit_except
				allow 192.168.0.1/24;
				
				# deny address | CIDR | UNIX | all 
				# 默认值
				# 上下文：http，server，location，limit_except
				deny all;
		}
}
```

- 规则示例

```shell
location / {
		# 规则从上到下
		
		# 拒绝
		deny 192.168.1.1;
		
		# 放行192.168.1.0网段，子网掩码24位（255.255.255.0），但是除了192.168.1.1
		allow 192.168.1.0/24;
		
		# 放行10.1.1.0网段，子网掩码16位（255.255.0.0）
		allow 10.1.1.0/16;
		
		# 放行ipv6
		allow 2001:0db8::/32;
		
		# 除了上面放行的，其他全部拒绝
		deny all;
}
```

#### 请求拦截

- auth_request
  - 基于子请求收到的HTTP响应码做访问限制
    - 如：拦截所有请求，先去做鉴权请求，通过后再放行

```shell
location /private {
		# 默认值 off
		# 上下文：http, server, location
		# 鉴权成功会返回后面实际内容，鉴权失败会返回鉴权服务的返回内容
		auth_request /auth;
		...
}

location /auth {
		proxy_pass http://localhost:8080/auth;
		proxy_pass_request_body off;
		proxy_set_header Content-Length "";
		proxy_set_header X-Original-URI $request_uri;
}
```

### 配置&变量详解

#### 全局配置main段

```shell
# 核心参数
# user USERNAME [GROUP]
# 解释：指定运行nginx的workder子进程的属主和属组，其中属组可以不指定
user nginx;

# worker_processes NUMBER | auto 
# 解释：指定nginx启动的worker子进程数量
# 【*auto：自动设置为物理CPU核心数】
worker_processes auto;

# pid DIR 
# 解释：指定运行nginx的master主进程的pid文件存放路径
pid /opt/nginx/logs/nginx.pid;

# worker_rlimit_nofile NUMBER
# 解释：指定worker子进程可以打开的最大文件句柄数
# 【系统最大打开65535，每个子进程打开数乘子进程数，实际也不会超过65535】
# 这个值需要调大
worker_rlimit_nofile 20480;

# worker_rlimit_core SIZE
# 指定worker子进程一场终止后的core文件，用于记录分析问题
worker_rlimit_core 50M;
working_directory /opt/nginx/tmp; #【必须对子进程用户赋写权限】

# 【解释】：将每个worker子进程与CPU物理核心绑定
# 【master负责调度，worker负责处理请求】
# 【假设CPU有4个核心，某一时刻worker1获取到了CPU1的工作调度时间片，时间片过后worker1从CPU1上面撤下来，CPU1去处理其他事件，下一时刻可能是CPU2、CPU3的时间片调度到了worker1上面，那么worker1就会在其他CPU上面工作，进程和CPU的调度切换是有损耗的，worker1如果绑定了CPU1，worker1将永远等待CPU1的调度，充分利用CPU缓存】
# 【主要作用：将每个worker子进程与特定CPU物理核心绑定，优势在于：避免同一个worker子进程在不同的CPU核心上切换，缓存失效，降低性能；其并不能真正避免进程切换（进程切换是CPU工作特性）】
# -- worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000; 8核心，8个worker
# -- worker_cpu_affinity 01 10 01 10; # 2核心，4个worker
worker_apu_affinity 0001 0010 0100 1000; # 4核心，4个worker

# 解释：指定worker子进程的nice值，以调整运行nginx的优先级，通常设定为“负值”，以优先调用nginx
# 【Linux默认进程的优先级值是120，值越小越优先；nice设定范围-20到+19】
# 【对Linux来说，优先级值则是100到139】
worker_priority -20;

# 指定worker子进程优雅退出时对超时时间，不管5秒内是否处理完，都强制退出
worker_shutdown_timeout 5s;

# worker子进程内部使用的计时器精度，调整时间间隔越大，系统调用越少，有利于性能提升；反之，系统调用越多，性能下降
# 比如某些计时的操作，worker需要去获取内核时间，频繁跟内核打交道会降低性能
timer_resolution 100ms;

# daemon on | off
# 设定nginx的运行方式，前台还是后台，前台用户调试，后台用于生产
daemon on;

# 负载均衡互斥锁文件存放路径
lock_file logs/nginx.lock;
```

#### events段

```shell
events {
		# Nginx使用何种事件驱动模型，一般不指定这个参数
		# use epoll;
		
		# worker子进程能够处理的最大并发连接数，多核情况最大其实达不到65535
		worker_connections 65535;
		
		# 是否打开负载均衡互斥锁，默认off（当master接收到请求时，会给每个worker发送消息去唤醒，状态为on时，则会有一个负载均衡锁，master会轮流发给每一个）
		accept_mutex on;
		
		# 新连接分配给worker子进程的超时时间，默认500ms，超时后会转给下一个worker处理请求
		accept_mutex_delay 100ms;
		
		# worker子进程可以接收的新连接个数（这个参数对性能影响不大）
		multi_accept on;
}
```

#### http段

- server段

```shell
server {
		listen 80;
		server_name www.test.com;
		location /picture {
				root /opt/nginx/html/picture;
				# 客户端请求 www.test.com/picture/1.jpg;
				# 对应磁盘映射路径为：/opt/nginx/html/picture/picture/1.jpg
		}
		location /picture {
				alias /opt/nginx/html/picture/;
				# 客户端请求 www.test.com/picture/1.jpg;
				# 对应磁盘映射路径为：/opt/nginx/html/picture/1.jpg
		}
}
```

- server_name的匹配规则

```shell
# 精确匹配，优先级最高，1 
server_name www.test.com;

# 左通配,优先级2
server_name *.test.com;

# 右通配，优先级3
server_name www.test.*;

# 正则通配，优先级最低，4
server_name ~^w\.test\..*$;

# 多个
server_name www.test.com *.test.com www.test.* ~^w\.test\..*$;

```

- location段

| 匹配规则 | 含义                   | 示例                      | 优先级（1最高） |
| -------- | ---------------------- | ------------------------- | --------------- |
| =        | 精确匹配               | location=/pic/            | 1               |
| ^~       | 匹配到即停止搜索       | location ^~/pic/          | 2               |
| ~        | 正则匹配，区分大小写   | location ~ `\.(Jpg|gif)#` | 3               |
| ~*       | 正则匹配，不区分大小写 | location ~ `\.(Jpg|gif)#` | 4               |
| 无符号   |                        | location /                | 5               |
| @        | 内部跳转               | location @errorpage       |                 |

```shell
# 测试样例
location ~ /test/8005/t/$ {
		return 200 'first regular expressions match!';
}
location ~* /test/8005/t/(\w+)$ {
		return 200 'longest regular expressions match!';
}
location ^~ /test/8005/t/ {
		return 200 'stop regular expressions match!';
}
location /test/8005/t/Test2 {
		return 200 'longest prefix string match!';
}
location /test/8005/t {
		return 200 'prefix string match!';
}
location = /test/8005/t {
		return 200 'exact match!';
}
```

| root与alias的区别 | root                    | alias                    |
| ----------------- | ----------------------- | ------------------------ |
| 语法              | root path               | alias path               |
| 上下文            | http,server,location,if | location                 |
| 区别              | 将定义路径与URI叠加     | 只取定义路径，末尾一定要 |

| location末尾带与不带/的区别 |                 |                                            |
| --------------------------- | --------------- | ------------------------------------------ |
| 不带/                       | location /test  | 尝试把test当成目录，如果找不到则找test文件 |
| 带/                         | location /test/ | 将test作为目录，如果不存在则直接返回404    |

```shell 
location /status {
		#监控模块
		stub_status;
}

# 页面结果
Active connections: 2
server accepts handled requests
	16 16 26
Reading: 0 Writing: 1 Waiting: 1
```

| 状态项             | 含义                                                         |
| ------------------ | ------------------------------------------------------------ |
| Active connections | 当前客户端与Nginx间的TCP连接数，等于下面Reading、Writing、Waiting数量之和 |
| accepts            | 自Nginx启动起，与客户端建立过的连接总数                      |
| handled            | 自Nginx启动起，处理过的客户端连接总数。如果没有超出worker_connections配置，该值与accepts相同 |
| requests           | 自Nginx启动起，处理过的客户端请求总数。由于存在HTTP Keep-Alive请求，故requests值会大于handled值 |
| Reading            | 正在读取HTTP请求头部的连接总数                               |
| Writing            | 正在向客户端发送响应数据的连接总数                           |
| Waiting            | 当前空闲的HTTP Keep-Alive连接总数                            |

| 内嵌变量             |                        |
| -------------------- | ---------------------- |
| 变量名               | 含义                   |
| $connections_active  | 同Active connections值 |
| $connections_reading | 同Reading值            |
| $connections_writing | 同Writing值            |
| $connections_waiting | 同Waiting值            |

- rewrite指令&return指令
  - return
    - 停止处理请求，直接返回响应码或重定向到其他URL
    - 执行return指令后，location中后续指令将不会被执行
  - rewrite
    - 根据指定正则表达式匹配规则，重写URL

```shell
location / {
		# 上下文：server, location, if 
		# return code [text];
		# text: 响应体内容（如果code是200）
		# return 200 "return 200 HTTP Code";
		# return code URL;
		# URL: 重定向
		# return 302 /test;
		# return URL;
		# URL:直接跟URL的话必须是http/https开头的完整路径
		# text：响应体内容
		return http://localhost:8000/test
}

location /test {
		index test.html;
}

location /search {
		# rewrite regex replacement [flag]
		# 上下文：server, location, if
		# flag:
		#			last: 重写后的url发起新请求，再次进入server段，重试location中的匹配
		#			break: 直接使用重写后的url，不再匹配其他location中的语句
		#			redirect: 返回302临时重定向
		#     permanent: 返回301永久重定向
		rewrite /(.*) https://www.baidu.com permanent;
}

location /test1 {
		# 继续匹配location.
		rewrite /images/(.*) /test2/$1 last;
		return 200 "return 200 in /test1";
}

location /test2 {
		# 不会再匹配，直接找test3下面的文件
		rewrite /pics/(.*) /test3/$1 last;
		return 200 "return 200 in /test2";
}

location /test3 {
		# 请求：/test3/index.html,
		# 结果：直接返回“return 200 in /test3", 不会再去找index.html文件
		return 200 "return 200 in /test3";
}

location /test4 {
		if ($remote_addr = "192.168.1.1") {
				return 200 "test if OK in URL /test4/";
		}
}

location /test5 {
		if($uri = "/images/") {
				rewrite (.*) /test2/ break;
		}
		# 执行了上面rewrite后，这里的return还会执行，通常不会联合一起写
		return 200 "test5 if failed\n";
}
```

#### Nginx变量分类

- TCP连接相关变量

```shell
# 客户端地址，例如192.168.1.1remote_addr # 客户端端口，例如58473remote_port# 客户端地址的整型格式binary_remote_addr # 已处理连接，是一个递增的序号connection# 当前连接上执行的请求数，对于keepalive连接有意义connection_request # 如果使用proxy_protocol协议，则返回原始用户的地址，否则为空proxy_protocol_addr # 如果只用proxy_protocol协议，则返回原始用户的端口，否则为空proxy_protocol_port # 服务器地址，例如192.168.184.240server_addr# 服务器端口，例如80server_port # 服务端协议，例如HTTP/1.1server_protocol
```

- HTTP请求相关变量

```shell
# 请求包体头部长度content_length# 请求包体类型content_type # URL中某个参数arg_参数名# 所有URL参数args# URL中有参数，则返回？；否则返回空is_args# 与args完全相同query_string# 请求的URL，不包含参数uri# 请求的URL，包含参数request_uri# 协议名，http或者httpsscheme# 请求的方法，GET,HEAD,POST等request_method# 所有请求内容的大小，包含请求行，头部请求体request_length# 由HTTP Basic Authentication协议传入的用户名remote_user# 客户端请求主体信息的临时文件名request_body_file# 包含请求的主要信息，在使用proxy_pass或fastcgi_pass指令的location中比较有意义request_body # 先看请求行，再看请求头，最后找server_name host# 用户浏览器标识http_user_agent# 从哪些链接过来的请求http_referer# 经过一层代表服务器，添加对应代理服务器的信息http_via# 获取用户真实IPhttp_x_forwarded_for# 用户cookiehttp_cookie
```

- Nginx处理请求时相关变量

```shell
# 请求处理到现在所耗费的时间，单位为秒，例如0.03代表30毫秒request_time # 请求处理完成，则返回OK，否则为空request_completion# 16进制显示的请求id，随机生成的request_id# 匹配上请求的server_name值server_name # 若开启https，则值为on，否则为空https# 待访问文件的完整路径request_filename# 由URI和root/alias规则生成的文件夹路径document_root# 将document_root中的软链接换成真实路径realpath_root# 返回响应时的速度上限值limit_rate
```

- Nginx处理响应时相关变量

```shell
# 响应体中真实内容的大小body_bytes_sent# 全部响应体大小body_sent# HTTP返回状态码status
```

- 系统变量

```shell
# nginx系统版本nginx_version# 服务器时间time_local
```

#### 时间空间单位

- 时间单位
  - ms：毫秒
  - s：秒
  - m：分钟
  - h：小时
  - d：天
  - w：周
  - M：月
  - y：年
- 空间单位
  - k/K：KB
  - m/M：MB
  - g/G：GB

#### HTTP状态码

| 分类 | 描述                                           |
| ---- | ---------------------------------------------- |
| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2**  | 成功，操作呗成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

| 状态码 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 100    | 继续。客户端应继续其请求                                     |
| 101    | 切换协议。服务器根据客户端的请求切换协议。智能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
| 200    | 请求成功。一般用于GET与POST请求                              |
| 201    | 已创建。成功请求并创建了新的资源                             |
| 202    | 已接收。已经接收请求，但未处理完成                           |
| 203    | 非授权信息。请求成功。但返回的meta信息不再原始的服务器，而是一个副本 |
| 204    | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
| 205    | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
| 206    | 部分内容。服务器成功处理了部分GET请求                        |
| 300    | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301    | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302    | 临时移动。与301类似，但资源只是临时被移动，客户端应继续使用原有URI |
| 303    | 查看其他地址。与301类似。使用GET和POST请求查看               |
| 304    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305    | 使用代理。所请求的资源必须通过代理访问                       |
| 306    | 以及被废弃的HTTP状态码                                       |
| 307    | 临时重定向。与302类似。使用GET请求重定向                     |
| 400    | 客户端请求的语法错误，服务器无法理解                         |
| 401    | 请求要求用户的身份认证                                       |
| 402    | 保留，将来使用                                               |
| 403    | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
| 404    | 服务器无法根据客户端的请求找到资源。通过此代码，网站设计人员可以设置“你所请求的资源无法找到”的个性页面 |
| 405    | 客户端请求中的方法被禁止                                     |
| 406    | 服务器无法根据客户端请求的内容特性完成请求                   |
| 407    | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408    | 服务器等待客户端发送的请求时间过长，超时                     |
| 409    | 服务器完成客户端的PUT请求时可能返回此代码，服务器处理请求时发生了冲突 |
| 410    | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过310代码指定资源的新位置 |
| 411    | 服务器无法处理客户端发送的不带Content-Length的请求信息       |
| 412    | 客户端请求信息的先决条件错误                                 |
| 413    | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭链接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414    | 请求的URI过长（URI通常为网址），服务器无法处理               |
| 415    | 服务器无法处理请求附带的媒体格式                             |
| 416    | 客户端请求的范围无效                                         |
| 417    | 服务器无法满足Expect的请求头信息                             |
| 500    | 服务器内部错误，无法完成请求                                 |
| 501    | 服务器不支持请求的功能，无法完成请求                         |
| 502    | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应 |
| 503    | 由于超载或系统维护，服务器暂时的无法处理客户端的请求，延时的长度可包含在服务器的Retry-Alter头信息中 |
| 504    | 充当网关或代理的服务器，未及时从远端服务器获取请求           |
| 505    | 服务器不支持请求的HTTP协议的版本，无法完成处理               |




参考资料：

- [大话数据结构]()