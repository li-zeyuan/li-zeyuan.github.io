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


三个角色

- Proposer（提案者）：多个；接受客户端请求，主动发起提案
- Acceptor（接受者｜投票者）：多个；被动接受提案消息，参与投票并返回结果给 Proposer,发送通知给 Learner
- Learner（学习者）：多个；被动接受 Acceptor 的通知，不参与投票，记录投票信息，并获取最终投票结果

<br>
两个阶段
- Prepare 阶段：Proposer 选择一个唯一的提案编号 n，向大多数 Acceptor 发送 Prepare 请求。 Acceptor 收到请求后，如果 n 大于自己响应的最大提案编号，回复响应并不会接受小于 n 编号提议。
- Accept 阶段：如果 Proposer 收到大多数 Acceptor 的响应，则会向 Acceptor 发送 Accept(n, value) 请求。Acceptor 收到请求后，如果 n 大于等于自己响应的最大提案编号，则接受该提案，并向 Learner 发送通知。

### Raft

### 参考
- [一文解读分布式一致性协议Paxos](https://www.infoq.cn/article/bmf7quks2rp887ghaza3)

## 二、弱（最终）一致性

用于提高 AP 架构的一致性

### quorum

### Gossip
