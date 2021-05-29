---
layout: post
title: "Golang: sync包"
date:  2020-05-30 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 包
---
官网文档对sync包的介绍
> Package sync provides basic synchronization primitives such as mutual exclusion locks. Other than the Once and WaitGroup types, most are intended for use by low-level library routines. Higher-level synchronization is better done via channels and communication.
> sync是synchronization同步这个词的缩写，所以也会叫同步包。这里提供了基本同步草错，比如互斥锁等等。这里除了Once和WaitGroup类型之外，大多数类型都是供低级库例程使用的。更高级别的同步最好通过channer通道和communication通信来完成。

### WaitGroup
同步等待组

在类型上，它是一个结构体。一个WaitGroup的用途是等待一个goroutine的集合执行完成。主goroutine调用了Add()方法来设置要等待的goroutine数量。然后，每个goroutine都会执行并且执行完成后调用Done()这个方法。于此同时。可以使用Wait()方法来阻塞，直到所有的goroutine都执行完成。

#### Add()方法
Add()这个方法，用来设置到WaitGroup的计数器的值。我可以理解为每个waitgroup中都有一个计数器用来表示这个同步等待组中要执行的goroutine的数量。

如果计数器的数值变为0，那么就表示等待时被阻塞的goroutine都被释放，如果计数器的数值为负数，那么就会引发恐慌，程序报错了。

#### Done()方法
Done()方法，就是当WaitGroup同步等待组中的某个goroutine执行完毕后，设置这个WaitGroup的counter数值减1

其实Done()的底层代码就是调用了Add()方法
```go
func (wg *WaitGroup) Done() {
    wg.Add(-1)
}
```

#### Wait()方法
Wait()方法，表示让当前的goroutine等待，进入阻塞状态。一直到WaitGroup的计数器为0，才解除阻塞，这个goroutine才继续执行。

#### 示例
我们创建并启动两个goroutine来打印字母和数字，并在main goroutine中，将这两个子goroutine假如到一个WaitGroup中，同时让main goroutine进入Wait(),让两个子goroutine先执行。当每个子goroutine执行完毕后，调用Done()方法，设置WaitGroup的counter减1。当两条子goroutine都执行完毕后，WaitGroup中的counter的数值为零，解除main goroutine的阻塞。

```go
package main 

import ( 
    "fmt"
    "sync"
)

var wg sync.WaitGroup 

func main() {
    wg.Add(2)
    go fun1() 
    go fun2()
    fmt.Println("main进入阻塞状态。。。等待wg中的子goroutine结束。")
    wg.Wait()
    fmt.Println("main，解除阻塞。。")
}

func fun1()  {
    for i:=1;i<=10;i++{
        fmt.Println("fun1.。。i:",i)
    }
    wg.Done() //给wg等待中的执行的goroutine数量减1.同Add(-1)
}
func fun2()  {
    defer wg.Done()
    for j:=1;j<=10;j++{
        fmt.Println("\tfun2..j,",j)
    }
}
```

### Mutex(互斥锁)
锁就是某个协程（线程）在访问某个资源时先锁住，防止其他协程的访问，等访问完毕解锁后其他协程再来加锁进行访问。一般用于处理并发中的临界资源问题。

Go语言中的sync包提供了两种锁类型：sync.Mutex 和 sync.RWMutex

Mutex锁最简单的一种锁类型，互斥锁，同时也比较暴力，当一个goroutine获得了Mutex后，其他goroutine就只能乖乖等到这个goroutine释放该Mutex

每个资源都对应一个可以称为“互斥锁”的标记，这个标记用来保证在任意时刻，只能有一个协程（线程）访问该资源。其他协程只能等待。

互斥锁是传统并发编程对共享资源进行访问控制的主要手段，它由标准库sync中的Mutex结构体类型表示。sync.Mutex类型只有两个公开的执政方法，Lock()和Unlock()。Lock锁定当前的共享资源，Unlock进行解锁。

在使用互斥锁时，一定要注意：对资源操作完成后，一定要解锁，否则会出现流程执行异常，死锁等问题。通常借助defer。锁定后，立即使用defer语句保证互斥锁及时解开。

#### 部分源码
```go
// A Mutex is mutual exclusion lock
// The zero value for a Mutex is an unlocked mutex.
// A Mutex must not be copied after first use.
type Mutex struct {
    state int32 // 互斥锁上锁状态枚举值如下所示
    sema uint32 // 信号量，向处于Gwaitting的G发送信号
}

// A Locker represents an object that can be locked and unlocked
type Locker interface {
    Lock()
    Unlock()
}

const (
    mutexLocked = 1 << iota // mutex is locked, 1.互斥锁是锁定的
    mutexWoken // 2 唤醒锁
    mutexStarving 
    mutexWaiterShift = iota // 统计阻塞在这个互斥锁上的goroutine数目需要移位的数值
    starvationThresholdNs = 1e6
)
```
#### Lock()方法
Lock()这个方法，锁定mutex。如果该mutex已经使用中，则调用goroutine将阻塞，直到互斥体可以用。

#### Unlock()方法
Unlock()方法，解锁锁mutex。如果mutex未在要解锁的条目上锁定，则为运行时错误。

锁定的互斥体不于特定的goroutine关联。允许一个goroutine锁定互斥体，然后安排另一个goroutine解锁互斥体

