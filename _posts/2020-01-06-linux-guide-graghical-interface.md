---
layout: post
title: "开源世界旅行手册：图形界面"
date:  2020-01-06 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## 图形界面

### 简介

Xwindow是工作站图形系统的工业标准，它有多种不同的实现，多数Linux系统中使用Xorg。

当然，Xwindow有悠久的历史和传统，不过那不在我们讨论的范围。您要注意的有两点。

- Xwindow和Xbox中的X意义是不同的，X只是W之后的一个字母，差不多应该这样理解，Xwindow是Window的接班人（注意，Window不是Windows）
- 同样，也不要吧Xwindow说成是Xwindows，那是一种亵渎！一切伟大的创造，都应得到应有的理解和尊重。

### 架构及原理

Xwindow使用服务器-客户端架构。无论本地图形界面还是远程图形界面，都以同样的流程工作。这样便不需要分别进行设计和维护。

```shell
本地X客户端┐                         ┌键盘
远程X客户端┼ X协议 - X服务器 - 驱动程序 ┼鼠标
远程X客户端┘                         └显示器 
```

#### Xserver

Xwindow系统服务器端，通过驱动程序（硬件规范）来管理硬件资源。

例如：当我们移动鼠标时，通过驱动程序，向Xserver发送信息：

​	“向右移动200点，向上移动100点”（向右上移动）；“按下左键”......

Xserver作出如下响应：

1. 上一次鼠标停止的坐标为600，500
2. 向右200，向上100。现在鼠标位于坐标800，600
3. 坐标800，600处，为窗口Firefox的“关闭”按钮
4. 根据预设动作，将“点击Firefox窗口的关闭按钮”翻译为“关闭窗口Firefox”
5. 向X客户端Firefox发送一个“退出”消息
6. Xserver通过显示子系统（显卡、显示器），全程显示鼠标的位置和移动

事实上，向程序发送“退出”信号，通常窗口管理器完成。为了描述方便，这里暂不区分。

#### Xclient

Xwindow系统客户端，通过X协议，实现与Xserver的交互。例如：

1. Xclient（假设Firefox）接收Xserver的消息：输入焦点在地址栏的范围内，“linuxtoy.org”，回车
2. Firefox根据预设动作，将这些消息识别为“打开链接linuxtoy.org”
3. Firefox向域名服务器请求链接“linuxtoy.org”。域名服务器将这个请求转换为“http://linuxtoy/org/”和IP地址211.148.131.7，发送回Firefox
4. Firefox将"http://linuxtoy.org/"显示在地址栏（向Xserver发送请求，在地址栏位置显示这个位置）
5. Firefox向地址211.148.131.7请示显示页面
6. Firefox将服务器发送回的页面显示在主窗口中

#### Xprotocal

Xwindow系统协议，Xserver和Xclient之间进行通信的规则。

### 窗口管理器

Window Manager是一种特殊的Xclient

使用窗口管理器时，Xserver并不直接与其他Xclient通信，而是通过WM中转，当一些消息被定义为WM指令时，它们会被拦截。例如Alt+F4关闭窗口、拖动标题栏......

消息“打开链接linuxtoy.org”，具体内容如下：

​	输入焦点在地址栏的范围内，“linuxtoy.org”，回车

Xserver并不能直接判断焦点，而是这样：

1. Xserver向WM发送位置和点击的信息，WM根据当前的“焦点策略”确定激活（最上层）的窗口为Firefox
2. Xserver将Firefox显示在最上层，高亮显示它的标题栏
3. 在窗口Firefox内点击地址栏，或者Ctrl+L，Xserver将位置信息发送给WM，WM再发送给Firefox
4. Firefox判断当前焦点后，显示一个闪动的文字输入光标
5. Firefox将输入光标通过WM发送给Xserver，Xserver在屏幕相应位置进行显示

那么，“窗口管理器”到底能做些什么呢？其实它所做的只有一件事——管理窗口。例如：

