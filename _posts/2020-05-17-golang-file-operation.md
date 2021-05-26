---
layout: post
title: "Golang：File文件操作"
date:  2020-05-18 10:04:53
author: "Viletyy"
header-style: text
tags:
  - Golang
---

### File文件操作
#### FileInfo接口
> FileInfo接口中定义了File信息相关的方法。

```go
type FileInfo interface {
  Name() string  				// base name of the file 文件名.扩展名 aa.txt 
  Size() int64  				// 文件大小， 字节数 12540
  Mode() FileMode 				// 文件权限 -rw-rw-rw-
  ModTime() time.TIme 		    // 修改时间 2018-04-13 16:30:53 +0800 CST
  IsDir() bool 					// 是否文件夹
  Sys() interface{}				// 基础数据源接口(can return nil)
}
```

#### 权限
> 操作权限perm，除非创建文件时才需要指定，不需要创建新文件时可以将其设定为0.
> 虽然go语言给perm权限设定了很多的常量，但是习惯上也可以直接使用数字，如0666（具体含义和Unix系统的一致）

#### 3. 打开模式
```go
const (
  O_RDONLY int = syscall.O_RDONLY // 只读模式打开文件
  O_WRONLY int = syscall.O_WRONLY // 只写模式打开文件
  O_RDWR int  = syscall.O_RDWR    // 读写模式打开文件
  O_APPEND int = syscall.O_APPEND // 写操作时将数据附加到文件尾部
  O_CREATE int = syscall.O_CREAT  // 如果不存在将创建一个新文件
  O_EXCL int  = syscall.O_EXCL    // 和O_CREATE配合使用，文件必须不存在
  O_SYNC int  = syscall.O_SYNC    // 打开文件用于同步I/O
  O_TRUNC int = syscall.O_TRUNC  // 如果可能，打开时清空文件
)
```

#### File操作

```go 
type File 
// File代表一个打开的文件对象

func Create(name string) (file *File, err error)
// Create采用模式0666 (任何人都可读写，不可执行）创建一个名为name的文件，如果文件已存在会截断它（为空文件）。如果成功，返回的文件对象可用于I/O；对于的文件描述符具有O_RDWR模式。如果出错，错误底层类型是*PathError.

func Open(name string) (file *File, err error) 
// Open打开一个文件用于读取。如果操作成功，返回的文件对象的方法可用于读取数据。对应的文件描述符具有O_RDONLY模式。如果出错，错误底层类型是*PathError

func OpenFile(name string, flag int, perm FileMode) (file *File, err error) 
// OpenFile是一个更一般的文件打开函数，大多数调用者都应用Open或Create代替本函数。它会使用指定的选项（如O_RDONLY等）、指定的模式（如0666等）打开指定名称的文件。如果操作成功，返回的文件对象可用于I/O。如果出错，错误底层类型是*PathError

func NewFile(fd uintptr, name string) *File 
//NewFile使用给出的Unix文件描述符和名称创建一个文件

func Pipe() (r *File, w *File, err error) 
// Pipe返回一对关联的文件对象。从r的读取将返回写入w的数据。本函数会返回两个文件对象和可能的错误

func (f *File) Stat() (fi FileInfo, err error) 
// Stat返回描述文件f的FileInfo类型值。如果出错，错误底层类型是*PathError.

func (f *File) Fd() uintptr 
// Fd返回与文件f对应的证书类型的Unix文件描述符

func (f *File) Chdir() error 
// Chdir将当前工作目录改为f， f必须是一个目录。如果出错，错误底层类型是*PathError 

func (f *File) Chmod(mode FileMode) error 
// Chmod修改文件的模式，如果出错,错误底层类型是*PathError

func (f *File) Chown(uid, gid int) error 
// Chown修改文件的用户id和组id， 如果出错，错误底层类型是*PathError 

func (f *File) Close() error 
// Close关闭文件f，使文件不能用于读写。它返回可能出现的错误

func (f *File) Readdir(n int) (fi []FileInfo, err error)
// Readdir读取目录f的内容，返回一个有n个成员[]FileInfo，这些FileInfo是被Lstat返回的，采用目录顺序。对本函数的下一次调用会返回上一次调用剩余未读取的内容的信息。如果n>0，Readdir函数会返回一个最多n个成员的切片。这时，如果Readdir返回一个空切片，它会返回一个非nil的错误说明原因。如果到达了目录f的结尾，返回err会是io.EOF。如果n<=0, Readdir函数返回目录中剩余所有文件对象的FileInfo构成的切片。此时，如果Readdir调用成功(读取所有内容直到结尾），它会返回该切片和nil的错误值。如果在到达结尾前遇到错误，会返回之前成功读取的FileInfo构成的切片和该错误

func (f *File) Readdirnames(n int) (names []string, err error) 
// Readdir读取目录f的内容, 返回一个有n个成员的[]string，切片成员为目录中文件对象的名字，采用目录顺序。对本函数的下一次调用会返回上一次调用剩余未读取的内容的信息。如果n>0，Readdir函数会返回一个最多n个成员的切片。这时，如果Readdir返回一个空切片，它会返回一个非nil的错误说明原因。如果到达了目录f的结尾，返回值err会是io.EOF。如果n<=0，Readdir函数返回目录中剩余所有文件对象的名字构成的切片。此时，如果Readdir调用成功（读取所有内容直到结尾），它会返回该切片和nil的错误值。如果在到达结尾前遇到错误，会返回之前成功读取的名字构成的切片和该错误。

func (f *File) Truncate(size int64) error 
// Truncate改变文件的大小，它不会改表I/O的当前位置。如果截断文件，多出的部分就会被丢弃。如果出错，错误底层类型是*PathError

func Remove(name string) error 
// 删除文件

func RemoveAll(name string) error 
// 删除目录
```