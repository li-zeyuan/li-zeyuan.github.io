---
title: slice与array
date: 2018-07-28 00:00:00 +0800
categories: [language, golang]
tags: [golang]
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

### 完整表达式
作用于 Array 和 Slice

```
a[low : high : max]
     |    |     |
     |    |     └── cap 的右边界（cap = max - low）
     |    └──────── len 的右边界（len = high - low）
     └───────────── 起始位置
```

```
original := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

//                           底层数组
//  index:  0  1  2  3  4  5  6  7  8  9

s1 := original[2:5]    // [2,3,4]  cap=8  可以看到 index 2~9
s2 := original[2:5:6]  // [2,3,4]  cap=4  只能看到 index 2~5
s3 := original[2:5:5]  // [2,3,4]  cap=3  只能看到 index 2~4

// s1 append 不扩容 → 覆盖 original[5]
// s3 append 必扩容 → 独立副本
```

为什么需要完整表达式？


防止子 slice/array 因为共享底层数组而污染数据。
