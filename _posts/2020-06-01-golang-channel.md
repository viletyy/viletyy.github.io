---
layout: post
title: "Golang: 通道"
date:  2020-06-01 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
---
通道可以被认为是Goroutine通信的管道。类似于管道中的水从一端到另一端的流动，数据可以从一端发送到另一端，通过通道接收。

在前面讲Go语言的并发的时候，当多个Goroutine想实现共享数据的时候，虽然也提供了传统的同步机制，但是Go语言强烈建议的是使用Channel通道来实现Goroutines之间的通信。

> 不要通过共享内存来通信，而应该通过通信来共享内存

Go语言中，要传递某个数据给另一goroutine(协程),可以把这个数据封装程一个对象，然后把这个对象的指针传入某个channel中，另外一个goroutine从这个channel中读出这个指针，并处理其指向的内存对象。Go从语言层面保证同一时间只有一个goroutine能够访问channel里面的数据，为开发者提供了一种优雅简单的工具，所以Go的做法就是使用channel来通信，通过通信来传递内存数据，使得内存数据在不同的goroutine中传递，而不是使用共享内存来通信。

### 什么是通道
通道是什么，通道就是goroutine之间的通道。它可以让goroutine之间相互通信。

每个通道都有与其相关的类型。该类型是通道允许传输的数据类型。（通道的零值为nil。nil通道没有任何用处，因此通道必须使用类似于map和切片的方法来定义）
#### 声明通道
声明一个通道和定义一个变量的语法一样：
```go 
// 声明通道
var 通道名 chan 数据类型
// 创建通道：如果通道为nil(就是不存在)， 就需要先创建通道
通道名 = make(chan 数据类型)
```
示例：
```go
package main 

import "fmt"

func main() {
    var a chan int 
    if a == nil {
        fmt.Println("channel是nil的，不能使用，需要先创建通道。。")
        a = make(chan int)
        fmt.Printf("数据类型是：%T", a)
    }
}
```
也可以简短的声明：
```go 
a := make(chan int)
```

#### 通道的数据类型
channel是引用类型的数据，在作为参数传递的时候，传递的是内存地址

示例
```go
package main 

import ( 
    "fmt"
)

func main() {
    ch1 := make(chan int)
    fmt.Printf("%T, %P\n", ch1, ch1)
    
    test1(ch1)
}

func test1(ch chan int) {
    fmt.Printf("%T, %P\n", ch, ch)
}
```
输出结果
```shell
chan int, 0xc00008e000
chan int, 0xc00008e000
```

#### 通道的注意点

1. 通道是用于goroutine，传递消息的
2. 通道，每个都有相关联的数据类型，nil chan：不能使用，类似于nil map, 不能之间存储键值对
3. 使用通道传递数据: <- chan <- data,发送数据到通道。data <- chan， 从通道中获取数据
4. 阻塞：发送数据： chan <- data，阻塞的，直到另一条goroutine，读取数据来解除阻塞。读取数据：data <- chan，也是阻塞的。直到另一条goroutine，写出数据解除阻塞。
5. 本身channel就是同步的，意味着同一时间，只能有一条goroutine来操作

最后：通道是goroutine之间的连接，所以通道的发送和接收必须处在不同的goroutine中。

### 通道使用语法
#### 发送和接收
```go
data := <- a // read from channel a 
a <- data // write to channel a 
```
#### 发送和接收默认是阻塞的
一个通道发送和接收数据，默认是阻塞的。当一个数据被发送到通道时，在发送语句中被阻塞，直到另一个Goroutine从该通道读取数据。相对地，当从通道读取数据时，读取被阻塞，直到一个Goroutine将数据写入该通道。

这些通道的特性时帮助Goroutines有效地进行通信，而无需像使用其他编程语言中非常常见的显式锁或条件变量。

