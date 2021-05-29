---
layout: post
title: "Golang：runtime包"
date:  2020-05-29 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 包
---
官网文档对runtime包的介绍

> Package runtime contains operations that interact with Go's runtime system, such as functions to control goroutines.Is also includes the low-level type information used by the reflect package; see reflect's documentation for the programmable interface to the run-time type system

> 尽管Go编译器产生的是本地可执行代码，这些代码仍旧运行在Go的runtime（这部分的代码可以在runtime包中找到）当中。这个runtime类似Java和.NET语言所用到的虚拟机，它负责管理包括内存分配、垃圾回收、栈处理、Goroutine、channel、切片（slice）、map和反射（reflection）等等

### 常用函数
**runtime**调度器是个非常有用的东西，关于**runtime**包几个方法：
- NumCPU: 返回当前系统的**CPU**核数
- GOMAXPROCS: 设置最大的可同时使用**CPU**核数
  > 通过runtime.GOMAXPROCS函数，应用程序可以在运行期间设置运行时系统中的P最大数量。但这会引起“Stop the World”。所以，应该在应用程序最早调用。并且最好是在运行Go程序之前设置好操作程序的环境变量GOMAXPROCS，而不是在程序中调用runtime.GOMAXPROCS函数。
    
  > 无论我们传递给函数的整数值是什么值，运行时系统P的最大值总会在1～256之间。

go1.8后，默认让程序运行在多个核上，可以不用设置，但是在go1.8之前，需要设置，为了更高效的利益cpu
- Gosched：让当前线程让出**CPU**以让其他线程运行，它不会挂起当前线程，因此当前线程未来会继续执行
- Goexit: 退出当前**Goroutine**（但是**defer**语句会照常执行）
- NumGoroutine: 返回正在执行核排队的任务总数
  > runtime.NumGoroutine函数在被调用后，会返回系统中的处于特定状态的Goroutine的数量。这里特指的是指Grunnable\Grunning\Gsyscall\Gwaition。处于这些状态的Goroutine即被看作是活跃的活着说正在被调度的。
    
  > 注意：垃圾回收所在Goroutine的状态也处于这个范围内的话，也会被纳入该计数器
    
- GOOS: 目标操作系统，很多时候我们会根据平台的不同实现不同操作，就可以使用它
- runtime.GC: 会让运行时系统进行一次强制性的垃圾收集
    1. 强制的垃圾回收：不管怎样，都要进行的垃圾回收
    2. 非强制的垃圾回收：只会在一定条件下进行的垃圾回收（即运行时，系统自上次垃圾回收一周新申请的堆内存的担忧（也称为单元增量）达到指定的数值）。
- GOROOT: 获取goroot目录

### 示例
#### 获取GOROOT和OS
```go
// 获取GOROOT目录：
fmt.Println("GOROOT-->", runtime.GOROOT())

// 获取操作系统
fmt.Println("os.platform-->", runtime.GOOS)
```

#### 获取CPU数量，和设置CPU数量
```go 
func init() {
    // 1. 获取逻辑cpu的数量
    fmt.Println("逻辑CPU的核数：", runtime.NumCPU())
    // 2. 设置go程序执行的最大cpu数
    n := runtime.GOMAXPROCS(runtime.NumCPU())
    fmt.Println(n)
}
```

#### 让出时间片，先让别的协议执行
```go
func main() {
    go func() {
        for i := 0; i < 5; i++ {
            fmt.Println("goroutine...")
        }
    }()
    
    for i := 0; i < 4; i++ {
        //让出时间片，先让别的协议执行，它执行完，再回来执行此协程
        runtime.Gosched()
        fmt.Println("main..")
    }
}
```

#### Goexit的使用
```go
func main() {
    // 创建新建的协程
    go func() {
        fmt.Println("goroutine开始。。。")
        
        // 调用了别的函数
        fun()
        
        fmt.Println("goroutine结束。。。")
    }()
}

func fun() {
    defer fmt.Println("defer..")
    runtime.Goexit() //终止所在的协程
    
    fmt.Println("fun函数")
}
```