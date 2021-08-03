---
layout: post
title: "开源世界旅行手册：国际化"
date:  2020-01-07 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## 国际化

### I18N简介

习惯上，internationalization（国际化）简写为 I18N，中间的数字为省略的字母个数。

在Linux系统中，通过定义一组环境变量来设置程序的语言环境，以实现对I18N的支持，这种机制称为：locale（本地化）

#### locale变量

| 变量名            | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| LANG              | 低优先级全局locale变量，如果下面的变量未赋值，默认使用此变量的值 |
| LC_COLLATE        | 比较和排序习惯，会影响目录列表的分类显示等                   |
| LC_CTYPE          | 定义系统等字符处理特性，哪些字符能被视为字母、数字，等等；与中文输入关系密切 |
| LC_MESSAGES       | 提示信息，错误信息，状态信息，标题，标签，按钮和菜单等       |
| LC_MONETARY       | 定义货币单位和货币型数值的格式                               |
| LC_NUMERIC        | 定义非货币型数值的格式，影响到千位分隔符和小数分隔符等       |
| LC_TIME           | 定义日期和时间的格式                                         |
| LC_NAME           | 姓名书写方式                                                 |
| LC_ADDRESS        | 地址书写方式                                                 |
| LC_TELEPHONE      | 电话号码书写方式                                             |
| LC_MEASUREMENT    | 度量衡表达方式                                               |
| LC_PAPER          | 定义默认的纸张尺寸                                           |
| LC_IDENTIFICATION | 对locale自身包含信息的概述                                   |
| LC_ALL            | 高优先级全局locale变量，为此变量赋值会强行覆盖上面变量的值，不推荐 |

假如未设置以上变量，系统将采用POSIX作为locale，也就是C locale

#### locale 值

locale变量的值有三个要素：语言代码（Language Code)、地域代码（Country Code）和字符集（Encoding）

`语言代码[_地域代码[.字符集]]`

例如：

| 语言/国家代码    | 描述     |
| ---------------- | -------- |
| en_US.ISO-8859-1 | 美国英语 |
| en_US.UTF-8      |          |
| zh_CN.UTF-8      | 简体中文 |
| zh_TW.UTF-8      | 繁体中文 |

#### 字符集

众所周知，计算机中的信息，是以数字形式表示的，字符也不例外。字符以数字编号的形式存储，使用时，根据这个编号，在字符集中找到相应的字符

字符集是字符在系统内的编码方式，也就是通常所说的内码（参见/usr/share/i18n/charmaps）不同的字符集有不同的编码方式。

例如“码”字，它的GB2312编码为426B；UTF-8编码为E7A081。如果错误的以GB2312编码来检索E7A081，将会产生类似“锘跨爜” 的乱码

只要系统中安装了中文字体，通过字符集的支持，便可以正常显示中文，而无需设置locale；locale可以使操作界面显示中文，并可以使用中文输入等。

#### 其他

另外还有一个本地化变量叫做LINGUAS。它会影响到基于gettext的程序；它还能决定某些特殊软件包的本地化，比如kde-i18n和openoffice。这个变量的值为一组以空格分隔的语言代码：

`LINGUAS="zh en"`

### 设置Locale

设置locale其实就是设置环境变量。在/etc/environment文件中设置全局环境变量：

`LANG="zh_CN.UTF-8"`

在~/.profile或~/.bashrc中设置用户环境变量

```shell
export LANG="zh_CN.UTF-8"
```

​	完成后可以使用locale命令检验

#### 生成locale

如果你设置了一个不可用的locale，请使用localedef生成该locale

```shell
localedef -c -i en_US -f ISO-8859-15 en_US.ISO-8859-15
```

可能你在系统中只要用到一个或者两个locale，你可以在/etc/locale.gen中指定所需的locale，添加locale到/etc/locale.gen:

```shell
en_GB ISO-8859-1 
en_GB.UTF-8 UTF-8
de_DE ISON-8859-1
de_DE@euro ISO-8859-15
```

下一步是执行locale-gen。它会生成/etc/locale.gen文件中指定的所有locale

### locale策略

#### 中文图形界面

由于Linux的控制台不能方便的显示中文，所以最使用的方案是“英文控制台+中文图形界面”

为了能够正常处理中文，需要使用locale-gen生成locale，在/etc/locale.gen文件中添加如下内容：

```shell
zh_CN.UTF-8 UTF-8
zh_CN.GB18030 GB18030
zh_CN.GBK GBK
zh_CN GB2312
```

然后在gdm启动菜单中选择中文，或者写入配置文件~/.dmrc

```shell
[Desktop]
Session=openbox
Language=zh_CN.UTF-8
```

#### 英文界面+中文输入

使用如下设定：

```shell
LANG="en_US.UTF-8"
LC_CTYPE="zh_CN.UTF-8"
```

#### 关于输入发的设置

在~/.profile或~/.bashrc中设置用户环境变量：

输入法配置.profile

```shell
export SMODIFIERS="@im=fcitx①"
export GTK_IM_MODULE=xim②
export QT_IM_MODULE=xim③
fcitx&
```

① 使用fcitx输入法

② GTK2程序输入法引擎。fcitx使用xim引擎

③ QT输入法引擎


参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)