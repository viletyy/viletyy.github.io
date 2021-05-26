---
layout: post
title: "Golang：流程语句"
date:  2020-05-15 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Golang
---

## 程序的流程结构

一共有三种：顺序结构，选择结构，循环结构

顺序结构：从上向下，逐行执行

选择结构：条件满足，某些代码才会执行。0-1次

分支语句：if, switch, select

循环结构：条件满足，某些代码会被反复的执行多次。0-N次

循环语句：for



### if分支语句

**语法格式**

```go
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
}

if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
} else {
  /* 在布尔表达式为 false 时执行 */
}

if 布尔表达式1 {
   /* 在布尔表达式1为 true 时执行 */
} else if 布尔表达式2{
   /* 在布尔表达式1为 false ,布尔表达式2为true时执行 */
} else{
   /* 在上面两个布尔表达式都为false时，执行*/
}
```

**示例代码**：

```go
package main

import "fmt"

func main() {
  /* 定义局部变量 */
  var a int = 10
  
  /* 使用 if 语句判断布尔表达式 */
  if a < 20 {
       /* 如果条件为 true 则执行以下语句 */
       fmt.Printf("a 小于 20\n" )
   }
   fmt.Printf("a 的值为 : %d\n", a)
}
```

### if变体

如果其中包含一个可选的语句组件（在评估条件之前执行）。它的语法是

```go
if statement; condition {  
}

if condition{

}
```

**示例代码**：

```go
package main

import (
	"fmt"
)

func main() {
  if num := 10; num % 2 == 0 {
    fmt.Println(num, "is even")
  } else {
    fmt.Println(num, "is odd")
  }
}
```

------

### switch分支语句

**语法格式**

```go
switch var1 {
  case val1:
  	...
  case val2:
  	...
  default:
  	...
}
```

**示例代码**

```go
package main

import "fmt"

func main() {
  /* 定义局部变量*/
  var grade string = "B"
  var marks int = 90
  
  switch marks {
    case 90: grade = "A"
    case 80: grade = "B"
    case 50,60,70 : grade = "C"
  	default: grade = "D"
  }
  
  switch {
    case grade == "A":
    	fmt.Printf("优秀！\n")
    case grade == "B":
    	fmt.Printf("良好 \n")
  	case grade == "D":
    	fmt.Printf("及格 \n")
    case grade == "F":
      fmt.Printf("不及格 \n")
    default:
      fmt.Printf("差 \n")
  }
  fmt.Printf("你的等级是 %s \n", grade)
}
```

### fallthrough

如需贯通后续的case，就添加fallthrough

```go
package main

import (
	"fmt"
)

type data [2]int 

func main() {
  switch x :=5; x {
    default: 
    	fmt.Println(x)
    case 5:
    	x += 10
      fmt.Println(x)
      fallthrough
    case 6:
    	x += 20
    	fmt.Println(x)
  }
}
```

输出结果

```go
15
35
```

------

### for循环语句

**语法结构**

```go
for init; condition; post { }

```

**示例代码**

```go
package main 

import (
	"fmt"
)

func main() {
  for i := 1; i <= 10; i++ {
    fmt.Printf(" %d", i)
  }
}

```

### for循环变体

**语法结构**

```go
for condition { }

```

**示例代码**

```go 
package main

import "fmt"

func main() {
  
  var b int = 15
  var a int 
  
  numbers := [6]int{1, 2, 3, 5}
  
  /* for循环 */
  for a := 0; a < 10; a++ {
    fmt.Printf("a的值为： %d\n", a)
  }
  
  for a < b {
    a++
    fmt.Printf("a的值为： %d\n", a)
  }
  
  for i, x := range numbers {
    fmt.Printf("第 %d 位x的值 = %d\n", i, x)
  }
}

```

------

### 跳出循环的语句

#### break语句

break：跳出循环体。break语句用于在结束其正常执行之前突然中止for循环

**示例代码**

```go
package main

import (
	"fmt"
)

func main() {
  for i := 1; i <= 10; i++ {
    if i > 5 {
      break
    }
    fmt.Printf("%d", i)
  }
  fmt.Printf("\nline after for loop")
}
```

#### continue语句

continue: 跳出一次循环。continue语句用于跳过for循环的当前迭代。在continue语句后面的for循环中的所有代码将不会在当前迭代中执行。循环将继续到下一个迭代

```go 
package main

import (
	"fmt"
)

func main() {
  for i := 1; i <= 10; i++ {
    if i%2 == 0 {
      continue
    }
    fmt.Printf("%d ", i)
  }
}
```

------

### goto语句

goto: 可以无条件的转移到过程中指定的行

**语法结构**

```go 
goto label;
..
..
label: statement;
```

**代码示例**

```go
package main

import "fmt"

func main() {
  /* 定义局部变量*/
  var a int = 10
  
  /* 循环 */
  LOOP: for a < 20 {
    if a == 15 {
      /* 跳过迭代 */
      a = a + 1
      goto LOOP
    }
    fmt.Printf("a的值为：%d\n", a)
  	a++
  }
}
```