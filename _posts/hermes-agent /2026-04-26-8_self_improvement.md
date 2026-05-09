---
title: 八、自我提升
date: 2026-04-26 00:00:00 +0800
categories: [ai,hermes-agent]
tags: [ai,hermes-agent]
author: ahern
---

核心：运行时主动反思 + Curator定时归纳，总结经验（Memory）和技能（Skills）。

## 运行时主动反思
后台异步 Agent 读取一份 messages_snapshot，使用复盘 prompt 判断用户是否表达了长期偏好和可复用流程。

### 触发条件：

- 本轮有正常 final_response
- 没有被用户打断 not interrupted
- memory 或 skill 任一 nudge 达到阈值。即：Agent长时间没有对memory、skill 做更新操作

### Memory 沉淀
将记录到MEMORY.md 和 USER.md文件

### Skills 沉淀
将调用 skill_manage tools 记录到技能库


## Curator
Curator 是 Hermes 自我演进后的技能库管理定时任务，负责把 agent-created skills 持续去重、修补、归档，并给用户留出 dry-run、pin、backup、rollback 这些安全阀。

## 参考
- [Curator](https://hermes-agent.nousresearch.com/docs/user-guide/features/curator)