#### 示例
模拟4个售票口出售火车票的案例。4个售票口同时卖票，会发生临界资源数据安全问题。我们使用互斥锁解决一下。(Go语言推崇的是使用Channel来实现数据共享，但是也还是提供了传统的同步处理方式)
```go
package main

import (
    "fmt"
    "time"
    "math/rand"
    "sync"
)

//全局变量，表示票
var ticket = 10 //100张票

var mutex sync.Mutex //创建锁头

var wg sync.WaitGroup //同步等待组对象
func main() {
    /*
    4个goroutine，模拟4个售票口，

    在使用互斥锁的时候，对资源操作完，一定要解锁。否则会出现程序异常，死锁等问题。
    defer语句
     */

     wg.Add(4)
    go saleTickets("售票口1")
    go saleTickets("售票口2")
    go saleTickets("售票口3")
    go saleTickets("售票口4")

    wg.Wait() //main要等待
    fmt.Println("程序结束了。。。")

    //time.Sleep(5*time.Second)
}

func saleTickets(name string){
    rand.Seed(time.Now().UnixNano())
    defer wg.Done()
    for{
        //上锁
        mutex.Lock() //g2
        if ticket > 0{ //ticket 1 g1
            time.Sleep(time.Duration(rand.Intn(1000))*time.Millisecond)
            fmt.Println(name,"售出：",ticket) // 1
            ticket-- // 0
        }else{
            mutex.Unlock() //条件不满足，也要解锁
            fmt.Println(name,"售罄，没有票了。。")
            break
        }
        mutex.Unlock() //解锁
    }
}

```
### RWMutex(读写锁)
Go语言包中的 sync 包提供了两种锁类型：sync.Mutex 和 sync.RWMutex。其中RWMutex是基于Mutex实现的，只读锁的实现使用类似引用计数器的功能。

RWMutex是读/写互斥锁。锁可以由任意数量的读取器或单个编写器持有。RWMutex的零值锁未锁定的mutex

如果一个goroutine持有一个rRWMutex进行读取，而另一个goroutine可能调用lock，那么在释放初始读写锁止器，任何goroutine都不应该期望能够获取读写锁。特别是，禁止递归读取锁定。这是为了确保锁最终可用；被阻止的锁调用会将新的读卡器排除在获取锁之外。

我们怎么理解读写锁呢？当有一个 goroutine 获得写锁定，其它无论是读锁定还是写锁定都将阻塞直到写解锁；当有一个 goroutine 获得读锁定，其它读锁定仍然可以继续；当有一个或任意多个读锁定，写锁定将等待所有读锁定解锁之后才能够进行写锁定。所以说这里的读锁定（RLock）目的其实是告诉写锁定：有很多人正在读取数据，你给我站一边去，等它们读（读解锁）完你再来写（写锁定）。我们可以将其总结为如下三条：

1. 同时只能有一个goroutine能够获得写锁定
2. 同时可以有任意多个goroutine获得读锁定
3. 同时只能存在写锁定或读锁定（读和写互斥）

所以，RWMutex这个读写锁，该锁可以加多个读锁或者一个写锁，其经常用于读次数远远多于写次数的场景。

读写锁的写锁只能锁定一次，解锁前不能多次锁定，读锁可以多次，但读解锁次数最多只能比读锁次数多一次，一般情况下我们不建议读解锁次数多于读锁次数。

基本遵循两大原则：

1. 可以随便读，多个goroutine同时读
2. 写的时候，啥也不能干。不能读也不能写。

读写锁即是针对于读写操作的互斥锁。它与普通的互斥锁最大的不同就是，它可以分别针对读操作和写操作进行锁定和解锁操作。读写锁遵循的访问控制规则和互斥锁有所不同。在读写锁管辖的范围内，它允许任意个读操作的同时进行。但是在同一时刻，它只允许有一个写操作在进行。

并且在某一个写操作被进行的过程中，读操作的进行也是不被允许的。也就是说读写锁控制下的多个写操作之间都是互斥的，并且写操作与读操作之间也都是互斥的。但是，多个读操作之间却不存在互斥关系
 
#### RLock()方法
读锁，当有写锁时，无法加载读锁，只有读锁活着没有锁时，可以加载读锁，读锁可以加载多个，所以适用于“读多写少”的场景。

#### RUnlock()方法
读锁解锁，RUnlock撤销单次RLock调用，它对于其他同时存在的读取器则没有效果。若RWMutex并没有因为读取而锁定，调用RUnlock就会引发一个运行时错误。

#### Lock()方法
写锁，如果在添加写锁之前已经有其他的读锁和写锁，则Lock就会阻塞直到该锁可用，为确保该锁最终可用，已阻塞的Lock调用会从获得的锁中排除新的读取锁，即写锁权限高于读锁，有写锁时优先进行写锁定。

#### Unlock()方法
写锁解锁，如果没有进行写锁定，则就会引起一个运行时错误。

#### 示例
```go
package main 

import (
    "fmt"
    "sync"
    "time"
)

var rwMutex *sync.RWMutex
var wg *sync.WaitGroup
func main() {
    rwMutex = new(sync.RWMutex)
    wg = new(sync.WaitGroup)
    
    wg.Add(3)
    go writeData(1)
    go readData(2)
    go writeData(3)
    
    wg.Wait()
    fmt.Println("main..over..")
}

func writeData(i int) {
    defer wg.Done()
    fmt.Println(i, "开始写：write start..")
    rwMutex.Lock() //写操作上锁
    fmt.Println(i, "正在写：writing..")
    rwMutex.Unlock()
    fmt.Println(i, "写结束：write over..")
}

func readData(i int) {
    defer wg.Done()
    fmt.Println(i, "开始读：read start..")
    rwMutex.RLock() // 读操作上锁
    fmt.Println(i, "正在读：reading..")
    rwMutex.RUnlock() 
    fmt.Println(i, "读结束：read over..")
}
```

总结：

1. 读锁不能阻塞读锁
2. 读锁需要阻塞写锁，直到所有读锁都释放
3. 写锁需要阻塞读锁，直到所有写锁都释放
4. 写锁需要阻塞写锁