- 最上层的窗口会把其他窗口挡住
- 它通常是一个“已激活窗口”，根据不同的“焦点策略”，窗口管理器确定被激活的窗口。激活窗口标题栏高亮显示，接收大部分的 键盘消息和窗口内的鼠标点击信息
- 为了美观和容易分辨，大多数窗口都要有标题栏和边框。为了方便，标题栏上还要有一些按钮，比如：最小化，最大化，关闭（这些按钮是窗口管理器请求的小窗口）
- 一个窗口可以在另一个窗口旁边显示，而不一定完全被遮挡。为了实现这一点，就要控制窗口显示的位置
- 为了控制窗口的显示位置，需要将整个屏幕用坐标描述，最好的办法是绘制一个填充整个屏幕的窗口，也就是根窗口。
- 因为根窗口是最大的，所以它可以严严实实的遮挡任何窗口，为了避免这一点，根窗口永远在最底层。这很形象的说明了为什么它叫作“根窗口”
- 根窗口不一定只有一个，大多数的窗口管理器可以使用“工作区”，来切换显示多个根窗口
- 根窗口固定位置上通常放置一些其他Xclient的窗口，例如底部面板，顶部面板，侧面板，程序启动图标
- 面板上又可以放一些其他的Xclient窗口，如任务条，启动栏，菜单......任务条可以以图表显示正在运行的任务，还可以作其他的杂活，像自动挂载USB设备......

### 启动流程

我们知道init是linux的根进程，是所有进程的父进程。同样，xinit是所有Xwindow进程的根进程。

#### Startx

startx可以在命令行下启动图形界面。执行startx命令时，实际执行这一命令：

```shell
xinit /etc/X11/xinit/xinitrc -- /etc/X11/xinit/xserverrc
```

根据脚本/etc/X11/xinit/xserverrc 启动Xserver，同时根据脚本/etc/X11/xinit/xinitrc 启动指定Xclient进程，例如窗口管理器

/etc/X11/xinit/xserverrc 以预设的参数运行程序 /usr/bin/X11/X

/etc/X11/xinit/xinitrc 脚本则指向/etc/X11/Xserssion，依次启动/etc/X11/Xsession.d目录中的脚本

- 您可以在在用户配置文件～/.Xsession中定义使用WM，它的优先级高于全局配置文件（对于GDM会话不起作用）
- startx启动时，并不会再进行身份认证。因为它启动的是/etc/X11/Xsession.d/gnome-session，而不是GDM会话

#### GDM会话

许多Linux系统启动时自动进入图形界面，不需要运行startx命令

在某些启动级别中，包含了gdm的启动脚本，例如：/etc/rc2.d/S13gdm

1. 指向/etc/gdm/gdm-cdd.conf文件，加载预设视觉主题，启动/usr/lib/gdm/gdmgreeter（登录屏幕）
2. 用户身份认证完成后，启动/etc/X11/default-display-manager这个文件中设定的默认窗口管理器/usr/sbin/gdm

gdm在启动时，会要求用户名和密码，也就是我们看到的登录屏幕（gdmgreeter）

/usr/share/xsessions目录下为所有可用登录会话的脚本

### 配置文件

#### X服务器

X服务器的主要配置文件为/etc/X11/xorg.conf

```shell
Section "ServerLayout" #布局
Identifier "Xorg Configured" #布局标识
Screen 0 "Default Screen" 0 0 #屏幕标识
InputDevice "Generic Keyboard" #键盘标识
InputDevice "Configured Mouse" #鼠标标识
EndSection

Section "Module"
	Load "ddc"
	Load "dbe"
	Load "dri"
	Load "extmod"
	Load "glx"
```

默认分辨率为默认色深下的第一个分辨率

#### 配置文件内部结构

```shell
/
├/ "ServerLayout" 布局
|├ "InputDevice" keyboard 键盘
|├ "InputDevice" mouse 鼠标
||
|└/ "Screen" 显示子系统
│ ├ "Monitor" 显示器
│ ├ "Device" videocard 显卡
|
|
├ "Files" 字体
└ "Module" 模块
```

