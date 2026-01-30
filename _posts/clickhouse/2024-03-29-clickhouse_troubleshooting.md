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
- 写入分布式表的分发parts堆积

#### 解决
1、删除分布式表

https://blanklin030.github.io/2022/01/12/clickhouse-distribute-insert-problem/

2、[慎重]直接删除.bin文件
- 找到占用空间大目录: `cd /data1/clickhouse/store && du --block-size=MiB --max-depth=1 | sort -rn | head -10`
- 删除bin文件：`cd /data1/clickhouse/store/xxx/xxx && rm -rf ./*`

## ddl语句卡住，执行超时
#### 原因
- ck中的ddl语句是串行执行，一条ddl卡住，将阻塞其他ddl语句执行
#### 解决
https://opensource.actionsky.com/20220505-clickhouse/

