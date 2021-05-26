---
layout: post
title: "Golang：ioutil包"
date:  2020-05-23 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 包
---
### ioutil包
除了io包可以读写数据，go还提供了一个辅助的工具包就是ioutil。
```go
import "io/ioutil"
```
该包的介绍只有一句话：Package ioutil implements some I/O utility functions.

#### ioutil包的方法
```go
// Discard 是一个io.Writer 接口，调用它的Write方法将不做任何事情
// 并且始终成功返回
var Discard io.Writer = devNull(0)

// ReadAll读取r中的所有数据，返回读取的数据和遇到的错误
// 如果读取成功，则err返回nil，而不是EOF，因为ReadAll定义为读取
// 所有数据，所以不会把EOF当作错误处理
func ReadAll(r io.Reader) ([]byte, error)

// ReadFile读取文件中的所有数据，返回读取的数据和遇到的错误
// 如果读取成功，则err返回nil，而不是EOF
func ReadFile(filename string) ([]byte, error)

// WriteFile向文件中写入数据，写入前会清空文件。
// 如果文件不存在，则会以指定的权限创建文件
// 返回遇到的错误
func WriteFile(filename string, data []byte, perm os.FileMode) error 

// ReadDir 读取指定目录中的所有目录和文件（不包括子目录）
// 返回读取到文件信息列表和遇到错误，列表是经过排序的。
func ReadDir(dirname string) ([]os.FileInfo, error)

// NopCloser 将r包装成一个ReadCloser类型，但Close方法不做任何事情
func NopCloser(r io.Reader) io.ReadCloser

// TempFile在dir目录中创建一个以prefix为前缀的临时文件，并将其以读写模式打开。返回创建的文件对象和遇到的错误
// 如果dir为空，则在默认的临时目录中创建文件（参加os.TempDir)，多次调用会创建不通的临时文件，调用者可以通过f.Name()获取文件的完整路径
// 调用本函数所创建的临时文件，应该由调用者字节删除
func TempFile(dir, prefix string) (f *os.File, err error)

// TempDir功能通TempFile，只不过创建的是目录，返回目录的完整路径
func TempDir(dir, prefix string) (name string, err error)
```

#### 示例
```go
package main 
import (
    "io/ioutil"
    "fmt"
    "os"
)

func main() {
    //1.读取文件中的所有数据
    fileName1 := "/Users/ruby/Documents/pro/a/aa.txt"
    data, err := ioutil.ReadFile(fileName1)
    fmt.Println(err)
    fmt.Println(string(data))
    
    //2.写出数据
    fileName2 := "/Users/ruby/Documents/pro/a/bbb.txt"
    s1 := "hello world"
    err := ioutil.WriteFile(fileName2, []byte(s1), 0777)
    fmt.Println(err)
    
    // 3.读取所有数据
    s2 := "qwertyuiop"
    r1 := strings.NewReader(s2)
    data, _ := ioutil.ReadAll(r1)
    fmt.Println(data)
    
    // 4.读取一个目录下子内容：子文件和子目录，但是仅有一层
    dirName := "/Users/ruby/Documents/pro/a"
    fileInfos, _ := ioutil.ReadDir(dirName)
    fmt.Println(len(fileInfos))
    for
}
```