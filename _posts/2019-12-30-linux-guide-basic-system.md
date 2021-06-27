---
layout: post
title: "开源世界旅行手册：基本系统"
date:  2019-12-30 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## 基本系统

### 目录结构

各种Linux发行版的目录结构可能不太一样，但它们都遵循FHS（Filesystem Hierarchy Stand）。

实际上FHS只是规定了根目录下的各主要目录应该放些什么文件，仅了解这些还不够，下面是一般情况下Linux系统的目录结构。

```shell
/ 根目录
|
|-boot/         启动文件。所有与系统启动相关的文件都保存在这里
| |-grub/       Grub引导器相关文件
|
|-dev/		设备文件
|-proc/		内核与进程镜像
|
|-mnt/		临时挂载
|-media/	挂载媒体设备
|
|-root/		root用户的$HOME目录
|-home/		
| |-user/	普通用户的$HOME目录
```

### 启动流程

1. 读取MBR的信息，启动Boot Manager①
2. 加载系统内核，启动init进程②
3. init进程 读取/etc/inittab文件中的信息，并进行预设的运行级别，按顺序运行该运行级别对应文件夹的脚本。脚本通常以start选项启动，并指向一个系统中的程序。③
4. 根据/etc/rcS.d/文件夹中对应的脚本启动Xwindow服务器xorg④
5. 启动登录管理器，等待用户登录⑤

①Windows使用NTLDR作为Boot Manager，如果您的系统中安装多个版本的Windows，您就需要在NTLDR中选择您要进入的系统。Linux通常使用功能强大，配置灵活的GRUB作为Boot Manager。

②init进程是Linux的根进程，所有的系统进程都是它的子进程

③通常情况下，/etc/rcS.d/目录下的启动脚本首先被执行，然后是/etc/rcN.d目录。例如您设定的运行级别为3，那么它对应的启动目录为/etc/rc3.d/。

④Xwindow为linux下图形用户界面系统

⑤大多Linux系统默认使用GDM作为登录管理器，您在登录管理器界面中输入用户名和密码后，便可以登录系统。（您可以在/etc/rc3.d/文件夹中找到一个名为S13gdm的链接）

#### 更改运行级别

在/etc/inittab文件中找到如下内容：

```shell
# The default runlevel.
id:2①:initdefault:
```

①2为系统的运行级别，默认的运行级别涵义如下：

| 运行级别 | 含义           |
| -------- | -------------- |
| 0        | 关机           |
| 1        | 单用户维护模式 |
| 2～5     | 多用户模式     |
| 6        | 重启           |

### 服务管理

#### 更改启动服务

在运行级别对应的文件夹中，您可以看到许多文件名以S##和K##起始的启动脚本链接。例如：

```shell
/etc/rcS.d/S35mountall.sh	挂载文件系统①
/etc/rcS.d/S40networking	启动网络支持
/etc/rc2.d/S13gdm		启动登录管理器
/etc/rc2.d/S20makedev		创建设备文件
/etc/rc2.d/S23xinetd		启动超级进程
/etc/rc2.d/K20powernowd		针对某种硬件的电源管理支持②
```

①init进程将以`start`为选项，按文件名顺序执行所有以S##起始的脚本。脚本名称中的数字越小，它将被越早执行。例如在/etc/rc2.d/文件中，S13gdm文件名中的数字小于S23xinetd，S13gdm将比S23xinetd先执行。

②如果一个脚本链接，以K##起始，表示它将以stop选项被执行。如果相应服务没有启动，则不执行该脚本。

如果您想禁用某一服务在启动时自动运行，您可以将相应运行级别中的脚本由S##xxx重命名为K##xxx。

#### 手动控制服务

您也可以手动运行带有一下选项的启动脚本，来控制系统服务。`start`启动`stop`停止`restart`重启。

```shell
/etc/rc2.d/K20powernowd start
```

