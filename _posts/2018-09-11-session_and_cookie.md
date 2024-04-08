---
title: session和cookie
date: 2018-09-11 00:00:00 +0800
categories: [root, go]
tags: [go]
author: ahern
---

### cookie

- 保存在客户端
- 安全性差
- 场景：可用于保存表单信息、购物车信息等

### session

- 保存在服务端，服务端生成session_id给客户端返回
- session_id一般借助cookie来保存
- 场景：用户权限认证
- 大量的session会占用服务器资源
