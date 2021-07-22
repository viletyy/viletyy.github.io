---
layout: post
title: "开源世界旅行手册：编译工具链"
date:  2020-01-05 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## 编译工具链

### 标准编译安装

**为什么要编译安装**

包管理系统是绝大多数发行版的必备组件，也是一个发行版区别于其他发行版的主要特征。但是有些软件，并不能通过包管理系统安装，这就需要下载源码编译安装。

一个软件可能有许多功能，但是发行版中提供的安装包，通常只具有一些常见的功能。如果提供所有功能，那么无疑会占用更多的资源，而这些功能，大多数用户不会用到；而你会用到的功能，可能安装包中刚好没有。编译安装可以灵活的定制软件，选择自己需要的，取消自己不需要的。

编译安装还可以针对特定的软件进行优化，以获得更好的性能表现。

**编译环境**

编译环境包括多个工具，它们环环相扣，称作编译工具链。主要包括以下工具：

| 工具     | 简介                                         |
| -------- | -------------------------------------------- |
| binutils | 连接器、汇编器和其他用于目标文件和档案的工具 |
| gcc      | 编译器，将源代码转换为机器代码               |
| glibc    | C库，提供标准例程（C函数）                   |
| autoconf |                                              |
| automake | 自动生成Makefile文件                         |
| make     | 安装Makefile文件中的规则编译程序             |

**标准编译安装**

首先，下载源代码，通常是压缩包，比如：xxx.tar.gz或者xxx.tar.bz2，解包：

| 压缩包格式 | 命令                 |
| ---------- | -------------------- |
| .tar.gz    |                      |
| .tgz       | tar zxvf xxx.tgz     |
| .tar.bz2   | tar jxvf xxx.tar.bz2 |

通常解包后会在当前位置得到一个xxx/目录，进入这个目录

```shell
cd xxx/
```

使用下列命令编译安装：

```shell
./configure ① --prefix=/opt/xxx②
make③
sudo make install④
make clean⑤
```

① 配置软件特性，检查编译环境，生成Makefile文件

② 最常用配置选项：指定软件的安装路径

③ 根据Makefile编译源代码

④ 将编译完成的程序安装到系统中。通常需要root权限

⑤ 清除源代码目录中的编译结果

### 编译过程

将下面代码保存为 Hello.c:

```c
#include <stdio.h>
int main(void)
{
  printf("Hello World!\n")①;
  return 0;
}
```

① printf()函数

执行命令`cc Hello.c`，得到一个可执行文件a.out，执行它 ./a.out

可以看到，C的源代码（Hello.C）是纯文本，不能够直接执行。可执行代码是计算机的本机语言或机器语言表示的代码，这种语言是由数字代码表示的详细指令组成，不同的计算机具有不同的机器语言。

编译器是一个程序，其工作是将源代码转换为可执行代码。

- 编译器用来将C语言转换成特定的机器语言。
- 编译器还从C的库中向最终程序加入代码。
- 编译器还检查源代码是否为有效的C语言程序。如果编译器发现错误，将报告错误，而且不生成可执行文件

编译器分三步完成这个工作：

1. 预处理：调用与处理器cpp对源代码文件中的文件包含（include）、预编译语句（如宏定义define等）进行分析
2. 编译：调用编译器cc将源代码转换为中间代码
3. 链接：调用链接器ld将中间代码与其他代码结合起来生成可执行文件。

这种方法使用程序便于模块化。分别编译各个模块，然后使用链接器将编译过的模块结合起来。这样，如果需要改变一个模块，则不必重新编译所有其他模块。

可执行文件包含**目标文件**、**库例程**和**启动代码**

编译器将源代码转换为机器语言代码（中间代码），将结果放置在目标文件（*.o）中。虽然目标文件包含机器代码，但该文件还不能运行，它还不是一个完整的程序。

启动代码（start-up code) 相当于程序和操作系统之间的接口。

库例程为函数的实现。几乎所有C程序都利用标准库C库中所包含的例程，目标代码文件不包含这一函数的代码，它只包含调用函数的指令。实际代码存储在一个称为“库”的文件中。库文件中包含许多函数的目标代码