```go 
package main 

import "fmt"

func main() {
    var ch1 chan bool 
    fmt.Println(ch1)
    fmt.Printf("%T\n", ch1)
    ch1 = make(chan bool)
    fmt.Println(ch1)
    
    go func() {
        for i:= 0; i <10; i++ {
            fmt.Println("子goroutine中, i:", i)
        }
        // 循环结束后，向通道中写数据，表示要结束了。。
        ch1 <- true 
        fmt.Println("结束")
    }()
    
    data := <-ch1 // 从ch1通道中读取数据
    fmt.Println("data-->", data)
    fmt.Println("main..Over...")
}
```
在上面的程序中，我们先创建了一个chan bool通道。然后启动了一个子Goroutine，并循环打印10个数字。然后我们向通道ch1中写入输入true。然后在主goroutine中，我们从ch1中读取数据。这一行代码是阻塞的，这意味着在子Goroutine将数据写入到该通道之前，主goroutine将不会执行到下一行代码。因此，我们可以通过channel实现子goroutine和主goroutine之间的通信。当子goroutine执行完毕前，主goroutine会因为读取ch1中的数据而阻塞。从而保证了子goroutine会执行完毕。这就消除了对时间的需求。在之前的程序中，我们要么让主goroutine进入睡眠，以防止主要的Goroutine退出。要么通过WaitGroup来保证子goroutine先执行完毕，主goroutine才结束。
```go 
package main 

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan int)
    done := make(chan bool)
    go func() {
        fmt.Println("子goroutine执行")
        time.Sleep(3 * time.Second)
        data := <-ch1 
        fmt.Println("data:", data)
        done <- true
    }()
    
    // 向通道中写数据
    time.Sleep(5 * time.Second)
    ch1 <- 100 
    
    <- done
    fmt.Println("main..over")
}
    
```
打印一个数字的每位数的平方和
```go
package main 

import (
    "fmt"
)

func calcSquares(number int, squareop chan int) {
    sum := 0
    for num != 0 {
        digit := number % 10
        sum += digit * digit 
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {
    sum := 0
    for number != 0 {
        digit := number % 19
        sum += digit * digit * digit 
        number /= 10
    }
    cubeop <- sum
}

func main() {
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```
运行结果
`Final output 1536`

#### 死锁
使用通道时要考虑的一个重要因素是死锁。如果Goroutine在一个通道上发送数据，那么预计其他的Goroutine应该接收数据。如果没有发生，那么程序将会在运行的时候出现死锁。

类似地，如果Goroutine正在等待从通道接收数据，那么其他的Goroutine应该写入数据，否则程序也会出现死锁。

> 在主流的编程语言中味了保证多线程之间共享数据安全性和一致性，都会提供一套基本的同步工具集，如锁，条件变量，原子操作等等。Go语言标准库也毫不意外的提供了这些同步机制，使用方式也和其他语言差不多。除了这些基本的同步手段，Go语言还提供了一种新的同步机制：Channel，它在Go语言中是一个像int，float32等的基本类型，一个channel可以认为是一个能够在多个Goroutine之间传递某一类型的数据的管道。Go中的channel无论是实现机制还是使用场景都和Java中的BlockingQueue很接近。

### 关闭通道
发送者可以通过关闭信道，来通知接收方不会有更多的数据被发送到channel上。
```go
close(ch)
```
接收者可以在接收来自通道的数据时使用额外的变量来检查通道是否已经关闭。
```go
v, ok := <- ch  
```
> 类似map操作，存储key，value键值对
> v,ok := map[key] //根据key从map中获取value，如果key存在， v就是对应的数据，如果key不存在，v是默认值

在上面的语句中，如果ok的值是true，表示成功的从通道中读取了一个数据value。如果ok是false，这意味着我们正在从一个封闭的通道读取数据。从闭通道读取的值将是通道类型的零值。

例如，如果通道是一个int通道，那么从封闭通道接收的值将为0。

示例：
```go
package main 

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan int)
    go sendData(ch1)
    for {
        time.Sleep(1 * time.Second)
        v, ok := <- ch1
        if !ok {
            fmt.Println("已经读取了所有的数据，", ok)
            break
        }
        fmt.Println("取出数据：", v, ok)
    }
    fmt.Println("main..over..")
}

func sendData(ch1 chan int) {
    // 发送方：10条数据
    for i := 0; i < 10; i++ {
        ch1 <- i
    }
    close(ch1)
}
```
在上面的程序中。send Goroutine将0到9写入ch1通道，然后关闭通道。主函数里有一个无限循环。它检查通道是否在发送数据后，使用变量ok关闭。如果ok是false，则意味着通道关闭，因此循环结束。还可以打印接收到的值和ok的值。

### 通道上的范围循环
我们可以循环从通道上获取数据，直到通道关闭。for循环的for range形式可用于从通道接收值，直到它关闭为止。
示例：
```go
package main

import (
    "time"
    "fmt"
)

func main()  {
    ch1 :=make(chan int)
    go sendData(ch1)
    // for循环的for range形式可用于从通道接收值，直到它关闭为止。
    for v := range ch1{
        fmt.Println("读取数据：",v)
    }
    fmt.Println("main..over.....")
}
func sendData(ch1 chan int)  {
    for i:=0;i<10 ; i++ {
        time.Sleep(1*time.Second)
        ch1 <- i
    }
    close(ch1)//通知对方，通道关闭
}
```
### 缓冲通道
之前学习的所有通道基本上都没有缓冲。发送和接收到一个未缓冲的通道是阻塞的。

