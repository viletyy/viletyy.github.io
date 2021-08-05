---
layout: post
title: "开源世界旅行手册：Grub"
date: 2020-01-09 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---

## Grub

### 硬件基础

一块硬盘，它起始的一部分扇区为主引导扇区，包括 MBR（主引导记录）和 DPT（分区表）

每个分区起始的一部分扇区为分区引导扇区。

在分区引导扇区之后的部分，为文件系统的索引，文件系统通过它定位文件在硬盘上的位置。不同的文件系统采用不同的索引，例如 FAT 文件系统使用文件分配表和目录区。

绝大多数操作系统，对硬盘的读写操作，通过文件系统来完成，因此引导扇区中的内容，我们不能够在文件系统中进行操作，而需要专用软件，比如引导管理器

我们对文件进行修改后，操作系统会将文件系统索引中的内容同步

#### 系统引导流程

1. 系统启动时，首先引导至 MBR，将控制权移交安装在 MBR 中的引导管理器（Windows 使用 NTLDR，Linux 通常用 Grub）
2. 引导管理器装载自身（Grub 先装载 MBR 中的 stage1，通过 stage1 来装载文件系统中的 stage2，显示菜单和 shell 等待用户。有时 stage1 不能识别 stage2 所在分区的文件系统，这就需要装载 stage1.5 来连接 stage1 和 stage2）
3. 引导管理器读取分区中的配置文件，并按配置文件中预设的参数运行（例如：Grub 读取`/boot/grub/menu.list`文件中内容，将可引导系统通过菜单显示）
4. 引导管理器根据您的选择，可能会有如下活动
   - 加载内核，启动 Linux 系统检查活动分区，并引导它
   - Windows 系统：读取相应分区的引导扇区，将控制权移交该扇区中的引导管理器

Linux 系统在安装 Grub 时，会提示您安装在 MBR 或者分区引导扇区中，如果将 Grub 安装在分区引导记录中，您必须确保 MBR 中引导管理器能够正确的引导至分区引导扇区，如果您在 MBR 中使用的是 Windows 的引导管理器 NTLDR，完成这件工作会非常困难，因此我们推荐您使用 Grub。

### Grub 介绍

Grub 主要有以下功能：

- 菜单式选择
- 命令行模式
- 支持开机画面
- 支持大硬盘

您可以运行命令`grub`启动它，会显示一些版本信息和使用提示，当然还有命令提示符，如下

```shell
GNU GRUB version 0.97 (640K lower / 3072K upper memory)

	[Minimal BASH-like line editing is supported. For
	the first word, TAB lists possible command completions.
	Anywhere else TAB lists the possible completions of
	a device/filename.]

grub >
```

您可以使用`TAB`键补全命令和路径，这非常重要，因为 Grub 中路径表示方式与操作系统是不同的，您可能比较陌生，所以尽量用 TAB 补全它，既方便，也不容易出错

您可以在 grub>提示符后按`TAB`键，会将所有可用的命令显示出来。

呵呵，是不是有点晕，命令可真不少啊！！！其实我们会用到的命令只有两个:**root** **setup**

### Grub 术语

在分区概念章节里，Linux 系统中表示分区的方法` /dev/sta5`

```shell
/dev/sdMN M为a起始的小写字母，表示硬盘序号；N为1起始的数字，表示分区序号
```

Grub 中使用的表示方法为 `hd0,1`

```shell
hdX,Y	X为0起始的数字，表示硬盘序号；Y为0起始的数字，表示分区序号
```

留意它们的区别：

- N 从 1 开始计数，X 和 Y 从 0 开始计数
- N 为 1 ～ 4，它是一个主分区；N 为 5 或者大于 5，它是第（N-4）个逻辑分区，Y 按分区在硬盘上排列的顺序排列，无论它表示的是主分区还是逻辑分区

举例来说：

| 分区分布  | 主分区 | 主分区 | 逻辑分区 | 逻辑分区 | 主分区 |
| --------- | ------ | ------ | -------- | -------- | ------ |
| /dev/sdMN | sda1   | sda2   | sda5     | sda6     | sda3   |
| hdX,Y     | hd0,0  | hd0,1  | hd0,2    | hd0,3    | hd0,4  |

现在我们来看 root 和 setup 命令的使用，sudo grub 进入 Grub 交互模式：

```shell
grub>root(hd0,1)①
grub>setup(hd0)②
```

① 这个命令将 Grub 的根分区定位为（hd0,1）

② 这个命令表示将 Grub 安装在（hd0）因为没有指定安装的分区，所以安装位置为 MBR

#### Grub 的根分区

为 Grub 配置文件 grub/menu.lst 和 stage 文件所在分区，假如您单独为/boot 目录挂载了一个分区，那么 Grub 的根分区通常为您系统中/boot 目录所在的分区。