有时您并不清楚某一运行级别有什么启动脚本；而且此类脚本的前三位字符并不固定，不便于记忆。这时，可以直接使用/etc/init.d/文件夹中的启动脚本（/etc/rcX.d/中的启动脚本链接到/etc/init.d/文件夹下相应脚本）这也是推荐的方式。

```shell
/etc/init.d/powernowd start
```

> 注意：以上命令的位置并没有包含在环境变量的搜索路径中，所以要输入完整路径。

#### 常见系统服务

| 服务                            | 解释                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| acpi-support                    | 高级电源管理支持                                             |
| acpid                           | acpi守护进程。这两个用于电源管理，非常重要                   |
| apmd                            | acpi的扩展                                                   |
| alsa                            | 声音子系统                                                   |
| alsa-utils                      | 声音子系统实用工具                                           |
| cron                            | 任务调度系统，建议开启                                       |
| anacron                         | cron的子系统，将系统关闭期间的计划任务，在下一次系统运行时执行 |
| atd                             | 类似于cron的任务调度系统。建议关闭                           |
| binfmt-support                  | 核心支持其他二进制的文件格式                                 |
| bluez-utiles                    | 蓝牙设备支持                                                 |
| bootlogd                        | 启动日志。开启它                                             |
| syslog-ng                       | 系统日志。建议开启                                           |
| klogd                           | 同上，使用一种就可以了                                       |
| sysklogd                        |                                                              |
| cupsys                          | 打印机子系统                                                 |
| dbus                            | 消息总线系统（message bus system）。非常重要                 |
| dns-clean                       | 使用拨号连接时，清除dns信息                                  |
| evms                            | 企业卷管理系统                                               |
| fetchmail                       | 邮件用户代理守护进程，用于收取邮件                           |
| gdm                             | gnome登录管理器                                              |
| gpm                             | 终端中的鼠标支持                                             |
| halt                            | 别动它                                                       |
| hdparm                          | 调整硬盘的脚本，配置文件为/etc/hdparm.conf                   |
| hibernate                       | 系统休眠                                                     |
| hotkey-setup                    | 笔记本功能键支持                                             |
| hotplug and hotplug-net         | 即插即用支持，比较复杂，建议不要动它                         |
| ifrename                        | 网络接口重命名脚本。如果您有十块网卡，您应该开启它           |
| inetd                           | 配置文件为/etc/inetd.conf                                    |
| linux-restricted-modules-common | 受限模块支持①                                                |
| lvm                             | 逻辑卷管理系统支持                                           |
| makedev                         | 创建设备文件，非常重要                                       |
| mdamd                           | 磁盘阵列                                                     |
| module-init-tools               | 从/etc/modules加载扩展模块，建议开启                         |
| networking                      | 网络支持，按/etc/network/interfaces文件预设激活网络，非常重要 |
| ntpdate                         | 时间同步服务，建议关闭                                       |
| pcmcia                          | pcmcia设备支持                                               |
| powernowd                       | 移动CPU节能支持                                              |
| ppp                             | 拨号连接                                                     |
| ppp-dns                         |                                                              |
| readahead                       | 预加载库文件                                                 |
| reboot                          | 别动它                                                       |
| resolvconf                      | 自动配置DNS                                                  |
| rmnologin                       | 清除nologin                                                  |
| rsync                           | rsync守护进程                                                |
| sendsigs                        | 在重启和关机期间发送信号                                     |
| single                          | 激活单用户模式                                               |
| sshd                            | ssh服务器                                                    |
| sudo                            | 检查sudo状态                                                 |
| udev                            | 用户控件dev文件系统(userspace dev filesystem)。重要          |
| umounttfs                       | 卸载文件系统                                                 |
| urandom                         | 随机数生成器                                                 |
| usplash                         | 开机画面支持                                                 |
| vbesave                         | 显卡BIOS配置工具。保存显卡的状态                             |
| xorg-common                     | 设置X服务ICE socket                                          |
| adjtimex                        | 调整核心时钟的工具                                           |
| dirmngr                         | 证书列表管理工具                                             |
| hwtools                         | irqs优化工具                                                 |
| libpam-devperm                  | 系统崩溃之后，用于修理设备文件许可的守护程序                 |
| lm-sensors                      | 板载传感器支持                                               |
| mdadm-raid                      | 磁盘陈列管理器                                               |
| screen-cleanup                  | 清除开机屏幕的脚本                                           |
| xinetd                          | 管理其他守护进程的一个inetd超级守护程序                      |

