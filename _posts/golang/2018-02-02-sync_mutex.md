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

加锁时首先通过 CAS（Compare and Swap，比较并交换） 尝试抢锁，如果失败会进行 短暂自旋，自旋仍失败则通过 信号量机制将 goroutine 挂起并加入等待队列。

解锁时会修改 state 状态，并通过信号量机制唤醒等待的 goroutine。

为了防止锁饥饿，Go 在 1.9 之后引入了 正常(公平) 和 饥饿 两种模式，在等待时间超过 1ms 时进入饥饿模式，锁会直接交给等待队列第一个 goroutine。

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