链接器的作用是将这3个元素（目标代码、系统的标准启动代码和库代码）结合在一起，并将它们存放在可执行文件中。

### gcc编译器

gcc是GNU推出的功能强大、性能优越的多平台编译器，是GNU的代表作品之一。它能将C、C++语言源程序、汇编语言源程序和目标程序编译、链接成可执行文件，如果没有给出可执行文件的名字，gcc将生成一个名为a.out的文件。

gcc通过后缀来区分输入文件的类型：

| 后缀          | 类型                       |
| ------------- | -------------------------- |
| .c            | C语言源代码文件            |
| .a            | 由目标文件构成档案库文件   |
| .C\|.cc\|.cxx | C++源代码文件              |
| .h            | 程序所包含的头文件         |
| .i            | 预处理过的C源代码文件      |
| .ii           | 预处理过的C++源代码文件    |
| .m            | Objective-C源代码文件      |
| .o            | 编译后的目标文件           |
| .s            | 汇编语言源代码文件         |
| .S            | 预编译的汇编语言源代码文件 |

前面我们已经使用gcc编译了一个程序：cc Hello.c

gcc还有许多选项：

| 选项      | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| -c        | 只编译，不链接成为可执行文件                                 |
| -o 文件名 | 设定输出文件名。默认为a.out                                  |
| -g        | 加入调试符号（默认）                                         |
| -O        | 编译、链接时进行优化，耗时比较多，但产生的可执行文件执行效率更高 |
| -O2       | 更高的优化级别，耗时更多                                     |

### 自动化编译

在前面的标准编译安装中，第一步是./configure，它会根据Makefile.in生成Makefile文件，然后make根据Makefile自动编译软件

通常在一个源码包中，已经包含了configure脚本和Makefile文件，作为课外知识，我们大致了解一下怎么生成这两个文件

**autoconf**

autoconf用来生成configure脚本，它可以检查系统特性、编译环境、环境变量、软件参数、依赖关系等

autoconf需要用到m4

1. 用autoscan扫描源代码目录生成configure.scan文件
2. 将configure.scan改名为configure.in
3. 用aclocal根据configure.in文件的内容，自动生成aclocal.m4文件
4. 使用autoconf，根据configure.in和aclocal.m4来产生configure文件

**automake**

automake可以从Makefile.am文件自动生成Makefile.in，它主要用来配置源代码

automake需用到perl

- 手工写Makefile.am
- 使用automake，根据configure.in和MakeFile.am来产生Makefile.in

**Makefile**

使用configure脚本，配合Makefile.in可以生成Makefile文件，然后用make自动化的编译软件

这里有一张生成Makefile的流程图



![image-20210722172633176](/img/in-post/2020-01-05-linux-guide-compile-toolkit-01.png)

Makefile的用途不只是编译软件，还可以利用它完成一些琐碎的工作，只要最后输出一个文件，都可以用make来完成

这是一个最简单的Makefile	

```shell
filelist①:②*③
④   ls -lF > filelist⑤
```

① 输出的目标文件，不能省略。如果有多个文件，可以使用all

② 分隔符，不能省略

③ 输入文件，可以省略

④ 这一行必须以TAB字符起始，不能使用空格代替

⑤ make的命令

可以使用变量代替命令，便于维护

```shell
TARGET = filelist①
SOURCE = * 
ARG = -lF
APPLICATION = ls

$(TARGET):$(SOURCE)②
    $(APPLICATION) $(ARG) $(SOURCE) > $(TARGET)
```

① 定义变量，传统上用大写

② 使用变量写Makefile

Makefile可以有多个目标文件，我们前面提到，gcc编译时先生成目标文件，再把目标文件链接成可执行文件，Makefile应该是这样的

```shell
OBJECTS = main.o kbd.o command.o display.o \①
            insert.o search.o files.o utils.o
exe : $(OBJECTS)
            cc -o exe $(OBJECTS)

main.o : main.c defs.h ②
        cc -c main.c 
kbd.o : kbd.c defs.h command.h 
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
```

