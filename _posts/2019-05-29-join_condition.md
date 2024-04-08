---
title: join接条件and、where区别
date: 2019-05-29 00:00:00 +0800
categories: [root, mysql]
tags: [mysql]
author: ahern
---

参考：https://blog.csdn.net/xingfeng0501/article/details/7816703

## 1、left join on and

on后面接生成临时表的条件，**保全左表**的记录，右边不符合条件的为null

## 2、left join on where

按照on条件连表后，用where条件过滤

## 例子：

```
table1：
id	size
1	10
2	20
3	30

table2：
10	a
20	b
20	c
```

#### sql

```sql
-- left join on where
select * from table1 left join table2 on table1.size=table2.size where table2.name='a'

中间表：
table1.id	table1.size		table2.size		table2.name
1			10				10				a
2			20				20				b
2			20				20				c
3			30				null			null

result:
table1.id	table1.size		table2.size		table2.name
1			10				10				a

-- left join on and
select * from table1 left join table2 on (table1.size=table2.size and table2.name='a')

result（中间表）:
table1.id	table1.size		table2.size		table2.name
1			10				10				a
2			20				null			null
3			30				null			null
```
