---
title: 常见一致性协议
date: 2025-04-10 00:00:00 +0800
categories: [system-design, distributed]
tags: [distributed]
author: ahern
---

## 一、强一致性

保证 CP 架构的一致性

### Paxos

Paxos 是指分布式系统中，针对某个值通过提案和投票，多数派达成一致的算法（协议）。
![img.png](./assets/images/img_63.webp){:height="70%" width="70%"}

> 参考：[一文解读分布式一致性协议Paxos](https://www.infoq.cn/article/bmf7quks2rp887ghaza3)

三个角色

- Proposer（提案者）：多个；接受客户端请求，主动发起提案
- Acceptor（接受者｜投票者）：多个；被动接受提案消息，参与投票并返回结果给 Proposer,发送通知给 Learner
- Learner（学习者）：多个；被动接受 Acceptor 的通知，不参与投票，记录投票信息，并获取最终投票结果

<br>
两个阶段
- Prepare (提案) 阶段：Proposer 选择一个唯一的提案编号 n，向大多数 Acceptor 发送 Prepare 请求。 Acceptor 收到请求后，如果 n 大于自己响应的最大提案编号，回复响应并不会接受小于 n 编号提议。
- Accept（接受）阶段：如果 Proposer 收到大多数 Acceptor 的响应，则会向 Acceptor 发送 Accept(n, value) 请求。Acceptor 收到请求后，如果 n 大于等于自己响应的最大提案编号，则接受该提案，并向 Learner 发送通知。

### Raft

是一种应用广泛的强一致性、去中心化、高可用分布式共识算法。

> 参考：[共识算法 Raft](https://www.thebyte.com.cn/distributed-system/raft.html)

三个角色

- Leader（领导者）：任何时刻，集群中唯一，接受客户端请求，并将请求转换为日志条目，复制到 Follower。
- Follower（跟随者）：接受来自 Leader 的日志条目。当 Leader 出现故障时，Follower 将变成 Candidate，发起选举。
- Candidate（候选者）：Leader 选举过程中临时角色，向其他节点发起投票请求，获取多数选票将成为新的 Leader。

<br>
Leader 选举

Leader 与 Follower 之间通过心跳机制保持联系。如果 Follower 在一定时间内没有收到 Leader 的心跳，它会变成 Candidate 并发起选举。Candidate 向其他节点发送请求投票的消息，获取多数选票后成为新的 Leader。

<br>
日志同步

Leader 接受客户端请求后，将请求转换为日志条目，并将其复制到所有 Follower。当多数 Follower 确认收到日志条目后，Leader 才向客户端返回成功响应。

<br>
安全性

成为 Leader 需要多数票，已经提交的日志条目一定存在多数派中，多数派必有交集。保证了新的 Leader 一定包含最新的日志条目。
要求节点数为奇数，最多容忍 (N-1)/2 个节点故障。

### ZAB

todo

### quorum

是一种用于分布式系统中实现一致性的机制，核心思想基于多数派原则，即读写操作必须获取多数节点确认，才能返回成功。

工作原理：

- 写操作：数据写入至少需要多数节点确认，写入操作才 ACK
- 读操作：数据读取至少需要多数节点确认，读取操作才 ACK

## 二、弱（最终）一致性

适用于实现 AP 架构最终一致性

### Gossip

Gossip 是一种通过“随机节点间周期性交换信息”来实现全网最终一致的数据传播算法。

> O(log N) 轮就可以传播到全网。

### 三、对比

| 特性      | Gossip协议                 | Quorum 协议          | RAFT 协议            | Paxos 协议                 |
| ------- | ------------------------ | ------------------ | ------------------ | ------------------------ |
| 类型      | 弱一致性协议框架                 | 一致性协议框架            | 共识算法               | 共识算法                     |
| 操作机制    | 随机节点间周期性交换信息             | 多数副本确认             | 领导者选举 + 日志复制       | 两阶段投票                    |
| 一致性保证   | 弱（最终一致性）                 | 高                  | 强一致性               | 强一致性                     |
| 最大容错能力  | -                        | (N-1)/2            | (N-1)/2            | (N-1)/2                  |
| 易用性/可读性 | 较易理解                     | 较易理解               | 易于实现与理解            | 理论复杂，难以实现                |
| 应用示例    | Redis Cluster, Cassandra | MongoDB, Cassandra | etcd, Consul, TiKV | Chubby, Google Spanner 等 |
