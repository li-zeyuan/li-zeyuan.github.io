---
title: select、poll、epoll之间的区别
date: 2021-10-09 00:00:00 +0800
categories: [root, linux]
tags: [linux]
author: ahern
---

|            | select   | poll   | epoll                                                |
| ---------- | -------- | ------ | ---------------------------------------------------- |
| 数据结构   | 数组     | 链表   | 红黑树：存监听文件描述符<br />链表：存就绪文件描述符 |
| 获取fd方式 | 遍历     | 遍历   | 事件回调                                             |
| 时间复杂度 | O(n)     | O(n)   | O(1)                                                 |
| 数据拷贝   | 4次      | 4次    | 内存映射（mmap）3次                                  |
| 连接数     | 一般1024 | 无限制 | 无限制                                               |


参考:https://juejin.cn/post/6931543528971436046