① 如果写在多行，要用脱字符换行

② 如何生成中间文件

③ 伪目标文件，make clean 并不生成clean文件，而是清理编译结果

### 使用make

利用configure所产生的Makefile文件有几个预先设定的目标可供使用：

| 目标           | 用途                                                  |
| -------------- | ----------------------------------------------------- |
| make all       | 产生预设的目标，只敲入make也可以                      |
| make clean     | 清除编译结果                                          |
| make distclean | 出了清除编译结果，也把configure所产生的Makefile清除掉 |
| make install   | 将程序安装到系统中                                    |
| make dist      | 将程序和相关的文档打包为一个压缩文档以供发布          |
| make distcheck | 打包并检验                                            |

### emerge

虽然我们能够使用autoconf、automake、make等工具实现自动化编译，但这种针对单个软件包的编译系统，在编译多个软件时仍然十分繁琐

假设需要编译emacs和vim，使用xft字体、图形界面支持，并去掉调试符号，需要分别作如下配置

```shell
emacs 
./configure --prefix=/usr/local/    \
--no-debug                          \
--with-xft  --with-x-toolkit=gtk    \
--with-freetype
vim 
./configure --prefix=/usr/local/    \
--no-debug                          \
--with-xft --with-x-toolkit=gtk
```

这就像点菜时，你必须告诉厨师：鱼香肉丝（多放辣椒、不放蒜）、宫保鸡丁（多放辣椒、不放蒜）、麻婆豆腐（多放辣椒、不放蒜）......

实际上，大多数人这样点菜：鱼香肉丝、宫保鸡丁、麻婆豆腐......多放辣椒、不放蒜

emerge就是这样一种点菜方式，它是gentoo的包管理系统，提供了更为现代化的编译方式

可以通过制定USE标记xft、gtk、-debug来确定所有软件的编译方式

**设置USE标记**

以下方法按优先级由低到高排列：

/etc/make.profile/目录是一个符号链接，里面包含一些make.defaults文件，放置开发者设置的USE标记：

```shell
/usr/portage/profile/base/make.defaults
/usr/portage/profile/default-linux/make.defaults
/usr/portage/profile/default-linux/x86/make.defaults
/usr/portage/profile/default-linux/x86/2008.0/make.defaults
```

在/etc/make.conf文件中声明永久USE标记（推荐）

```shell
USE="nptl nptlonly nls cjk php mysql -kde -qt3 -qt4"
```

带-的USE标记，表示排除

在/etc/portage/package.use文件中为单个包声明USE标记

```shell
app-editors/emacs-cvs xft 
www-servers/lighttpd fastcgi
dev-lang/php mysqli cgi gd ctype pcre session unicode pic posix 
dev-db/phpmyadmin vhosts
app-shells/zsh doc 
net-ftp/pure-ftpd -ldap mysql pam ssl vchroot
```

使用环境变量声明临时USE标记

```shell
USE="-java"
emerge seamonkey
```

查看使用的USE标记：

```shell
merge --pretend --verbose seamonkey
Calculating dependencies ...done!
[ebuild		R		] www-client/seamonkey-1.0.7 USE="crypt gnome java -debug -ipv6
-ldap -mozcalendar -mozdevelop -moznocompose -moznoirc -moznomail -moznopango
-moznoroaming -postgres -xinerama -xprint"  0 kb
```

**编译选项**

/etc/make.conf

```shell
CFLAGS="-O2 -march=i686 -pipe"①
CXXFLAGS="-O2 -march=i686 -pipe"②

CHOST="i686-pc-linux-gnu"③
MAKEOPTS="-j2"④

FEATURES="parallel-fetch ccache" ⑤
CCACHE_DIR="/var/tmp/ccache"⑥
CCACHE_SIZE="2G"⑦

ACCEPT_KEYWORDS="x86"⑧

USE="nptl nptlonly nls cjk php mysql"⑨

FETCHCOMMAND="/usr/bin/axel -a -n4 \${URI} -o \${DISTDIR}" ⑩
RESUMECOMMAND="/usr/bin/axel -a -n4 \${URI} -o \${DISTDIR}"
```

