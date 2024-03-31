---
title: kubernetes故障排查
date: 2024-03-29 00:00:00 +0800
categories: [root, kubernetes]
tags: [kubernetes,troubleshooting]
author: ahern
---

## 多行文本格式化丢失问题
#### 原因
- 文本不要以空格结尾
- 不要换行前再带个空格
- 不要在文本中添加不可见特殊字符

#### 解决
https://kennylong.io/fix-yaml-multi-line-format/

## 手动暴露pod端口
#### 解决
kubectl port-forward pods/\<pod_name\> 3306:3306 -n \<namespace\> --address 127.0.0.1,\<node_ip\>