①/lib/linux-restricted-modules/文件夹中的模块为受限模块。例如某些驱动程序，如果您没有使用受限模块，就不需要开启它。

### 配置文件

> 无论何种情况，修改配置文件之前，先备份它

建议使用这个命令

```shell
sudo cp xxx xxx_`date + %y%m%d_%H:%M`
```

当然这很麻烦，您可以新建一个名为bak的文件，内容如下

```shell
#!/bin/bash
sudo cp $1 $1_`date + %y%m%d_%H:%M`
```

把它放在您能够记住的目录下，比如/home，执行命令sh /home/bak xxx, 就可以将当前文件夹下的文件xxx另存为xxx_yymmdd_HH:MM的格式了。

#### 全局配置文件

![image-20210627205333114](/img/in-post/2019-12-30-linux-guide-basic-system-01.png)

#### 用户配置文件

/etc/目录下的文件，只有root用户才有权修改。应用软件的全局配置文件，普通用户也不应该修改，因为所有用户都要用到。

如果要通过配置软件，来适应特殊需求，您可以修改用户配置文件。

用户配置文件通常为全局配置文件的同名隐藏文件，放在$HOME目录下，例如：

| 全局配置文件     | 用户配置文件          |
| ---------------- | --------------------- |
| /etc/inputrc     | /home/user/.inputrc   |
| /etc/vim/vimrc   | /home/user/.vim/vimrc |
| /etc/bash.bashrc | /home/user/.bashrc①   |

①也有少数例外。通常是系统程序。

### 环境变量

环境变量是作用在整个系统中的变量。

很多软件工作的时候都要读取环境变量（必须由大量不同程序共享，独立于用户程序的优先选项，通常由环境变量指定，如果通过配置文件给每个程序分别指定，既麻烦又容易出差错。例如语言环境，如果能够使用多语言的程序都使用自己的配置文件确定其工作界面的语言，逐一去修改它们的配置文件将是一件苦恼的事情）的值来确定其工作方式。例如cd这个程序，如果不带任何选项执行，它会读取$HOME这个变量的值，然后进入到这个目录，也就是用户的HOME目录。使用`echo`显示一个环境变量：

```shell
echo $HOME ①
/home/user

echo HOME②
HOME
```

①依据惯例，变量名称通常使用大写字母：$告诉echo这是一个变量，不要将字符串“HOME”直接输出到屏幕

②输出字符串"HOME"到屏幕

```shell
WELCOME="Hello!"①
echo $WELCOME②
Hello!
env③ | grep WELCOME④
export WELCOME⑤
env | grep WELCOME
Hello!
```

①定义一个变量

②输出这个变量

③env输出所有环境变量

④将输出通过管道发送到grep，检查其中是否有“WELCOME”这个值

⑤export将变量WELCOME导出为环境变量

定义一个环境变量，可以将定义变量和导出变量一起进行

```shell
export WELCOME="Hello!"
```

①实际上不会有什么程序要用到$WELCOME这个变量，定义它只是为了演示。

#### 常用环境变量

| 变量名         | 说明       |
| -------------- | ---------- |
| SHELL          | 默认shell  |
| HISTSIZE       | 历史记录   |
| PATH           | 搜索路径   |
| EDITOR         | 默认编辑器 |
| TERM           | 默认终端   |
| HOME           | 用户目录   |
| LANG           | 系统语言   |
| XMODIFIERS=@im | 默认输入法 |


参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)