#### X客户端

在/etc/X11/Xsession文件中可以发现下列内容

```shell
OPTIONFILE=/etc/X11/Xsession.options①
②
SYSRESOURCES=/etc/X11/Xresources
USRRESOURCES=$HOME/.Xresources
③
SYSSESSIONDIR=/etc/X11/Xsession.d
USERXSESSION=$HOME/.xsession
ALTUSERXSESSION=$HOME/.Xsession
ERRFILE=$HOME/.xsession-errors
```

① 设定X进程的启动参数。例如运行用户进程allow-user-xsession

② X资源文件。许多程序保留了X接口，允许X服务器管理一些视觉选项，例如窗口内的字体，配色等

③ X进程。可以设置一些启动时自动运行的程序，也可以用来设定自己的窗口管理器（窗口管理器和桌面环境或者登录管理器是无关的）

### 字体

#### freetype 渲染引擎

作为Xorg服务器的一个模块，freetype的功能包括读取Truetype字体信息，如大小、分辨率、编码等，并以之为依据渲染字体-freetype2.x相对于freetype1.x增加了抗锯齿等功能-(/etc/X11/xorg.conf 的Module字段中，可以选择字体渲染模块)

freetype只负责渲染字体。而查找字体，则可以由X服务器、X客户端或者字体服务器来完成。找到字体后，使用freetype引擎进行渲染

**X核心字体**X服务器根据X客户端的请求（字符编码），查找字体并进行渲染，然后显示

**Xft字体**X客户端自行查找字体并进行渲染，X服务器只负责显示。由于Xft字体的渲染在客户端完成，所以它可以动态的加载，而不需要随同X服务器一同启动

**字体服务器**当客户端请求字体时，X服务器将请求转发到字体服务器，由字体服务器查找字体，并使用freetype引擎渲染，将结果传回X服务器，X服务器进行显示

#### X核心字体

/etc/X11/xorg.conf中可以配置X核心字体的搜索路径

```shell
Section "Files"
FontPath "/usr/X11R6/lib/X11/fonts/misc/"
FontPath "/usr/X11R6/lib/X11/fonts/Type1/"
FontPath "/usr/X11R6/lib/X11/fonts/Speedo/"
FontPath "/usr/X11R6/lib/X11/fonts/100dpi/"
FontPath "/usr/X11R6/lib/X11/fonts/75dpi/"
EndSection
```

- 当X客户端向X服务器请求显示文字的时候，X服务器会按上面列表的先后顺序查找字体。显示中文时，如果第一个路径中的字体不包含中文，则查找下面的路径，直到发现中文字体
- 请将您便好的字体放在靠前的位置

要使安装的字体能够作为X核心字体使用，将字体的安装路径添加到上面的列表中，使用mkfontscale、mkfontdir扫描文件夹中的字体，并生成索引，就可以了（建议使用ttmkfdir生成fonts.scale，将其复制为fonts.dir）

字体的选择及显示风格，可以修改GTK1的配置文件，或者在Xresources文件中对程序单独进行定义

事实上，在我们的日常应用中，X核心字体环境并不常见，使用GTK1图形库的程序、某些类型的终端......

**XFT字体**

Xft字体相关选项在/etc/fonts/fonts.conf文件中配置

```shell
<dir>/usr/share/X11/fonts</dir>
<dir>/usr/share/fonts</dir>
<dir>/usr/local/share/fonts</dir>
<dir>~/.fonts</dir>
```

可以使用fc-cache命令，递归扫描上面目录中的字体（包括子文件夹中的字体），建立字体缓存

多数支持GTK2或者Qt图形库的X客户端能够使用Xft字体渲染技术

安装字体，只要将字体拷贝到以上任意目录，fc-cache -fv刷新字体缓存即可（选项：-f强制刷新；-v显示过程）

使用命令fc-list列出所有可用字体。字体的选择及显示风格，可以修改GTK2或者Qt的配置文件，建议使用图形界面配置


参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)