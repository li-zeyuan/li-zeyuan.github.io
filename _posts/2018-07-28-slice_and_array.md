---
title: slice与array
date: 2018-07-28 00:00:00 +0800
categories: [root, go]
tags: [go]
author: ahern
---

### slice and array

- 相同点

  - len()获取长度，通过下表获取
  - 分配一块连续的内存空间

- array

  - 值类型
  - var，:= 创建，
  - 不可用make（运行时）、**append**、**copy**
  - 创建后长度、容量不可改变

- slice  

  ```go
  // slice 源代码表示
  type SliceHeader struct {
  	Data uintptr // 指向底层数组
  	Len  int
  	Cap  int
  }
  ```

  - 引用类型
  - var，:= 、make创建
  - 可扩充，1024个元素内2倍增长，往后1/4增长
  - 底层实现是指向一个array，容量为底层array的大小

### 使用场景

  - 一般使用slice
  - 不确定len用 `slice`，确定大小使用`array`。

### 完整表达式：a[low:high:max]
- 完整切片表达式：http://www.jalen-qian.com/p/go%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80%E4%B9%8B%E5%88%87%E7%89%87/
