---
layout: post
title: "Golang：I/O操作"
date:  2020-05-19 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 包
---
## I/O操作
I/O操作也叫输入输出操作。其中I是指Input，O是指Output，用于读或者写数据的，有些语言中也叫流操作，是指数据通信的通道。

Golang标准库对IO的抽象非常精巧，各个组件可以随意组合，可以做为接口设计的典范

### io包
io包中提供I/O原始操作的一系列接口。它主要包装了一些已有的实现，如os包中的那些，并将这些抽象成为实用性的功能和一些其他相关的接口

由于这些接口和原始操作以不同的实现包装了低级操作，客户不应假定它们对于并行执行是安全的

### 读操作
Reader接口的定义，Read()方法用于读取数据

```go 
type Reader interface {
    Read(p []byte) (n int, err error) 
}
// Read将len(p)个字节读取到p中，它返回读取到字节数n(0 <= n <= len(p))以及任何遇到的错误。即使Read返回的n < len(p)，它也会在调用过程中使用p的全部作为暂存空间。若一些数据可用但不到len(p)个字节，Read会照例返回可用的东西，而不是等待更多。
// 当Read在成功读取n>0个字节后遇到一个错误或EOF情况，它就会返回读取的字节数，它会从相同的调用中返回(非nil)的错误或从随后的调用中返回错误(和n==0)，这中一般情况的一个例子就是Reader在输入流结束时会返回一个非零的字节数，可能的返回不是err == EOF 就是 err == nil. 无论如何，下一个Read都应当返回0, EOF
// 调用者应当总在考虑到错误err 前处理n 0 的字节，这样做可以在读取一些字节，以及允许的EOF行为后正确地处理I/O错误
// Read的实现会阻止返回零字节的计数和一个nil错误，调用者应将这种请款视作空操作。
```
ReaderFrom接口的定义，封装了基本的ReadFrom方法

```go 
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
// ReadFrom从r中读取数据到对象的数据流中
// 直到r返回EOF或r出现读取错误为止
// 返回n是读取的字节数
// 返回err就是r的返回值err
```
定义ReaderAt接口，ReaderAt接口封装了基本的ReadAt方法

```go
type ReaderAt interface {
    ReadAt(p []byte, off int64) (n int, err error)
}
// ReadAt从对象数据流的off处读出数据到p中
// 忽略数据的读写指针，从数据的起始位置偏移off处开始读取
// 如果对象的数据流只有部分可用，不足以填满p
// 则ReadAt将等待所有数据可用之后,继续向p中写入
// 直到将p填满后再返回
// 在这点上ReadAt要比Read严格
// 返回读取的字节数 n 和读取时遇到的错误
// 如果 n < len(p),则需要返回一个err值来说明
// 为什么没有将p填满(比如EOF)
// 如果 n = len(p) 而且对象的数据没有全部读完，则err将返回nil
// 如果 n = len(p),而且对象的数据刚好读完，则err将返回EOF或者nil(不确定)
```

### 文件读取
file类是在os包中的，封装了底层的文件描述符和相关信息，同时封装了Read和Write的实现

```go 
func(f *File) Read(b []byte) (n int, err error) 
// Read方法从f中读取最多len(b)字节数据并写入b。它返回读取的字节数和可能遇到的任何错误。文件终止标志是读取0个字节且返回值err为io.EOF
func(f *File) ReadAt(b []byte, off int64) (n int, err error)
// ReadAt从指定的位置(相对于文件开始位置)读取len(b)字节数据并写入b。它返回读取的字节数和可能遇到的任何错误。当n小于len(b)时，本方法总是返回错误；如果是因为到达文件结尾，返回值err会是io.EOF.
```

### 写操作
Writer接口的定义，Write()方法用于写出数据

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
// Write 将 len(p) 个字节从 p 中写入到基本数据流中。它返回从 p 中被写入的字节数n（0 <= n <= len(p)）以及任何遇到的引起写入提前停止的错误。若 Write 返回的n < len(p)，它就必须返回一个非nil的错误。Write 不能修改此切片的数据，即便它是临时的。
```
Seeker接口的定义，封装了基本的Seek方法

```go 
type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}
// Seeker用来移动数据的读写指针
// Seek设置下一次读写操作的指针位置，每次的读写操作都是从指针位置开始的
// whence的含义：
//  如果whence为0:表示从数据的开头开始移动指针
//  如果whence为1:表示从数据的当前指针位置开始移动指针
//  如果whence为2:表示从数据的尾部开始移动指针
// offset 是指针移动的偏移量
//  返回移动后的指针位置和移动过程中遇到的任何错误
```
WriterTo接口的定义，封装了基本的WriteTo方法

```go
type WriterTo interface {
    WriteTo(w Writer) (n int64, err error)
}
// WriterTo将对象的数据流写入到w中
//  直到对象的数据流全部写入完毕或遇到写入错误为止
//  返回值n是写入的字节数
//  返回值err就是w的返回值err
```
定义WriterAt接口，WriterAt接口封装了基本的WriteAt方法

```go 
type WriterAt interface {
    WriteAt(p []byte, off int64) (n int, err error)
}
// WriteAt将p中的数据写入到对象数据流的off出
//  忽略数据的读写指针，从数据的起始位置偏移off处开始写入
//  返回写入的字节数和写入时遇到的错误
//  如果n < len(p),则必须返回一个err值来说明
//  为什么没有将p完全写入
```

### 文件写出
file类时在os包中的，封装了底层的文件描述符和相关信息，同时封装了Read和Write的实现

```go 
func (f *File) Write(b []byte) (n int, err error) 
// Write向文件中写入len(b)字节数据。它返回写入的字节数和可能遇到的任何错误。如果返回值n != len(b),本方法会返回一个非nil的错误
func (f *File) WriteString(s string) (ret int, err error) 
// WriteString类似Write，但接受一个字符串参数
func (f *File) WriteAt(b []byte, off int64) (n int, err error) 
// WriteAt在指定的位置（相对于文件开始位置）写入len(b)字节数据。它返回写入的字节数和可能遇到的任何错误。如果返回值n!=len(b),本方法会返回一个非nil的错误
func (f *File) Seek(offset int64, whence int) (ret int64, err error) 
// Seek设置下一次读/写的位置。offset为相对偏移量。而whence决定相对位置：0为相对文件开头，1为相对当前位置，2位相对文件结尾。它返回新的偏移量（相对开头）和可能的错误
func (f *File) Sync()(err error) 
// Sync递交文件的当前内容进行稳定的存储，一般来说，这表示将文件系统的最近写入的数据在内存中的拷贝刷新到硬盘中稳定保存
```