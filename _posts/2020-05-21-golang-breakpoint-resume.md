---
layout: post
title: "Golang：实现断点续传"
date:  2020-05-21 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
---
### 断点续传
#### Seeker接口
```go 
type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}
```
第一个参数：偏移量，
第二个参数： 
    0：seekStart表示相对于文件开始
    1：seekCurrent表示相对于当前偏移量
    2：seekend表示相对于文件结尾
```go
const (
    SeekStart = 0 
    SeekCurrent = 1
    SeekEnd = 2
)
```

#### 示例
```go
func main() {
    srcFile := "/Users/ruby/Documents/pro/a/guliang.jpeg"
    destFile := "guliang4.jpeg"
    tempFile := destFile + "temp.txt"
    file1, _ := os.Open(srcFile) 
    file2, _ := os.OpenFile(destFile, os.O_CREATE|os.O_WRONLY, os.ModePerm)
    file3, _ := os.OpenFile(tempFile, os.O_CREATE|os.O_RDWR, os.ModePerm)
    
    defer file1.Close() 
    defer file2.Close()
    
    file3.Seek(0, io.SeekStart)
    bs := make([]byte, 100, 100)
    n1, err := file3.Read(bs)
    fmt.Println(n1)
    countStr := string(bs[:n1])
    fmt.Println(countStr)
    count, _ := strconv.ParseInt(countStr, 10, 64)
    fmt.Println(count)
    
    // 设置读，写的偏移量
    file1.Seek(count, 0)
    file2.Seek(count, 0)
    data := make([]byte, 1024, 1024)
    n2 := -1 
    n3 := -1 
    total := int(count)
    
    for{
        n2, err  = file1.Read(data)
        if err == io.EOF {
            fmt.Println("文件复制完毕...")
            file3.Close()
            os.Remove(tempFile)
            break
        }
        // 将数据写入到目标文件
        n3, _ = file2.Write(data[:n2])
        total += n3 
        file3.Seek(0, io.SeekStart)
        file3.WriteString(strconv.Itoa(total))
    }
}
```