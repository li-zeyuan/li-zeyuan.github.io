---
title: 大文件读写
date: 2024-03-29 00:00:00 +0800
categories: [root, go]
tags: [go]
author: ahern
---

大文件的读写操作要考虑到内存问题，一次性加载容易撑爆内存导致OOM

## 概念

- 批处理系统（离线系统）：系统接收一批数据，一段时间后（几分钟或几天）给用户返回结果。数据有明显的分界线，如一天、一周的数据量。一般用实时性要求不高的场景。
- 流处理系统（准实时系统）：实时性介于在线和批处理系统之间，系统接收到数据后，及时进行处理。数据量没有明显的分界点，该系统的延时也相对低一些

## read handle

- 分片处理：创建固定大小的buf

```go
func BufHandle(filePath string, handleFun func([]byte)) error {
      f, err := os.Open(filePath)
      if err != nil {
          return err
      }
      defer f.Close()

      buf := make([]byte, 4096)
      for {
          n, err := f.Read(buf)
          if err != nil && err != io.EOF {
              return err
          }
          if err == io.EOF {
              return nil
          }

          handleFun(buf[:n])
      }
  }
```

- 按行处理：读取一行，处理完成后读取下一行

```go
func LineHandle(filePath string, handle func([]byte)) error {
      f, err := os.Open(filePath)
      defer f.Close()
      if err != nil {
          return err
      }

      buf := bufio.NewReader(f)

      for {
          line, isPrefix, err := buf.ReadLine()
          if err != nil && err != io.EOF {
              return err
          }
          if err == io.EOF {
              return nil
          }
          // 行太长，buf溢出
          if isPrefix {
              return errors.New("line too long")
          }

          handle(line)
      }
  }
```

## write handle

```go
func BufWrite() error {
    filePath := "./test.txt"
    file, err := os.OpenFile(filePath, os.O_WRONLY|os.O_CREATE, 0666)
    if err != nil {
        return err
    }
    defer file.Close()

    write := bufio.NewWriter(file)
    for i := 0; i < 5; i++ {
        write.WriteString("http://c.biancheng.net/golang/ \n")
    }

    //Flush将缓存的文件真正写入到文件中
    write.Flush()

    return nil
}
```

## 总结

- 文件的读写，采用缓冲区（内存）的方式效率更高，因为这样可以避免频繁的磁盘IO
- write.Flush()是将缓冲区（内存）的数据写入磁盘

## 参考

- bufio库：https://golang.org/pkg/bufio/
- 详解golang中bufio包的实现原理：https://studygolang.com/articles/24343