一次发送操作对应一次接收操作，对于一个goroutine来讲，它的一次发送，在另一个goroutine接收之前都是阻塞的。同样的，对于接收来讲，在另一个goroutine发送之前，它也是阻塞的。

缓冲通道就是指一个通道，带有一个缓冲区。发送到一个缓冲通道只有在缓冲区满时才被阻塞。类似地，从缓冲通道接收的信息只有在缓冲区为空时才会被阻塞。

可以通过将额外的容量参数传递给make函数来创建缓冲通道，该函数指定缓冲区的大小。

语法：
```go
ch := make(chan type, capacity)
```
上述语法的容量应该大于0，以便通道具有缓冲区。默认情况下，无缓冲通道的容量为0，因此在之前创建通道时省略了容量参数。

示例：
```go
package main 

import (
    "fmt"
    "strconv"
    "time"
)

func main() {
    ch1 := make(chan int)           // 非缓存的通道
    fmt.Println(len(ch1), cap(ch1))  // 0 0 
    ch2 := make(chan int, 5)
    fmt.Println(len(ch2), cap(ch2))  // 0 5
    ch2 <- 100
    fmt.Println(len(ch2), cap(ch2))  // 1 5
    
    ch3 := make(chan string, 4)
    go sendData3(ch3)
    for {
        time.Sleep(1 * time.Second)
        v, ok := <-ch3
        if !ok {
            fmt.Println("读完了。。", ok)
            break
        }
        fmt.Println("\t读取的数据是：", v)
    }
    
    fmt.Println("main..over..")
}

func sendData3(ch3 chan string) {
    for i := 0; i < 10; i++ {
        ch3 <- "数据" + strconv.Itoa(i)
        fmt.Println("子goroutine，写出第", i, "个数据")
    }
    close(ch3)
}
```
### 定向通道
通道，channel，是用于实现goroutine之间的通信。一个goroutine可以向通道中发送数据，另一条goroutine可以从该通道中获取数据。截止到现在所接触的都是既可以发送数据也可以读取数据的，这种叫双向通道。

> 单向通道，也就是定向通道

之前我们学习的通道都是双向通道，我们可以通过这些通道接收或者发送数据。我们也可以创建单向通道，这些通道只能发送或者接收数据。

双向通道，示例：
```go
package main 

import "fmt"

func main() {
/*
    双向：
        chan T -->
            chan <- data, 写数据
            data <- chan, 读数据
    单向：
        chan <- T, 只支持写
        <- chan T, 只读
*/
    ch1 := make(chan string)
    done := make(chan bool)
    go sendData(ch1, done)
    data := <- ch1 // 阻塞
    fmt.Println("子goroutine传来：", data)
    ch1 <- "我是main" // 阻塞
    
    <- done
    fmt.Println("main..over...")
}

//子goroutine-->写数据到ch1通道中
//main goroutine-->从ch1通道中取
func sendData(ch1 chan string, done chan bool)  {
    ch1 <- "我是小明"// 阻塞
    data := <-ch1 // 阻塞
    fmt.Println("main goroutine传来：",data)

    done <- true
}
```
创建仅能发送数据的通道，示例代码：
```go
package main 

import "fmt"

func main() {
    /*
        单向：
            chan <- T, 只支持写
            <- chan T, 只读
    */
    ch1 := make(chan int)
    
    go fun1(ch1)
    data := <- ch1 
    fmt.Println("fun1中写出的数据是：", data)
    
    go fun2(ch1)
    ch1 <- 200
    fmt.Println("main...over...")
}

// 该函数接收，只写的通道
func fun1(ch chan <- int) {
    // 函数内部，对于ch只能写数据，不能读数据
    ch <- 100 
    fmt.Println("fun1函数结束。。")
}

// 该函数接收，只读的通道
func fun2(ch <- chan int) {
    data := <- ch
    fmt.Println("fun2函数,从ch中读取的数据是:",data)
}
```

### time包中的通道相关函数
> 主要是定时器，标准库中的Timer让用户可以定义自己的超时逻辑。尤其是在应对select处理多个channel的超时、单channel读写的超时等情形时尤为方便。

Timer是一次性的时间触发事件，而Ticker是按照一定时间间隔持续触发事件。

常见的创建方式：
```go
t := time.NewTimer(d) // 定时时间：就是d
t := time.AfterFunc(d, f) // 触发动作：就是f
c := time.After(d) // 时间channel：也就是t.C
```

#### time.NewTimer()
NewTimer()创建一个新的计时器，该计时器将在其通道上至少持续d之后发送当前时间。它的返回值是一个Timer。

