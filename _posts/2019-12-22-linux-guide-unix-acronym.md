---
layout: post
title: "开源世界旅行手册：Unix缩写风格"
date:  2019-12-22 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## Unix缩写风格

### 缩写习惯

构建于图形界面之上的操作系统，使用鼠标作为主输入设备，是否使用缩写并不重要。比如Windows系统中的目录，几乎都是全称。点击两次鼠标进入文件夹**pf**，并不意味着点击13次才能进入文件夹**Program Files**

而构建于命令行之上的操作系统，如Linux，只要3个字母以上的单词，几乎都要缩写。例如：`cd`命令是**Change Directory**的缩写。作为常用命令，如果使用它的全称**Change Directory**，绝对是无聊和乏味的工作。

最常见的缩写，取每个单词的首字母。如下

| 缩写 | 实际意义                    |
| ---- | --------------------------- |
| cd   | Change Directory            |
| dd   | Disk Dump                   |
| df   | Disk Free                   |
| du   | Disk Usage                  |
| pwd  | Print Working Directory     |
| ps   | Processes Status            |
| PS   | Prompt Strings              |
| su   | Substitute User             |
| rc   | Run Command                 |
| Tcl  | Tool Command Language       |
| cups | Common Unix Printing System |
| apt  | Advanced Packaging Tool     |
| bg   | BackGround                  |
| ping | Packet InterNet Grouper     |

如果首字母后为“h”。通常保留

| 缩写  | 实际意义           |
| ----- | ------------------ |
| chsh  | CHange SHell       |
| chmod | CHange Mod         |
| chown | CHange OWNer       |
| chgrp | CHange GRouP       |
| bash  | Bourne Again SHell |
| zsh   | Z SHell            |
| ksh   | Korn SHell         |
| ssh   | Secure SHell       |

递归缩写（定义中包含自身缩写，如：GNU，GNU's Not Unix，使用这个定义来解释定义中的缩写：

