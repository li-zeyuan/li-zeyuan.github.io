---
title: Docker问题排查
date: 2024-03-29 00:00:00 +0800
categories: [root, docker]
tags: [docker,troubleshooting]
author: ahern
---

## 容器端口（本地/远程）不能正常访问
#### 复现
无

#### 原因
- 容器不正常运行
- 位开放端口
- iptables拦截
- 安全组问题

#### 解决
1. 检查容器是否正常启动，进入容器是否能正常telnet通端口
2. 检查端口是否绑定0.0.0.0
3. 查看input规则：iptables  -L  -n；增加input规则：iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
4. 检查安全组是否放行端口

## docker不能访问宿主机端口
#### 复现
无

#### 原因
- iptables拦截

#### 解决
- 查看input规则：iptables  -L  -n；增加input规则：iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
- 更换ip地址访问：https://javaforall.cn/171590.html

## docker磁盘占用满
#### 复现
无

#### 原因
- 服务输入大量日志到控制台

#### 解决
- 采用logging限制：https://docs.docker.com/compose/compose-file/compose-file-v3/#logging
- 清理磁盘空间：https://www.jb51.net/article/241905.htm