源代码：
```go
// NewTimer creates a new Timer that will send
// the current time on its channel after at least duration d.
func NewTimer(d Duration) *Timer {
    c := make(chan Time, 1)
    t := &Timer {
        C: c,
        r: runtimeTimer{
            when: when(d),
            f: sendTime,
            arg: c,
        },
    }
    startTimer(&t.r)
    return t
}
```
通过源代码我们可以看出，首先创建一个channel，关联的类型为Time，然后创建了一个Timer并返回。

* 用于在指定的Duration类型时间后调用函数或计算表达式
* 如果只是想指定时间后执行。使用time.Sleep()
* 使用NewTimer(),可以返回Timer类型在计时器到期之前，取消该计时器。
* 直到使用<-timer.C发送一个值，该计时器才会过期

```go
package main 

import (
    "time"
    "fmt"
)

func main() {
    // 新建一个计时器：timer
    timer := time.NewTimer(3 * time.Second)
    fmt.Printf("%T\n", timer) // *time.Timer
    fmt.Println(time.Now()) // 2021-06-01 10:41:21.800768 +0800 CST m=+0.000461190
    // 此处在等待channel中的信号，执行此段代码时会阻塞3秒
    ch2 := time.C
    fmt.Println(<-ch2) //2021-06-01 10:41:24.803471 +0800 CST m=+3.003225965
}
```

#### timer.Stop
timer.Stop是计时器停止

示例：
```go
package main

import (
    "time"
    "fmt"
)

func main() {
    timer2 := time.NewTimer(5 * time.Second)
    // 新开goroutine处理触发后的事件
    go func() {
        //等触发时的信号

        <-timer2.C

        fmt.Println("Timer 2 结束。。")
    }()
    
    // 由于上面的等待信号是在新线程中，所以代码会继续往下执行
    time.Sleep(3 *time.Second)
    stop := timer.Stop()
    
    if stop {
        fmt.Println("Timer 2 停止。。")
    }
}
```

#### time.After()
在等待持续时间之后，然后在返回的通道上发送当前时间。它相当于NewTimer(d).C。在计时器触发之前，垃圾收集器不会恢复底层计时器。如果效率有问题，使用NewTimer代替，并调用Timer。如果不再需要计时器，请停止。

源码：
```go
func After(d Duration) <- chan Time {
    return NewTimer(d).C
}
```
示例：
```go
package main

import (
    "time"
    "fmt"
)

func main() {
    ch1 := time.After(3 * time.Second) //3s后
    fmt.Printf("%T\n", ch1) // <-chan time.Time
    fmt.Println(time.Now()) //2021-06-01 09:56:41.529883 +0800 CST m=+0.000465158
    time2 := <-ch1
    fmt.Println(time2) //2021-06-01 09:56:44.532047 +0800 CST m=+3.002662179
}
```

### select语句
#### 语法结构
select语句的语法结构和switch语句很相似，也有case语句和default语句：
```go
select {
    case communication clause  :
       statement(s);      
    case communication clause  :
       statement(s); 
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s);
}
```
说明：

* 每个case都必须是一个通信
* 所有channel表达式都会被求值
* 所有被发送的表达式都会被求值
* 如果有多个case都可以运行，select会随机公平的选出一个执行，其他不会执行
* 否则：
    * 如果default子句，则执行该语句
    * 如果没有default子句，select将堵塞，直到某个通信可以运行：go不会重新对channel或值进行求值

#### 示例
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    /*
    分支语句：if，switch，select
    select 语句类似于 switch 语句，
        但是select会随机执行一个可运行的case。
        如果没有case可运行，它将阻塞，直到有case可运行。
     */
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- 200
    }()
    go func() {
        time.Sleep(2 * time.Second)
        ch1 <- 100
    }()

    select {
    case num1 := <-ch1:
        fmt.Println("ch1中取数据。。", num1)
    case num2, ok := <-ch2:
        if ok {
            fmt.Println("ch2中取数据。。", num2)
        }else{
            fmt.Println("ch2通道已经关闭。。")
        }

    }
}
```
运行结果：可能执行第一个case，打印100，也可能执行第二个case，打印200。(多运行几次，结果就不同了)

select语句结合time包的和chan相关函数，示例代码：
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    //go func() {
    //  ch1 <- 100
    //}()

    select {
    case <-ch1:
        fmt.Println("case1可以执行。。")
    case <-ch2:
        fmt.Println("case2可以执行。。")
    case <-time.After(3 * time.Second):
        fmt.Println("case3执行。。timeout。。")

    //default:
    //  fmt.Println("执行了default。。")
    }
}
```