搞错了根分区，Grub 就不能正确读取配置文件，自然不能正确引导。

#### 系统根目录所在分区

Linux 根目录/的挂载分区。Linux 系统的分区挂载信息保存在文件系统分配表/etc/fstab 文件中

Grub 首先读取根分区中/boot/grub/menu.lst 文件，并转到引导分区，如果是 Windows 等系统，则将控制权移动分区引导扇区中的启动管理器。如果是 Linux 系统，则加载内核和设备，并根据/etc/fstab 文件的内容挂载文件系统。

看这个例子：（假设 Grub 安装在 MBR 中，Grub 的安装位置为（hd0））

```shell
title 	Linux
root    (hd0,0)①
kernel	(hd0,1)②/vmlinuz-2.6.15-25-686 root=/dev/sda3③ ro splash vga=0x31b
initrd	(hd0,1)④/initrd.img-2.6.15-25-686
boot
```

① 这一行表示引导分区为第一块硬盘的第一个分区（hd0,0）如果下面也指定了这个参数，那么下面的优先

② 表示当前系统的/boot 目录挂载到第一块硬盘的第二个分区（hd0,1）它是引导分区。一般情况下在“root”项中指定

③ 表示当前系统的/目录挂载到第一块硬盘的第三个分区（hd0,2），内核根据该分区中的/etc/fstab 文件来挂载文件系统。

④ 同 2

### Grub 配置文件

/boot/grub/menu.lst 文件，主要由一些下面这样的块构成的。

```shell
timeout 	5①
default 	0②

title 		Linux③
root            (hd0,2)④
kernel          (hd0,2)/boot/vmlinuz-2.6.15-25-686 root=/dev/sda3⑤ ro⑥ splash⑦ vga=0x31b⑧
initrd		(hd0,2)/boot/initrd.img-2.6.15-25-686
boot


title 		Windows
root            (hd0,0)
makeactive⑨
chainloader +1 ⑩
```

① 等待时间，如果用户在这段时间内未进行选择，则引导默认系统

② 默认系统，从 0 开始计数，每个 title 项为一个系统

③ 标题，title 和分隔符后的内容为 Grub 菜单中显示的条目

④ 引导分区

⑤ 表示当前系统的/目录挂载到第一块硬盘的第三个分区(hd0,2)，内核根据该分区中的/etc/fstab 文件来挂载文件系统

⑥ 以只读模式挂载

⑦ 显示启动画面

⑧ 启动屏幕分辨率

⑨ 设置活动分区

⑩ 链式引导

### Grub 安装

#### 将 Grub 安装到系统中

```shell
grub-install --no-floppy① --root-directory=/boot② /dev/sdM③
```

① 不使用软盘

② 文件安装目录，通常不需要指定

③ 目标磁盘

这一步只是拷贝 Grub 到文件。如果是修复系统，通常情况下它们已经存在，直接进行下一步就可以

#### 将 Grub 安装到 MBR

`sudo grub`进入 Grub 交互模式：

```shell
grub> find /boot/grub/stage2①
	(hd0,0)

grub> root (hd0,0)③
	Filesystem type is xfs, partition type 0x83④

grub> setup (hd0)⑤
	Checking if "/boot/grub/stage1" exists... yes
	Checking if "/boot/grub/stage2" exists... yes
	Checking if "/boot/grub/xfs_stage1_5" exists... yes⑥
	Running "embed /boot/grub/xfs_stage1_5 (hd0)"... 19 sectors are embeded.⑦succeeded
	Running "install /boot/grub/stage1 (hd0) (hd0)1+19 p (hd0,0)/boot/grub/stage2/boot/grub/menu.lst"... succeeded⑧
```

① 搜索 stage2 文件所在的分区，通常就是 Grub 根分区

② 返回 Grub 根分区名称

③ 将这个分区设为 Grub 根分区

④ 返回 Grub 根分区的文件系统信息

⑤ 将 Grub 安装到 MBR。注意是(hd0)而不是(hd0,0)

⑥ 检查需要的文件是否存在，因为用了 XFS 文件系统，所以需要 xfs_stage1_5

⑦ 将 xfs_stage1_5 嵌入到 MBR，不然 Grub 不能够读取 XFS 分区中的 stage2

⑧ 将 stage1 安装到 MBR，并指向 stage2 和 menu.list

⑨ 退出

> 可以使用**`TAB`**补全，或者按两次**`TAB`**列出提示

### Grub 使用

在 Grub 启动菜单中，您可以选择您要的选项，按下 d 删除该项；或者按下 e 键，进入到编辑模式

修改您的启动参数，完成后回车

按 b 键，Grub 将以您修改后的参数引导系统。

在 Grub 启动菜单中，按下 c 进入命令模式，操作与交互模式基本相同；只不过最后一个命令是 boot，而不是 quit


参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)
