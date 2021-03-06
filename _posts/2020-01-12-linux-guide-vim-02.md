---
layout: post
title: "开源世界旅行手册：Vim编辑器（二）"
date: 2020-01-12 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Linux
---
## Vim编辑器（二）

### 搜索和替换

按下 / 键，编辑器底部会出现 / 符号，接着输入字符串，便可以进行搜索

| 按键 | 含义       |
| ---- | ---------- |
| /    | 向下搜索   |
| ?    | 向上搜索   |
| n    | 搜索下一个 |
| N    | 搜索上一个 |



| 输入                                | 含义                             |
| ----------------------------------- | -------------------------------- |
| :s/源字符串/目标字符串              | 将源字符串替换为目标字符串       |
| :s/源字符串/目标字符串/g            | 替换当前行中所有符合条件的字符串 |
| :行号1,行号2s/源字符串/目标字符串/g | 在指定行中进行替换               |
| :%s/源字符串/目标字符串/g           | 全文替换                         |

### 寄存器操作

Vim可以将不同字段剪切或复制到不同寄存器中，您可以从不同寄存器中取出内容后粘贴

```
"寄存器名称
```

> 按下"和另一个字符键，便可以定义一个寄存器，例如 "a "1

定义寄存器后直接进行操作

```
"ayy 将当前行复制到寄存器a中
"ap 将寄存器a中的内容粘贴到光标之后
```

- :registers查看所有寄存器的内容
- 通常情况下，寄存器+（先按"，再按Shift +=）对于X下的剪贴板，您在其他程序中复制的内容，可以使用 "+p 粘贴到Vim中，您在Vim中，可以使用 "+y 将内容复制到剪贴板，再粘贴到其他程序中
- 没有指定寄存器时，Vim使用“无名寄存器”存储内容

### 宏

您可以将一系列的操作录制为一个宏，然后执行它

```
q宏名称	开始录制宏，宏名称为一个字符
q	录制中按下“q”，结束录制
@宏名称	执行宏
```

> 可以使用 :registers（寄存器列表）命令查看已记录的宏

### 插入模式下的快捷键

| 快捷键            | 含义                 |
| ----------------- | -------------------- |
| Ctrl+r寄存器名称  | 插入指定寄存器内容   |
| Ctrl+k（2个字符） | 输入二合字符         |
| Ctrl+v数字        | 通过数字编码输入字符 |
| Ctrl+v键位        | 输入键位的名称       |

### 键绑定、缩写

前面我们已经向您介绍来键绑定

```shell
map! <M-j> <Down>
```

尖括号及其中的内容，为Vim配置文件的约定，分别描述了按键和功能，表示将功能编写到按键上。如果绑定的只是普通字符，例如：

```shell
map! xxx XXXXX
```

表示将XXXXX 绑定到xxx上。当您键入xxx时，编辑器会自动替换为XXXXX

如果您只是想将字符串绑定为缩写，方便输入，我们建议您使用iabbrev来绑定。例如

```shell
iabbrev g google
```

在插入模式下键入g，编辑器会自动替换为google。您可以将iabbrev命令缩写为iab，例如

```shell
iab g google
```

以上命令，您可以直接在命令模式下输入，临时启用。也可以写入配置文件，永久启用。

### 单词补全

```shell
<Ctrl+n>下一个匹配项
<Ctrl+p>上一个匹配项
```

您可以在匹配文件中定义补全的方式

```
"自动补全方式：（使用逗号分隔）
set complete=k,.

" . 当前文件
" b 已被装缓冲区，但是没有在窗口内的文件	
" d	在当前的文件中定义和由 #include 包含进来的文件
" i	由 #include 包含进来的文件
" k 由dectionary选项定义的文件
" kfile	名为{file}的文件
" t	标记(tags)的文件
"	u 没有载入的缓冲区
" w 在其他窗口中的文件

"设定自动补全字典：
```

### 命令模式

前面介绍了普通模式和插入模式。我们发现，普通模式主要用来浏览和修改文本内容，而插入模式则用来向文本中添加内容。

而命令模式则多用于操作文本文件（而不是操作文本文件的内容），例如保存文件；或者用来更改编辑器本身的状态，例如设定多栏窗口、标签或者退出编辑器......

w(rote)将更改写入文件

```shell
:w
```

q(uit)退出编辑器

```shell
:q
```

某些情况下，编辑器会阻止命令的执行。例如您修改了文件，而没有保存，那么您使用:q命令退出时，编辑器就不会执行这条命令，而是提醒您保存文件。

这个时候，您可以在命令末尾追加！来强制执行命令

```shell
:命令!
```

> 例如:q!，即便您没有保存已修改的文件，使用此命令，编辑器也会放弃修改而强行退出

