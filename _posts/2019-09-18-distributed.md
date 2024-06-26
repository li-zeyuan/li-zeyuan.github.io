---
title: 分布式
date: 2019-09-18 00:00:00 +0800
categories: [root, go]
tags: [go]
author: ahern
---

### CAP

```
CAP定理又叫布鲁尔定理，在分布式体统中，不可能同时满足CAP，只能选择CP或AP
```

- C（Consistency）：一致性；在不同节点读取到的数据是一样的，是最新的，强一致性。
- A（Availability）：可用性：非故障节点要求在合理的时间返回，返回的数据是正确的。
- P（Partition tolerance）：分区容错性：一个集群中，当一台机器出现问题，这个集群仍然可以正常工作。

### 分布式事务解决方案

#### 2PC

![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210306203256.png){:height="10%" width="50%"}

- 第一阶段：事务管理器要求涉及到的数据库预提交（precommit），数据库并反馈是可以提交
- 第二阶段：事务管理器要求涉及数据库提交，或者回滚

````
缺点
- 单点问题：事务管理器扮演者重要角色，如果宕机，将导致分布式事务不可用
- 同步阻塞：事务管理器在通知涉及数据库precommit后，处于阻塞状态，直到提交事务
- 数据不一致：若第二阶段，某数据库网络不可达，其他数据已经提交，这样就导致数据不一致问题
````

#### TCC

![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210306204938.png){:height="10%" width="50%"}

- Try阶段：尝试执行，检查节点系统，预留资源
- Confirm阶段：确认执行，仅对try阶段预留的资源进行操作
- Cancel阶段：释放try阶段预留的资源

#### 本地消息表

- 将分布式消息存储到数据表，由定时任务去轮询这张消息表，将消息消费掉，适合弱一致性的场景

#### MQ事务

![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210306210120.png){:height="10%" width="50%"}

- 1、生产者发送prepared消息
- 2、生产者执行本地事务
- 3、若本地事务执行成功，确认prepared消息
- 4、消费者消费消息

- 全局唯一
- 递增、不连续
- 高可用、高性能

### 分布式id

#### 数据库自增id

- 优点：简单，开发成本小
- 缺点：有序的递增不安全、分表分库麻烦

#### uuid

- 全局唯一
- 字符串导致性能下降
- bson.NewObject也是基于时间戳+机器码+序列号

#### 雪花算法

- ![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210203110553.png){:height="10%" width="50%"}

- 占位bit（1bit）+ 时间戳位（41bit）+ 机器码位（10bit）+ 序列号位（12bit）
- 生成64bit大小的整数
- 通过和上一个id对比时间戳，解决时钟回拨问题

#### 项目中的id

- 基于雪花算法，对雪花算法的位数做了调整，是18位的十进制整数

### 微服务划分原则
- 接口分离
    - 避免不同端复用接口
- 可部署性
    - 容器化
    - 网关化
    - 日志集中化，日志追踪
- 事件驱动，服务间调用
    - http、rpc、消息中间件
- 松耦合
    - 消息队列解耦
    - 一个服务一个库，数据隔离
- 单一职责
    - 根据业务边界划分服务职责
    - 根据性能问题划分服务
    - 避免过度划分

### 参考

- https://juejin.cn/post/6844903647197806605
