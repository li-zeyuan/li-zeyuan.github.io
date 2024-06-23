---
title: Linux源码安装Nginx
date: 2024-06-18 00:00:00 +0800
categories: [root, middleware]
tags: [nginx]
author: ahern
---

## 安装环境

- Ubuntu 22.04 LTS
- nginx版本：1.25.2

## 安装步骤

> 源码目录：`/opt/nginx-1.25.2/`
>
> 安装目录：`/opt/nginx-1.25.2/nginx`
>
> logs文件：`/opt/nginx-1.25.2/nginx/logs`
>
> 配置文件：`/opt/nginx-1.25.2/nginx/conf`
> 
> service文件：`/usr/lib/systemd/system/nginx.service`



1. 下载安装包：`cd /opt && wget https://nginx.org/download/nginx-1.25.2.tar.gz`

2. 解压：`tar -xzf nginx-1.25.2.tar.gz`

3. 指定安装目录：`cd /opt/nginx-1.25.2 && ./configure --prefix=/opt/nginx-1.25.2/nginx --with-http_ssl_module && make && make install`

4. 启动服务：`/opt/nginx-1.25.2/nginx/sbin/nginx -c  /opt/nginx-1.25.2/nginx/conf/nginx.conf`

5. 验证：`curl 127.0.0.1:80`; 成功后停止服务
   ```
   ...
   <title>Welcome to nginx!</title>
   ...
   ```

6. 创建service文件：`vim /usr/lib/systemd/system/nginx.service`
   
   ```
   [Unit]
   Description=sweetpotato teams nginx
   After=network.target
   
   [Service]
   Type=forking
   ExecStart=/opt/nginx-1.25.2/nginx/sbin/nginx -c  /opt/nginx-1.25.2/nginx/conf/nginx.conf
   ExecReload=/opt/nginx-1.25.2/nginx/sbin/nginx -s reload
   ExecStop=/opt/nginx-1.25.2/nginx/sbin/nginx -s stop
   Restart=always
   RestartSec=30s
   
   [Install]
   WantedBy=multi-user.target
   ```

7. 启动nginx：`systemctl start nginx.service && systemctl enable nginx.service`

## 故障排除
1. ./configure: error: the HTTP rewrite module requires the PCRE library.
```
apt-get install libpcre3-dev
```

## 参考
1. 自动生成nginx配置：[https://nginxconfig.io/](https://nginxconfig.io/)
