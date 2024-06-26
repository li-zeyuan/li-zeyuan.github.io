---
title: 定时任务的实现方案
date: 2022-08-18 00:00:00 +0800
categories: [root, linux]
tags: [scheduler]
author: ahern
---

### 1、PriorityBlockingQueue + Polling

- PriorityBlockingQueue 为优先队列

- 生产者随机往队列中发送消息

- 消费者轮询获取消息并消费

  ```
  缺点：轮询的时间间隔不好控制，时间间隔太长，任务无法及时处理，间隔太短，消耗CPU
  ```

### PriorityBlockingQueue + 时间差

- 生产者往队列中发送消息

- 消费者从队列尾部取出消息，计算时间差，然后sleep()

  ```
  缺点：若队尾的元素的sleep时间过长，而队列中的消息的执行时间短，会导致队中消息不可被执行
  ```

### DelayQueue（延时消息）

- 消费者向消息队列发送带有延时时间的消息

### 时间轮(HashedWheelTimer)

![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210315164845.png)

- 单位时间后，指针指向环形队列的下一个元素
- 指针指向的位置，取出该位置的元素集合并执行

