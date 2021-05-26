---
layout: post
title: "Golang：bufio包"
date:  2020-05-22 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 包
---
### bufio包
这个包可以大幅提高文件读写的效率。

#### bufio包原理
bufio是通过缓冲来提高效率
io操作本身的效率并不低，低的是频繁的访问本地磁盘的文件。所以bufio就提供了缓冲区(分配一块内存)，读和写都先在缓冲区中，最后再读写文件，来降低访问本地磁盘的次数，从而提高效率。
简单的说就是，把文件读取进缓冲（内存）之后再读取的时候就可以避免文件系统的io 从而提高速度。同理，在进行写操作时，先把文件写入缓冲（内存），然后由缓冲写入文件系统。看完以上解释有人可能会表示困惑了，直接把 内容-件 和 内容-冲-件相比， 缓冲区好像没有起到作用嘛。其实缓冲区的设计是为了存储多次的写入，最后一口气把缓冲区内容写入文件。

![](/img/in-post/2020-05-22-golang-bufio-01.png)

#### bufio包
bufio包实现了有缓冲的I/O。它包装一个io.Reader或io.Writer接口对象，创建另一个也实现了该接口，且同时还提供了缓冲和一些文本I/O的帮助函数的对象。
bufio.Reader：

bufio.Reader 实现了如下接口： io.Reader io.WriterTo io.ByteScanner io.RuneScanner
```go
// NewReaderSize 将 rd 封装成一个带缓存的 bufio.Reader 对象，
// 缓存大小由 size 指定（如果小于 16 则会被设置为 16）。
// 如果 rd 的基类型就是有足够缓存的 bufio.Reader 类型，则直接将
// rd 转换为基类型返回。
func NewReaderSize(rd io.Reader, size int) *Reader

// NewReader 相当于 NewReaderSize(rd, 4096)
func NewReader(rd io.Reader) *Reader

// Peek 返回缓存的一个切片，该切片引用缓存中前 n 个字节的数据，
// 该操作不会将数据读出，只是引用，引用的数据在下一次读取操作之
// 前是有效的。如果切片长度小于 n，则返回一个错误信息说明原因。
// 如果 n 大于缓存的总大小，则返回 ErrBufferFull。
func (b *Reader) Peek(n int) ([]byte, error)

// Read 从 b 中读出数据到 p 中，返回读出的字节数和遇到的错误。
// 如果缓存不为空，则只能读出缓存中的数据，不会从底层 io.Reader
// 中提取数据，如果缓存为空，则：
// 1、len(p)  缓存大小，则跳过缓存，直接从底层 io.Reader 中读
// 出到 p 中。
// 2、len(p) < 缓存大小，则先将数据从底层 io.Reader 中读取到缓存
// 中，再从缓存读取到 p 中。
func (b *Reader) Read(p []byte) (n int, err error)

// Buffered 返回缓存中未读取的数据的长度。
func (b *Reader) Buffered() int

// ReadBytes 功能同 ReadSlice，只不过返回的是缓存的拷贝。
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)

// ReadString 功能同 ReadBytes，只不过返回的是字符串。
func (b *Reader) ReadString(delim byte) (line string, err error)

```
bufio.Writer：

bufio.Writer 实现了如下接口： io.Writer io.ReaderFrom io.ByteWriter

```go
// NewWriterSize 将 wr 封装成一个带缓存的 bufio.Writer 对象，
// 缓存大小由 size 指定（如果小于 4096 则会被设置为 4096）。
// 如果 wr 的基类型就是有足够缓存的 bufio.Writer 类型，则直接将
// wr 转换为基类型返回。
func NewWriterSize(wr io.Writer, size int) *Writer

// NewWriter 相当于 NewWriterSize(wr, 4096)
func NewWriter(wr io.Writer) *Writer

// WriteString 功能同 Write，只不过写入的是字符串
func (b *Writer) WriteString(s string) (int, error)

// WriteRune 向 b 写入 r 的 UTF-8 编码，返回 r 的编码长度。
func (b *Writer) WriteRune(r rune) (size int, err error)

// Flush 将缓存中的数据提交到底层的 io.Writer 中
func (b *Writer) Flush() error

// Available 返回缓存中未使用的空间的长度
func (b *Writer) Available() int

// Buffered 返回缓存中未提交的数据的长度
func (b *Writer) Buffered() int

// Reset 将 b 的底层 Writer 重新指定为 w，同时丢弃缓存中的所有数据，复位
// 所有标记和错误信息。相当于创建了一个新的 bufio.Writer。
func (b *Writer) Reset(w io.Writer)

```

#### 读取数据示例
```go
func main() {
    fileName := "/Users/ruby/Documents/pro/a/english.txt"
    file, err := os.Open(fileName)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer file.Close()
    
    // 创建Reader对象
    b1 := bufio.NewReader(file)
    // 1. Read()高效读取
    p := make([]byte, 1024)
    n1, err := b1.Read(p)
    fmt.Println(n1)
    fmt.Println(string(p[:n1]))
    
    // 2. ReadLine()
    data, flag, err := b1.ReadLine()
    fmt.Println(flag)
    fmt.Println(err) 
    fmt.Println(data)
    fmt.Println(string(data))
    
    // 3. ReadString() 
    
    for {
        s1, err := b1.ReadString('\n')
        if err == io.EOF {
            fmt.Println("读取完毕..")
            break
        }
        fmt.Println(s1)
    }
    // 4. ReadBytes()
    data, err := b1.ReadBytes('\n')
    fmt.Println(err)
    fmt.Println(string(data))
    
    // Scanner 
    s2 := ""
    fmt.Scanln(&s2)
    fmt.Println(s2)
    
    b2 := bufio.NewReader(os.Stdin)
    s2, _ := b2.ReadString('\n')
    fmt.Println(s2)
}
```

#### 写数据示例
```go
func main(){
    fileName := "/Users/ruby/Documents/pro/a/cc.txt"
    file, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY, os.ModePerm)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer file.Close()
    
    w1 := bufio.NewWriter(file)
    for i:=1; i<=1000; i++ {
        w1.WriteString(fmt.Sprintf("%d:hello", i))
    }
    w1.Flush()
}
```