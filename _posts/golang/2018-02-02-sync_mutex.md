---
title: sync.Mutex / RWMute原理
date: 2018-02-02 00:00:00 +0800
categories: [language, golang]
tags: [golang]
author: ahern
---

## sync.Mutex
Go 的 sync.Mutex 底层主要由 state + semaphore 两个字段组成。
```go
type Mutex struct {
  state int32    // 状态
  sema  uint32   // 信号量
}
```

![Snipaste_2022-02-17_14-59-04](https://raw.githubusercontent.com/li-zeyuan/access/master/img/Snipaste_2022-02-17_14-59-04.png)

state位：
  - waiter_num：等待goroutine
  - starving：是否处于饥饿状态
  - woken：是否有goroutine正在加锁
  - locked：是否有goroutine持有该锁

Mutex.sema
  - 信号量，用于唤醒阻塞的goroutine


<br>
加锁时首先通过 CAS（Compare and Swap，比较并交换） 尝试抢锁，如果失败会进行 短暂自旋，自旋仍失败则通过 信号量机制将 goroutine 挂起并加入等待队列。

解锁时会修改 state 状态，并通过信号量机制唤醒等待的 goroutine。



<br>
锁的两种模式

```
引入饥饿模式，保证锁公平性
公平性：goroutine获取锁的顺序与请求锁的顺序一致
```

- 正常模式
  - 唤醒阻塞队列队头的goroutine，该goroutine与新请求锁的goroutine共同竞争锁，新请求的goroutine大概率会获得锁，因为占有cpu时间片
  
- 饥饿模式
  - 唤醒阻塞队列队头的goroutine直接获得锁
  - 新请求锁goroutine不参与锁竞争
  
  ```
  饥饿模式的触发条件：
  	1、有一个goroutine获取锁的时间超过1ms
  饥饿模式的取消条件：
  	1、获取到锁的goroutine为阻塞队列的最后一个时，恢复正常模式
  	2、获取到锁的goroutine等待时间小于1ms，恢复正常模式
  ```

## sync.RWMute
```go
type RWMutex struct {
  w           Mutex   // 写锁（本质还是 Mutex）
  readerCount int32   // 当前读者数量
  readerWait  int32   // 等待释放的读者数
  writerSem   uint32  // 写等待信号量
  readerSem   uint32  // 读等待信号量
}
```

RWMute是通过Mutex + 计数器 + 读/写信号量实现的.



如何解决写饥饿问题？
- 抢占写锁 -> 阻止新读锁加锁 -> 等待已有读锁结束


## 什么情况下使用RWMute，什么情况下使用Mutex
- RWMute：适用读多写少场景。
- Mutex：适用低并发，读写平均场景。
  > 因为RWMute频繁加写锁开销比Mutex大，RWMute的写锁需要解决写饥饿。

## Panic场景

> 注意：重复加锁不是Panic，是死锁

- 解锁未加锁的 Mutex
```
func main() {
    var mu sync.Mutex
    mu.Unlock() // fatal error: sync: unlock of unlocked mutex
}
```

- 重复解锁
```go
func main() {
    var mu sync.Mutex
    mu.Lock()
    mu.Unlock()
    mu.Unlock() // panic: sync: unlock of unlocked mutex
}
```

- RWMutex 未加读/写锁就解
```go
func main() {
    // 场景 A：读锁未加就解
    rw.RUnlock() // panic: sync: RUnlock of unlocked RWMutex

    // 场景 B：写锁未加就解
    rw.Unlock()  // panic: sync: unlock of unlocked mutex
}
```