① 针对C语言的优化选项，-march=设置目标架构

② 针对C++语言的优化选项

③ 进行编译工作的机器架构

④ 编译选项

⑤ emerge特性。并行下载、使用ccache缓冲编译结果

⑥ ccache缓存目录

⑦ ccache缓存大小

⑧ 通过关键字选择分支。x86表示x86架构的稳定分支，-x86表示x86架构的不稳定分支

⑨ USE标记

⑩ 使用axel加速下载

gentoo支持多种架构：x86、sparc、amd64、ppc、ppc64、alpha、hppa、mips、ia64、arm，我们使用的PC多为x86架构

假设你主要使用x86稳定分支，但少数软件要使用最新版本，在/etc/portage/package.keywords文件中为单个包设置关键字

```shell
app-editors/emacs-cvs -x86
x11-misc/emacs-desktop -x86
app-i18n/fcitx -x86
app-editors/vim x86
app-editors/vim-core x86

media-video/mplayer -x86
media-libs/win32codecs -x86

app-i18n/man-pages-zh_CN -x86
```

**微调**

在/etc/portage/目标下包含一些文件，可以在软件包级别上进行调节。前面已经介绍了package.use和package.keywords

- **package.keywords**还未被确认适合你的系统或机构，但是你希望能安装的软件包
- **package.use**特定软件包而不是整个系统使用的USE标记
- **package.provided**屏蔽的软件包（需要指明版本号）
- **package.mask**永远不希望Portage安装的软件包。
- **package.unmask**被Gentoo开发者屏蔽的软件包，但是你希望能安装的软件包

软件包可能由于以下原因被屏蔽

| 原因                       | 详细                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ~架构 keyword              | 意味着这个软件没有经过充分的测试，不能进入稳定分支，请等待一段时间后在尝试使用它 |
| -架构 keyword 或-* keyword | 意味着这个软件不能工作在您机器的体系结构中                   |
| missing keyword            | 意味着这个软件还没有在您机器的体系结构中进行过测试           |
| package.mask               | 意味着这个软件被认为是损坏的，不稳定的活着有更严重的问题，它被故意标识为"不应使用" |

例如：

```shell
gnome-base/gnome-2.8.0_pre1 (masked by: -86 keyword)
lm-sensors/lm-sensors-2.8.7 (masked by: -sparc keyword)
sys-libs/glibc-2.3.4.20040808 (masked by: -* keyword)
dev-util/cvsd-1.0.2 (masked by: missing keyword)
games-fps/unreal-tournament-451 (masked by: package.mask)
sys-libs/glibc-2.3.2-r11 (masked by: profile)
```

**使用emerge**

查找名字包含pdf的软件包

```shell
emerge --search pdf
```

查找与pdf相关的软件包

```shell
emerge --searchdesc pdf 
emerge -S pdf
```

查找软件拥有的USE标记

```shell
emerge -vp 软件包名称
```

安装软件包

```shell
emerge 软件包名称
```

模拟安装软件包

```shell
emerge --pretend 软件包名称
```

下载软件包的源代码包

```shell
emerge --fetchonly 软件包名称
```

从系统中删除软件包

```shell
emerge --unmerge 软件包名称
```

更新系统

```shell
emerge --update --ask world
emerge -ua world
```

更新整个系统

```shell
emerge --update --deep world
emerge -uD world
```

使用新的USE标记 重新构建系统

```shell
emerge --update --deep --newuse world 
emerge -uDN world
```

移除孤立依赖的软件包

```shell
emerge --update --deep --newuse world①
emerge --depclean②
revdep-rebuild③
```

① 重新构建系统

② 清除孤立依赖包

③ 重新构建依赖关系

revdep-rebuild工具由gentoolkit包提供；使用前别忘了首先emerge它：`emerge gentoolkit`



参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)