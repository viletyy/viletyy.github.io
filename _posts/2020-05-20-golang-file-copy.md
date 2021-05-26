---
layout: post
title: "Golang：实现文件复制"
date:  2020-05-20 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
---
### 文件复制
在io包中主要是操作流的一些方法，今天主要学习一下。
它的原理都是通过程序，从源文件读取文件中的数据，在写出到目标文件里

![](/img/in-post/2020-05-20-golang-file-copy-01.png)


#### io包下的Read()和Write()方法实现
通过io包下的Read()和Write()方法，边读边写，就能实现复制。这个方法是按块读取文件，块的大小也会影响到程序的性能
```go
func File1(srcFile, destFile string) (int, error) {
    file1, err := os.Open(srcFile)
    if err != nil {
        return 0, err 
    }
    file2, err := os.OpenFile(destFile, os.O_WRONLY|os.O_CREATE, os.Modeperm)
    if err != nil {
        return 0, err 
    }
    defer file1.Close()
    defer file2.Close()
    // 拷贝数据
    bs := make([]byte, 1024, 1024)
    n := -1 
    total := 0
    for {
        n, err = file1.Read(bs)
        if err == io.EOF || n == 0 {
            fmt.Println("拷贝完毕。。")
        } else if err != nil {
            fmt.Println("报错了。。")
        }
        total += n
        file2.Write(bs[:n])
    }
    return total, nil 
}
```

#### io包下的copy()方法实现
```go
func File2(srcFile, destFile string) (int64, error) {
    file1, err := os.Open(srcFile)
    if err != nil {
        return 0, err 
    }
    file2, err := os.OpenFile(destFile, os.O_WRONLY|os.O_CREATE, os.ModePerm)
    if err != nil {
        return 0, err 
    }
    defer file1.Close()
    defer file2.Close()
    
    return io.Copy(file2, file1) 
}
```

#### ioutil包中的WriteFile()和ReadFile()实现
但由于使用一次性读取文件，再一次性写入文件的方式，所以该方法不适用于大文件，容易内存溢出。
```go 
func File3(srcFile, destFile string) (int, error) {
    input, err := ioutil.ReadFile(srcFile)
    if err != nil {
        fmt.Println(err)   
        return 0, err
    }
    
    err = ioutil.WriteFile(destFile, input, 0644)
    if err != nil {
        fmt.Println("操作失败：", destFile)
        fmt.Println(err)
        return 0, err 
    }
    
    return len(input), nil
}
```