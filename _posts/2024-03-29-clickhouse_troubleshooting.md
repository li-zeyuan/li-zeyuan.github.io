---
title: Clickhouse故障排查
date: 2024-03-29 00:00:00 +0800
categories: [root, clickhouse]
tags: [clickhouse,troubleshooting]
author: ahern
---

## 写入时间字段（不带时区）时区问题

#### 复现
1. insert 语句没有带时区,时间字段的值不会发生变化
```sql
insert into trace_names_all
values ('2022-07-15 00:00:08',
'test_service_name',
'test_span_name');
```

2. 修改clickhouse-server时区：UTC -> UTC+8
- 修改前：时区为UTC时插入的数据：
```sql
insert into trace_names_all
values ('2022-07-15 00:00:00',
'test_service_name',
'test_span_name');
```
- **修改时区后2022-07-15 00:00:00.000 变成 2022-07-15 08:00:00.000，加8小时**
┌───────────────timestamp─┬─serviceName───────┬─spanName───────┐
│ 2022-07-15 08:00:00.000 │ test_service_name │ test_span_name │
└─────────────────────────┴───────────────────┴────────────────┘
- 修改后：时区为UTC+8插入的数据：
```sql
insert into trace_names_all
values ('2022-07-15 00:00:08',
'test_service_name',
'test_span_name');
```
┌───────────────timestamp─┬─serviceName───────┬─spanName───────┐
│ 2022-07-15 00:00:08.000 │ test_service_name │ test_span_name │
└─────────────────────────┴───────────────────┴────────────────┘

#### 原因
插入时间字段时（不带时区），ch默认该字段时区为ch的时区。

#### 解决
无

## 分布式表写堆积问题
#### 原因
- 某个节点故障

#### 解决
- https://blanklin030.github.io/2022/01/12/clickhouse-distribute-insert-problem/