以:引导一个Shell命令，则可以从Vim临时切换到Shell中，执行一个Shell

```shell
:!命令
```

> 例如 :!ls

命令模式还可以用来调节一些选项，例如

```shell
:set linespace=6
```

- 使用Tab键补全命令
- 命令可以直接写入配置文件
- 在选项后加上一个&使用默认值，例如： :set linespace&

### 折叠

| 命令               | 含义                         |
| ------------------ | ---------------------------- |
| zf数字             | 创建折叠，数字参数为折叠行数 |
| zo                 | 打开折叠                     |
| zc                 | 关闭折叠                     |
| zd                 | 删除折叠                     |
| :set foldcolumn=4  | 显示折叠树                   |
| :mkview            | 保存折叠                     |
| :loadview          | 读取折叠                     |
| zr                 | 打开同一层级所有折叠         |
| zm                 | 关闭同一层级所有折叠         |
| zO                 | 打开某一行的所有层级折叠     |
| zC                 | 关闭某一行的所有层级折叠     |
| zR                 | 打开所有折叠                 |
| zM                 | 关闭所有折叠                 |
| :set foldclose=all | 光标离开折叠自动关闭         |
| :set foldopen=all  | 关闭遇到折叠自动打开         |

您可以在配置文件中定义折叠的方式

```
"设定折叠方式
set foldmethod=manual

" manual 手动折叠
" indent 按缩进折叠
" marker 按标记折叠
" syntax 按语法折叠
" expr 按表达式折叠
```

按标记折叠的例子：

```shell
#起始标记 ( 将标记放到注释里，不会影响文件的功能

起始标记和结束标记之间的内容折叠
显示为起始标记所在行
#结束标记 )
##### 将折叠方式写到模式行里 #####
# vim: foldmethod=marker
```

### 多栏窗口

您可以使用以下命令，将当前窗口水平分为两栏

```shell
:new
```

新建一栏空白窗口，将当前文件分两栏显示

```shell
:split
```

同理，您可以使用下列命令，将当前窗口垂直分为两栏

```shell
:vnew
:vsplit
```

先按下ctrl+w键，再按下方向键 j、k、l、h，您可以切换到其他栏；在当前栏中使用 :q 命令，可以退出当前栏，也可以使用其他命令，对当前栏作处修改

如果您希望当前命令在所有栏中生效，您可以在命令的末尾追加all

```she
:命令all
```

> 例如：:qall

如果您希望这条命令强制执行，那么!位于命令的最末

> 例如：:qall!  强行退出所有栏窗口

### 标签页

Vim在7以后的版本，开始支持标签页的功能

```shell
:tabnew 新建一个标签
:tabnext 转到下一个标签
:tabprevious 转到上一个标签
```

您也可以使用鼠标点击标签进行切换

### 文件管理

使用Vim编辑目录时，会进入文件管理模式：

```shell
vim .
```

文件管理模式中可以实现一些简单的文件管理功能：

| 按键   | 含义                        |
| ------ | --------------------------- |
| enter  | 打开文件或文件夹            |
| Ctrl+o | 后退                        |
| o      | 新开一栏进行编辑            |
| O      | 在前一次打开的栏中编辑      |
| p      | 预览                        |
| i      | 显示/隐藏文件大小、日期     |
| s      | 切换文件排序方式            |
| r      | 切换顺序、逆序              |
| -      | 回到上一级目录              |
| c      | 将Vim执行目录设定为当前目录 |
| R      | 重命名                      |
| D      | 删除                        |
| x      | 运行                        |

> 其中，:cd 设定所有窗口执行目录； :lcd 设定当前栏窗口执行目录

### 加密

使用 :X 命令为文件设定一个密码

```shell
:X
Enter encryption key: ******
Enter same key again: ******
```

> 如果要取消加密，可以设置密码为空

可以禁用交换文件，以免泄密

```shell
vim -x -n file.txt
```

如果你已经编辑这个文件了，那么交换文件swapfile可以用下面的命令禁止：

```shell
:setlocal noswagfile
```

由于没了交换文件，文件复原就不可能了。为了避免失去编辑的成成果，要比平时更勤快地存盘你的文件

现在你可以像平时一样编辑这个文件并把你所有的秘密放进去。当你编完文件要退出Vim时，这个文件就被加密存盘了

当你下次用Vim编辑这个文件时，它就会询问你密码

如果你试图用另一个程序来阅读这个文件，你将读到一堆垃圾。如果你用Vim来编辑这个文件，但输入了错误的密码，你也只能得到垃圾。Vim并不具备检验密码正确性的机制（这一点使得破译密码更为困难）

### 版本

在Windows平台下，请使用gvim7ole版本



参考资料：

- [开源世界旅行手册](https://i.linuxtoy.org/docs/guide/)
