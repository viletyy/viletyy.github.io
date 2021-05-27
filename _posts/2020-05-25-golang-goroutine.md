---
layout: post
title: "Golang：Goroutine"
date:  2020-05-25 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
---
### 什么是Goroutine

Go中使用Goroutine来实现并发concurrently。

Goroutine是Go语言特有的名词。区别于进程Process，线程Thread，协程Coroutine，因为Go语言的创造者们觉得和他们是有所区别的，所以专门创造了Goroutine

Goroutine是与其他函数或方法同时运行的函数或方法。Goroutines可以被认为是轻量级的线程。与线程相比，创建Goroutine的成本很小，它就是一段代码，一个函数入口。以及在堆上为其分配的一个堆栈（初始大小为4K，会随着程序的执行自动增长删除）。因此它非常廉价，Go应用程序可以并发运行数千个Goroutines

#### Goroutines在线程上的优势

1. 与线程相比，Goroutines非常偏移。它们只是堆栈大小的几个kb，堆栈可以根据应用程序的需要增长和收缩，而在线程的情况下，堆栈大小必须指定并且是固定的。
2. Goroutines被多路复用到较少的OS线程。在一个程序中可能只有一个线程与数千个Goroutines。如果线程中的任何Goroutine都表示等待用户输入，则会创建另一个OS线程，剩下的Gooutines被转移到新的OS线程。所有这些都由运行时进行处理，我们作为程序员从这些复杂的细节中抽象出来，并得到了一个与并发工作相关的干净的API。
3. 当使用Goroutines访问共享内存时，通过设计的通道可以防止竞态条件发生。通道可以被认为是Goroutines通信的管道。

#### 主Goroutine
**封装main函数的Goroutine称为主Goroutine**

主Goroutine所做的事情并不是执行main函数那么简单。它首先要做是：设定每一个goroutine所能申请的栈空间的最大尺寸。在32位的计算机系统中此最大尺寸为250MB，而在64位的计算机系统中此尺寸为1GB。如果有某个goroutine的栈空间尺寸大于这个限制，那么运行时系统就会引发一个栈溢出（stack overflow）的运行时恐慌。随后，这个go程序的运行也会终止。

此后，主Goroutine会进行一系列的初始化工作，涉及的工作内容大致如下：

1. 启动专用于在后台清扫内存垃圾的Goroutine，并设置GC可用的标识。
2. 执行main包中的init函数。
3. 执行main函数

执行完main函数后，它还会检查主goroutine是否引发了运行时恐慌，并进行必要的处理。最后主Goroutine会结束自己以及当前进程的运行。

#### 如何使用Goroutines

在函数或方法调用前面加上关键字go，您将会同时运行一个新的Goroutine。

示例：

```go 
package main 

import (
    "fmt"
)

func hello() {
    fmt.Println("Hello world goroutine")
}

func main() {
    go hello()
    fmt.Println("main function")
}
```

运行结果：可能会值输出“main function”。

**Goroutine的规则：**

 1. 当新的Goroutine开始时，Goroutine调用立即返回。与函数不同，Go不等待Goroutine执行结束。当Goroutine调用，并且Goroutine的任何返回值被忽略之后，go立即执行到下一行代码
 2. main的Goroutine应该为其他的Goroutine执行。如果main的Goroutine终止了，程序将被终止，而其他Goroutine将不会运行

修改以上代码：

```go 
package main 

import (
    "fmt"
    "time"
)

func hello() {
    fmt.Println("Hello world goroutine")
}

func main() {
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```

运行结果：得到两行输出

在上面的程序中，我们调用了时间包的Sleep方法，它会在执行过程中睡觉。在这种情况下，main的Goroutine被用来睡觉1秒。现在调用go hello()有足够的时间在main Goroutine终止之前执行。这个程序首先打印Hello world goroutine， 等待1秒，然后打印main函数。

#### 启动多个Goroutines

示例：

```go 
package main 

import (
    "fmt"
    "time"
)

func numbers() {
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d", i)
    }
}

func alphabets() {
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c", i)
    }
}

func main() {
    go numbers() 
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main terminated")
}
```

运行结果：
`1 a 2 3 b 4 c 5 d e main terminated`

时间轴分析；
![](/img/in-post/2020-05-25-golang-goroutine-01.png)
