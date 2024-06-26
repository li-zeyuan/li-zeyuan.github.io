---
title: 分布式锁
date: 2019-09-12 00:00:00 +0800
categories: [root, go]
tags: [go]
author: ahern
---

## Set [key] [value] ex [xx] nx

- 简单业务中使用
- 存在锁续期问题

## 单机Redis实现：Redisson

- 概述

  - 基于lua脚本，原子性
  - Redis官方推荐

- 加锁机制

  - 判断是否存在锁
  - 若无则`hincrby myLock 285475da-9152-4c83-822a-67ee2f116a79:52 1`
  - 若存在锁，则进入互斥机制

- 锁互斥机制

  - 客户端获取锁不成功，则会返回锁的TTL
  - 通过Redis的发布订阅机制订阅锁释放
  - 超过最大等待时间则返回获取锁失败

- watch dog机制

  - 默认加锁时间为30秒则启用watch Dog

  ```
  watch dog原理：
  成功启动watch dog的线程，会将线程id放到一个列表中，定时判断线程是否存活，然后给锁续期
  ```

- 可重入机制

  - ```
    127.0.0.1:6379> HGETALL myLock
    1) "285475da-9152-4c83-822a-67ee2f116a79:52"
    2) "2"
    ```

  - value值加1

- 锁释放机制

  - 删除锁
  - 发送释放锁消息
  - 删除watch dog

- 如何防死锁

  - 给key设置过期时间

- 如何保证加锁解锁是同一个线程

  - 指定一个key为锁的标记，值为线程id

- 锁续期问题

  - 通过watch dog

- 优点

  - 性能优
  - 可重入
  - 支持续期（watch dog）
  - Redis的订阅发布，优化等待锁线程流程

- 缺点

  - Redis Master-Slave 架构，master宕机后，锁没有同步到slave，则会出现多个客户端同时加锁

    ```
    解决：
    1、选举成master后，在TTL后才提供加锁服务
    2、用Redlock
    ```


## 多机Redis实现：Redlock

![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210324200845.png){:height="10%" width="50%"}

- redlock算法流程
  - 1、获取当前时间戳
  - 2、依次重各个master获取锁，获取锁的超时时间远小于TTL
  - 3、获取到n/2+1个锁后才算是获取锁成功
  - 4、锁的实际有效时间：TTL-获取所有用时-时钟偏移
  - 5、若中途获取锁失败，则需要删除已经获取成功的锁
- 如何防锁
  - 设置过期时间
- 如何保证加锁解锁是同一个线程
  - 指定一个key为锁的标记，值为线程id
- 锁续期问题
  - 结合redisson
- 优点
- 缺点

## 参考

- redisson：https://zhuanlan.zhihu.com/p/135864820
- redlock：https://www.cnblogs.com/rgcLOVEyaya/p/RGC_LOVE_YAYA_1003days.html
