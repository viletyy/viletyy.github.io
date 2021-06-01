---
layout: post
title: "Golang: CSP模型"
date:  2020-06-02 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
---
Go语言的最大两个亮点，一个是goroutine，一个就是channel了。二者合体的典型应用CSP，基本就是大家认可的并行开发神器，简化了并行程序的开发难度。

### CSP是什么
CSP是Communicating Sequential Process简称，中文可以叫做通信顺序进程，是一种并发编程模型，是一个很强大的并发数据模型，是上个世纪七十年代提出的，用于描述两个独立的并发实体通过共享的通讯channel（管道）进行通信的并发模型。相对于Actor模型，CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel

严格来说，CSP说一门形式语言（类似于calculus），用于描述并发系统中的互动模式，因此称为一众面向并发的编程语言的理论源头，并衍生出了Occam/Limbo/Golang...

而具体到编程语言，如golang，其实只用到了CSP的很小部分，即理论中的Process/Channel（对应到语言中的goroutine/channel）：这两个并发原语之间没有从属关系。Process可以订阅任意个channel，channel也并不关心是哪个Process在利用它通信；Process围绕Channel进行读写，形成一套有序阻塞和可预测的并发模型。

### Golang CSP
与主流语言通过共享内存来进行并发控制方式不太一样，Go语言采用的是CSP模式。这是一种描述两个独立的并发实体通过共享的通讯Channel（管道）进行通信的并发模型。

Golang就是借用CSP模型的一些概念为之实现并发进行理论支持，其实从实际出发，go语言并没有完全实现CSP模型的所有理论，仅仅是借用了process和channel这两个概念。process是在go语言上表现就是goroutine上实际并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。

Go语言的CSP模型是由协程Goroutine和通道Channel实现：

1. Goroutine：是一种轻量线程，它不是操作系统的线程，而是将一个操作系统线程分段使用，通过调度器实现写作式调度。是一种绿色线程，微线程。
2. Channel：类似Unix的Pipe，用于协程之间通讯和同步，协程之间虽然解耦，但是它们和Channel有着耦合。

### Channel
Goroutine和Channel是Go语言并发编程的两大基石。Goroutine用于执行并发任务，channel用于goroutine之间的同步和通讯。

Channel在goroutine间架起了一条管道，在管道里传输数据，实现gouroutine之间的通信；由于它是线程安全的，所以用起来非常方便；channel还提供“先进先出”的特性；它还能影响goroutine的阻塞和唤醒。

> Do not communicate by sharing memory; instead, share memory by communicating.
> 不要通过共享内存来通信，要逃过通信来共享内存

这就是Go的并发哲学，它依赖CSP模型，基于channel实现。

Channel 是 Go 语言中一个非常重要的类型，是 Go 里的第一对象。通过 channel，Go 实现了通过通信来实现内存共享。Channel 是在多个 goroutine 之间传递数据和同步的重要手段。

使用原子函数、读写锁可以保证资源的共享访问安全，但使用 channel 更优雅。

channel 字面意义是 “通道”，类似于 Linux 中的管道。声明 channel 的语法如下：
```go
chan T // 声明一个双向通道
chan <- T // 声明一个只能用于发送的通道
<-chan T // 声明一个只能用于接收的通道
```

单向通道的声明用`<-`来表示，它指明通道的方向。你只要明白，代码的书写顺序是从左到右就马上能掌握通道的方向是怎样的。

因为channel是一个引用类型，所以它在被初始化之前，它的值是nil，channel使用make函数进行初始化。可以向它传递一个int值，代表channel缓冲区的大小（容量），构造出来的是一个缓冲型的channel；不传或传0时，构造的就是一个非缓冲型的channel

两者有一定的差别：非缓冲型channel无法缓冲元素，对它的操作一定顺序是“发送->接收->发送->接收->...“。如果想连续向一个非缓冲chan发送2个元素，并且没有接收的话，第一次一定会阻塞；对于缓冲型channel的操作，则要”宽松“一些，毕竟是带了”缓冲“光环。

对 chan 的发送和接收操作都会在编译期间转换成为底层的发送接收函数。

Channel 分为两种：带缓冲、不带缓冲。对不带缓冲的 channel 进行的操作实际上可以看作 “同步模式”，带缓冲的则称为 “异步模式”。

同步模式下，发送方和接收方要同步就绪，只有在两者都 ready 的情况下，数据才能在两者间传输（后面会看到，实际上就是内存拷贝）。否则，任意一方先行进行发送或接收操作，都会被挂起，等待另一方的出现才能被唤醒。

异步模式下，在缓冲槽可用的情况下（有剩余容量），发送和接收操作都可以顺利进行。否则，操作的一方（如写入）同样会被挂起，直到出现相反操作（如接收）才会被唤醒。

小结一下：同步模式下，必须要使发送方和接收方配对，操作才会成功，否则会被阻塞；异步模式下，缓冲槽要有剩余容量，操作才会成功，否则也会被阻塞。

简单来说，CSP 模型由并发执行的实体（线程或者进程或者协程）所组成，实体之间通过发送消息进行通信， 这里发送消息时使用的就是通道，或者叫 channel。

CSP 模型的关键是关注 channel，而不关注发送消息的实体。Go 语言实现了 CSP 部分理论，goroutine 对应 CSP 中并发执行的实体，channel 也就对应着 CSP 中的 channel。

### Goroutine调度模型——G-P-M模型
在操作系统提供的内核线程之上，Go搭建了一个特有的两级线程模型。goroutine机制实现了M : N的线程模型，goroutine机制是协程（coroutine）的一种实现，golang内置的调度器，可以让多核CPU中每个CPU执行一个协程。

### 最后
Golang 的 channel 将 goroutine 隔离开，并发编程的时候可以将注意力放在 channel 上。在一定程度上，这个和消息队列的解耦功能还是挺像的。如果大家感兴趣，还是来看看 channel 的源码吧，对于更深入地理解 channel 还是挺有用的。

Go 通过 channel 实现 CSP 通信模型，主要用于 goroutine 之间的消息传递和事件通知。

有了 channel 和 goroutine 之后，Go 的并发编程变得异常容易和安全，得以让程序员把注意力留到业务上去，实现开发效率的提升。

要知道，技术并不是最重要的，它只是实现业务的工具。一门高效的开发语言让你把节省下来的时间，留着去做更有意义的事情，比如写写文章。

* [CSP理论模型](http://www.usingcsp.com/cspbook.pdf)
