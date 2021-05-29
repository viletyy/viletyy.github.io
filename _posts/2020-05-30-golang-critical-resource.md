---
layout: post
title: "Golang：临界资源安全问题"
date:  2020-05-30 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 代码安全
---
### 临界资源
指并发环境中多个进程/线程/协程贡献的资源。

但是在并发编程中对临界资源的处理不当，往往会导致数据不一致的问题。

#### 示例代码
```go
package main 

import (
    "fmt"
    "time"
)

func main() {
    a := 1
    go func() {
        a = 2
        fmt.Println("子goroutine。。", a)
    }()
    a = 3
    time.Sleep(1)
    fmt.Println("main goroutine..", a)
}
```

### 临界资源安全问题
并发本身并不复杂，但是因为有了资源竞争的问题，就使我们开发出好的并发程序变得复杂起来，因为会引起很多莫名其妙的问题。

如果多个goroutine在访问同一个数据资源的时候，其中一个线程修改了数据，那么这个数值就被修改了，对于其他的goroutine来讲，这个数值可能是错误的。

举个例子。我们通过并发来实现火车站售票这个功能。一共有100张票，4个售票口同时出售

#### 示例代码
```go 
package main 

import (
    "fmt"
    "math/rand"
    "time"
)

// 全局变量
var ticket = 100

func main() {
    // 4个goroutine，模拟4个售票口，4个子程序操作同一个共享数据。
    go saleTickets("售票口1") // g1, 100
    go saleTickets("售票口2") // g2, 100
    go saleTickets("售票口3") // g3, 100
    go saleTickets("售票口4") // g4, 100
    
    time.Sleep(5*time.Second)
}

func saleTickets(name string) {
    rand.Seed(time.Now().UnixNano())
    for {
        if ticket > 0 {
            // 睡眠
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
            // g1, g3, g2, g4
            fmt.Println(name, "售出：", ticket)
            ticket--
        } else {
            fmt.Println(name, "售罄，没有票了。。")
            break
        }
    }
}
```
为了更好的观察临界资源问题，每个goroutine先睡眠一个随机数，然后再售票，发现程序的运行结果，还可以卖出编号为负数的票。
#### 分析
我们买票逻辑是先判断票数的编号是否为负数，如果大于0，然后我们就进行卖票，只不过在卖票先睡眠，然后再卖，假如说此时已经卖票到只剩一张票了，某一个goroutine持有了CPU的时间片，那么它在判断是否有票的时候，条件是成立的，所以它可以卖票编号为1的最后一张票。但是因为它在卖之前，先睡眠了，那么其他的goroutine就会持有CPU的时间片，而此时这张票还没有被卖出，那么第二个gotoutine再判断是否有票的时候，条件也是成立的，那么它也可以卖出这张票，然后它也进入了睡眠。其他的第三个第四个goroutine都是这样的逻辑，当某个goroutine醒来的时候，不会再判断是否有票，而是直接售出了，这样就卖出最后一张票了，然而其他的goroutine醒来的时候就会陆续卖出第0张，-1张，-2张。

这就是临界资源的不安全问题。某一个goroutine在访问某个数据资源的时候，按照数值，已经判断好了条件，然后又被其他的goroutine抢占了资源，并修改了数值，等这个goroutine再继续访问这个数据的时候，数值已经不对了。

### 临界资源安全问题的解决
要想解决临界资源安全的问题，很多编程语言的解决方案都是同步。通过上锁的方式，某一个时间段，只能允许一个goroutine来访问这个共享数据，当前goroutine访问完毕，解锁后，其他的goroutine才能访问。
#### 借助sync包下的锁操作
```go
package main

import (
    "fmt"
    "math/rand"
    "time"
    "sync"
)

var ticket = 10 

var wg sync.WaitGroup 
var matex sync.Mutex // 创建锁头

func main() {
    wg.Add(4)
    go saleTickets("售票口1")
    go saleTickets("售票口2")
    go saleTickets("售票口3")
    go saleTickets("售票口4")
    wg.Wait()
}

func saleTickets(name string) {
    rand.Seed(time.Now().UnixNano())
    defer wg.Done()
    for {
        matex.Lock()
        if ticket > 0 {
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
            fmt.Println(name, "售出：", ticket)
            ticket--
        } else {
            matex.Unlock()
            fmt.Println(name, "售罄，没有票了。。")
            break
        }
    }
    matex.Unlock()
}
```

### 写在最后
> 不要以共享内存的方式去通信，而要以通信的方式去共享内存。

在Go语言中并不鼓励用锁保护共享状态的方式在不同的Goroutine中分享信息（以共享内存的方式去通信）。而是鼓励通过**channel**将共享状态或共享状态的变化在各个Goroutine直接传递（以通信的方式去共享内存），这样同样能像用锁一样保证在同一个时间只有一个Goroutine访问共享状态

当然，在主流的编程语言中未来保证多线程直接共享数据安全性和一致性，都会提供一套基本的同步工具集，如锁，条件变量，原子操作等等。Go语言标准库也毫不意外的提供了这些同步机制，使用方式也和其他语言也差不多。