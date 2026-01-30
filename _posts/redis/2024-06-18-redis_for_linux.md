---
title: Linux源码安装Redis
date: 2024-06-18 00:00:00 +0800
categories: [middleware, redis]
tags: [redis]
author: ahern
---

## 安装环境

- 一台机器上部署两个实例
  ```
  redis.conf文件务必区别：

  dir：数据文件
  logfile：日志文件
  dbfilename：转储文件
  pidfile：进程ID文件
  ```
- Ubuntu 22.04 LTS
- redis版本：7.0.0

## 安装dev

> 源码目录：`/opt/redis-7.0.0/`
>
> 可执行文件目录：`/opt/redis-7.0.0/bin/`
>
> 配置文件：`/opt/redis-7.0.0/redis-dev/redis.conf`
> 
> service文件：`/usr/lib/systemd/system/redis-dev.service`



1. 下载安装包：`cd /opt && wget http://download.redis.io/releases/redis-7.0.0.tar.gz`

2. 解压：`tar -xzf redis-7.0.0.tar.gz`

3. 编译：`cd /opt/redis-7.0.0 && make && make test`

4. 指定安装目录：`make install PREFIX=/opt/redis-7.0.0`

5. 创建配置文件：`cp /opt/redis-7.0.0/redis.conf /opt/redis-7.0.0/redis-dev/redis.conf`; 并修改
   
   ```
   bind * -::*
   requirepass xxx     # 密码
   dir /opt/redis-7.0.0/redis-dev
   logfile "/opt/redis-7.0.0/redis-dev/log.log"
   dbfilename dump6379.rdb
   pidfile /opt/redis-7.0.0/redis-dev/redis.pid
   ```

6. 创建service文件：`vim /usr/lib/systemd/system/redis-dev.service`
   
   ```
   [Unit]
   Description=sweetpotato teams redis-dev
   After=network.target
   
   [Service]
   Type=simple
   ExecStart=/opt/redis-7.0.0/bin/redis-server /opt/redis-7.0.0/redis-dev/redis.conf
   ExecReload=/opt/redis-7.0.0/bin/redis-server -s reload
   ExecStop=/opt/redis-7.0.0/bin/redis-server -s stop
   Restart=always
   RestartSec=30s
   
   [Install]
   WantedBy=multi-user.target
   ```

7. 启动redis：`systemctl start redis-dev.service && systemctl enable reids-$env.service`

## 安装pro
从以上第5开始

## 故障排除