(GNU's Not Unix)'s Not Unix

这意味着它是可以无限递归的：

(((((GNU's Not Unix)'s Not Unix)'s Not Unix)'s Not Unix)'s Not Unix)'s Not Unix ……）也属于这一类，如

| 缩写 | 实际意义                    |
| ---- | --------------------------- |
| GNU  | GNU's Not Unix              |
| PHP  | PHP: Hypertext Proprocessor |
| RPM  | RPM Package Manager         |
| WINE | WINE Is Not an Emulator     |
| PNG  | PNG's Not GIF               |
| nano | Nano's ANOther editor       |

有些缩写可能有多种定义，如

**rpm**

- RPM Package Manager
- RedHat Package Manager

**bc**

- Basic Calculator
- Better Calculator

这方面Emacs可谓是独领风骚：

**Emacs**

- Editor MACroS
- Emacs Makes A Computer Slow
- Escape Meta Alt Control Shift
- Emacs Makers Are Crazy Sickos
- Emacs Makes All Computing Simple
- Emacs Makefiles Annihilate C-Shells
- Emacs Manuals Always Cause Senility
- Emacs May Allow Customized Screwups
- Emacs Manuals Are Cryptic and Surreal
- Eventually Munches All Computer Storage
- Eight Megabytes And Constantly Swapping
- Elsewhere Maybe All Commands are Simple
- Excellent Manuals Are Clearly Suppressed
- Emacs May Alienate Clients and Supporters
- Except by Middle Aged Computer Scientists
- Extended Macros Are Considered Superfluous
- Every Mode Accelerates Creation of Software
- Each Manual's Audience is Completely Stupefied
- Exceptionally Mediocre Algorithm for Computer Scientists
- Easily Maintained with the Assistance of Chemical Solutions
- Eradication of Memory Accomplished with Complete Simplicity  

如果只有一个单词，通常取每个音节的首字母：

| 缩写 | 实际意义 |
| ---- | -------- |
| cp   | CoPy     |
| ln   | LiNk     |
| ls   | LiSt     |
| mv   | MoVe     |
| rm   | ReMove   |

对于目录，通常使用前几个字母作为缩写：

| 缩写 | 实际意义              |
| ---- | --------------------- |
| bin  | BINaries              |
| dev  | DEVices               |
| etc  | ETCetera              |
| lib  | LIBrary               |
| var  | VARiable              |
| proc | PROCesses             |
| sbin | Superuser BINaries    |
| tmp  | TeMPorary             |
| usr  | Unix Shared Resources |

这种缩写的其他情况

| 缩写   | 实际意义           |
| ------ | ------------------ |
| diff   | DIFFerences        |
| cal    | CALendar           |
| cat    | CATenate           |
| ed     | EDitor             |
| exec   | EXECute            |
| tab    | TABle              |
| regexp | REGular EXPression |

如果某种缩写比较深入人心，例如：“mesg”代表“message”，在新的复合缩写中，将沿用这种缩写方式

| 缩写   | 实际意义           |
| ------ | ------------------ |
| dmesg  | Diagnostic MESsaGe |
| sed    | Stream EDitor      |
| stty   | Set TTY            |
| fstab  | FileSystem TABle   |
| passwd | PASSWorD           |

有些缩写，第一个字母“g”，代表“GNU”

| 缩写  | 实际意义                     |
| ----- | ---------------------------- |
| awk   | Aho Weiberger and Kernighan  |
| gawk  | GNU AWK                      |
| gpg   | GNU Privacy Guard            |
| grep  | GNU Regular Expression Print |
| egrep | Extended GREP                |

### 命令选项，从a到z

Linux命令的选项繁复庞杂，让人眼花缭乱。不过这些选项往往具有相对固定的含义，熟悉了它们，记忆便不再困难。

**-a**

- all：所有（ls, lsattr, uname）
- archive：存档（cp，rsync）
- append：附加（tar -A， 7z）

**-b**

- blocksize：块大小，带参数（du，df）
- batch：批处理模式（交互模式的程序通常拥有此选项，如top -b）

**-c**

- commands：执行命令，带参数（bash，ksh，python）
- create：创建（tar）

**-d**

- debug：测试
- delete：删除
- directory：目录（ls）

**-e**

- execute：执行，带参数（xterm，perl）
- edit：编辑
- exclude：排除

**-f**

- force：强制，不经确认（cp，rm，mv）
- file：文件，带参数（tar）
- configuration file：指定配置文件（有些守护进程拥有此选项，如ssh，lighttpd）

**-g**

**-h**

- --help：帮助
- human readable：人性化显示（ls，du，df）
- headers：头部

**-i**

- interactive：交互模式，提示（rm，mv）
- include：包含

**-k**

- keep：保留
- kill

**-l**

- long listing format：长格式（ls）
- list：列表
- load：读取（gcc，emacs）

**-m**

- message：消息（cvs）
- manual：手册（whereis）
- create home：创建home目录（usermod，useradd）

**-n**

- number：行号、编号（cat，head，tail，pstree，lspci）
- no：（useradd，make）

**-o**

- output：输出（cc，sort）
- options：选项（mount）

**-p**

- port：端口，带参数（很多网络工具拥有此选项，如ssh，lftp）
- protocal：协议，带参数
- passwd：密码，带参数

**-q**

- queit：静默

**-r**

- reverse：反转
- recursive：递归（cp，rm，chmod -R）

**-s**

- silent：安静
- size：大小，带参数
- subject

**-t**

- tag
- type：类型（mount）

**-u**

- user：用户名、UID，带参数

**-v**

- verbose：冗长
- version：版本

**-w**

- width：宽度
- warning：警告

**-x**

- exclude：排除（tar，zip）

**-y**

- yes

**-z**

- zip：启用压缩（bzip，tar，zcat，zip，cvs）
------

参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)