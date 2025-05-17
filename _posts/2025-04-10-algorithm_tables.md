---
title: 一、数据结构和算法-表
date: 2020-06-17 00:00:00 +0800
categories: [root, algorithm]
tags: [algorithm]
author: ahern
---

### 跳表
https://juejin.cn/post/6844903955831619597
  - 优点：
    - 更新时，局部性更好
    - 支持范围查询
  - 缺点：
    - 占空间
  - 应用场景：
    - 内存多级页表
    - redis key管理
    - redis zset类型
  - 时间复杂度：近似二分查找，等于 索引高度：logn * 每层索引遍历元素的个数 = logn
  - 空间复杂度：n/2（一级索引） + n/4（二级索引） + n/8（三级索引） + … + 8 + 4 + 2 = n-2，空间复杂度是 O(n)
  ```
  redis中为啥用跳表而不用红黑树
  1、更新时，跳表需要更新的范围更小，竞争锁的开销更小，并发高
  https://blog.csdn.net/qq9808/article/details/104865385
  ```

### 哈希表
  - 优点：
    - 时间复杂度O(1)
  - 缺点：
    - 基于数组，扩充成本高，大数据量性能下降严重
    - 存在hash冲突
    - 不支持范围查询
  - 应用场景：
    - map结构


### 参考：

- https://jishuin.proginn.com/p/763bfbd2febb
