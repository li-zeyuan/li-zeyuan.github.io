---
title: Prometheus系统架构和原理
date: 2026-02-14 00:00:00 +0800
categories: [middleware, prometheus]
tags: [prometheus]
author: ahern
---

## 整体架构
```
                 +----------------+
                 |   Grafana      |
                 +--------+-------+
                         |
                       PromQL
                         |
+---------+      +-------v--------+
| Exporter| ---> |  Prometheus    |
|         |      |  Server        |
+---------+      +-------+--------+
                         |
                    TSDB Storage
                         |
                    Alerting Rules
                         |
                    AlertManager
```
- Prometheus Server: 负责数据采集，处理存储和查询请求，触发告警。
- Exporter: 收集系统和应用指标数据，暴露 HTTP 接口
- Grafana: 可视化监控面板，使用 PromQL 查询指标数据
- TSDB Storage: Prometheus 内置的时序数据库
- AlertManager: 负责告警聚合，告警抑制和通知

## 时序数据库（TSDB）

数据结构：`metric_name{label1=v1,label2=v2} value timestamp`
- 时间压缩
- label索引


<br>
原理：
- WAL（Write Ahead Log）：机器故障恢复
- Head Block（内存）：最新 2 小时数据存在内存
- Block Files（磁盘）：每 2 小时生成一个 block 。
  ```
  block
  ├── chunks
  ├── index
  └── meta.json
  ```

## 数据采集流程
Prometheus 主动 pull 模式。
```
Prometheus ----HTTP----> Target
```

## 数据查询流程
```
PromQL
  ↓
Query Engine
  ↓
TSDB Index
  ↓
Chunk Data
  ↓
Result
```

## Prometheus与 K8s 结合
通过`kubernetes_sd_configs`, 自动生成 targets。

## 为什么 Prometheus 使用 Pull，而不是 Push
- 统一采集控制（监控中心可控）：Prometheus 可以自己决定抓取频率，采集策略
- 云原生场景，自动服务发现：使用kubernetes_sd_configs即可自动发现targets，避免 push 模式主动注册复杂性。
- 服务健康检查能力：pull 模式天然支持 up 状态监控，push 模式服务挂掉后无法上报监控
- 防止监控系统被压垮：防止大量 push 数据压垮系统


