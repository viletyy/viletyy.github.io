---
layout: post
title: "Golang：键盘输入和打印输入"
date:  2020-05-11 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Golang
---

### 打印输入

- fmt包常用打印函数

  ` func Print(a ...interface{})(n int, err error) `

  `func Printf(format string, a ...interface{})(n int, err error) `

  `func Println(a ...interface{})(n int, err error)`b

- 常用占位符

  - %v, 原样输出
  - %T, 打印类型
  - %t, bool类型
  - %s, 字符串
  - %f, 浮点
  - %d, 10进制的整数
  - %b, 2进制的整数
  - %o,, 8进制
  - %x, %X, 16进制
    - %x: 0-9, a-f
    - %X: 0-9, A-F
  - %c, 打印字符
  - %p, 打印地址

- 示例代码

```go
package main

import (
	"fmt"
)

func main() {
  a := 100
  b := 3.14
  c := true
  d := "Hello World"
  e := Ruby
  f := 'A'
  fmt.Printf("%T, %b\n", a, a)
  fmt.Printf("%T, %f\n", b, b)
  fmt.Printf("%T, %t\n", c, c)
  fmt.Printf("%T, %s\n", d, d)
  fmt.Printf("%T, %s\n", e, e)
  fmt.Printf("%T, %d, %c\n", f, f, f)
  fmt.Println("-----------------------")
  fmt.Printf("%v\n", a)
  fmt.Printf("%v\n", b)
  fmt.Printf("%v\n", c)
  fmt.Printf("%v\n", d)
  fmt.Printf("%v\n", e)
  fmt.Printf("%v\n", f)
}
```

运行结果

![](/img/in-post/2020-05-11-golang-keyboard-input-output-01.png)

### 键盘输入

#### fmt包读取键盘输入

- fmt包读取键盘输入，常用方法如下

  `func Scan(a ...interface{})(n int, err error)`

  `func Scanf(format string, a ...interface{})(n int, err error)`

  `func Scanln(a ...interface{})(n int, err error)`

- 示例代码

```go
package main

import (
	"fmt"
)

func main() {
  var x int
  var y float64
  fmt.Println("请输入一个整数，一个浮点类型：")
  fmt.Scanln(&x, &y)
  fmt.Printf("x的数值：%d, y的数值：%f\n", x, y)
  
  fmt.Scanf("%d, %f", &x, &y)
  fmt.Printf("x:%d, y:%f\n", x,y)
}
```

#### bufio包读取

- 示例代码

```go
package main

import (
	"fmt"
  "os"
  "bufio"
)

func main() {
  fmt.Println("请输入一个字符串：")
  reader := bufio.NewReader(os.Stdin)
  s1, _ := reader.ReadString("\n")
  fmt.Println("读到的数据：", s1)
}
```
