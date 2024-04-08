---
title: RabbitMQ、Redis消息队列对比
date: 2019-01-25 00:00:00 +0800
categories: [root, redis]
tags: [redis, rabbit_mq]
author: ahern
---

转自：https://www.cnblogs.com/kevincaptain/p/5876070.html

## RabbitMQ

用于分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然

## Redis
NoSQL存储系统，可以当做轻量级队列服务来使用

------

## 对比
可靠消费
  - Redis：没有相应的可靠消费保证机制，需要手动处理
  - rabbitMQ：具有消息消费确认，即使消费失败，也会自动返回原队列，同时可全程持久化，保证消息被正确消费

可靠发布
  - Redis：不提供，需自行实现
  - rabbitMQ：具有发布确认功能，保证消息被发布到服务器

高可用
  - Redis：采用主从模式，读写分离，但是故障转移还没有非常完善的官方解决方案
  - rabbitMQ：集群采用磁盘、内存节点，任意单点故障都不会影响整个队列的操作

持久化
  - Redis：将整个Redis实例持久化到磁盘
  - rabbitMQ：队列，消息，都可以可以选择是否持久化

消费者负载均衡
  - Redis：不提供，需自行实现
  - rabbitMQ：根据消费情况，进行消息的均衡发布

队列监控
  - Redis：不提供，需自行实现
  - rabbitMQ：后台可以监控某个队列的所有消息

流量控制
  - Redis：不提供，需自行实现
  - rabbitMQ：服务器过载情况，对生产者速率会进行限制，保证服务可靠性

出入队性能
  - rabbitMQ性能更好一些
