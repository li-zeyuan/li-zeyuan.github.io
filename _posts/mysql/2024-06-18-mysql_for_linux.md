---
title: Linux二进制安装Mysql
date: 2024-06-18 00:00:00 +0800
categories: [middleware, mysql]
tags: [mysql]
author: ahern
---

## 安装环境

- 一台机器上部署两个实例
  ```
  my.cnf文件务必区别：

  port=3306
  basedir=/opt/mysql-8.0.11
  datadir=/opt/mysql-8.0.11/mysql-dev/data
  socket=/tmp/mysql-dev.sock
  log-error=/opt/mysql-8.0.11/mysql-dev/logs/err.log
  ```
- Ubuntu 22.04 LTS
- mysql版本：8.0.11

## 安装mysql-dev

> 安装包：`/opt/mysql-8.0.11/`
> 
> 配置文件：`/opt/mysql-8.0.11/mysql-dev/conf/my.cnf`
> 
> service文件：`/usr/lib/systemd/system/mysql-dev.service`
> 
> 数据路径：`/opt/mysql-8.0.11/mysql-dev/data/`
> 
> 日志路径：`/opt/mysql-8.0.11/mysql-dev/logs/`



1. 下载安装包：`cd /opt && wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz`

2. 解压：`tar -xzf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz && mv mysql-8.0.11-linux-glibc2.12-x86_64 mysql-8.0.11`

3. 创建数据目录、配置目录、日志目录：`mkdir -p /opt/mysql-8.0.11/mysql-dev/logs /opt/mysql-8.0.11/mysql-dev/conf /opt/mysql-8.0.11/mysql-dev/data`

4. 创建配置文件：`vim /opt/mysql-8.0.11/mysql-dev/conf/my.cnf`
   
   ```
   [mysql]
   default-character-set=utf8mb4
   
   [mysqld]
   user=root
   port=3306
   basedir=/opt/mysql-8.0.11
   datadir=/opt/mysql-8.0.11/mysql-dev/data
   socket=/tmp/mysql-dev.sock
   log-error=/opt/mysql-8.0.11/mysql-dev/logs/err.log
   default-time_zone='+0:00'
   character-set-server=utf8mb4
   default_authentication_plugin=mysql_native_password
   max_binlog_size=100M
   #忘记密码时使用
   #skip-grant-tables
   ```

5. 创建service文件：`vim /usr/lib/systemd/system/mysql-dev.service`
   
   ```
   [Unit]
   Description=sweetpotato teams mysql-dev
   After=network.target
   
   [Service]
   Type=simple
   WorkingDirectory=/opt/mysql-8.0.11/mysql-dev
   ExecStart=/opt/mysql-8.0.11/bin/mysqld --defaults-file=/opt/mysql-8.0.11/mysql-dev/conf/my.cnf
   Restart=always
   RestartSec=30s
   
   [Install]
   WantedBy=multi-user.target
   ```

6. 初始化data目录: `/opt/mysql-8.0.11/bin/mysqld --defaults-file=/opt/mysql-8.0.11/mysql-dev/conf/my.cnf  --initialize --user=root`

7. 启动mysqld：`systemctl start mysql-dev.service && systemctl enable mysql-dev.service`

## 安装mysql-pro
从以上第3步开始即可

## 首次登陆
> 在`/data/mysql/logs/err.log`找到初始密码
1、查找初始密码：`grep root@localhost /opt/mysql-8.0.11/mysql-dev/logs/err.log`

2、登陆myslq：`mysql -S /tmp/mysql-dev.sock -p`

3、修改密码：`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';`, `flush privileges;`

> 忘记密码

1. `/opt/mysql-8.0.11/mysql-dev/conf/my.cnf`增加`skip-grant-tables`

2. 重启mysqld：`systemctl restart mysql-dev.service`

3. 登陆mysql：`mysql -uroot -p 3306 -S /tmp/mysql-dev.sock`,不用输入密码

4. 清空密码：`update mysql.user set authentication_string='' where user='root';`

5. `/opt/mysql-8.0.11/mysql-dev/conf/my.cnf`去掉`skip-grant-tables`

6. 重启mysqld：`systemctl restart mysql-dev.service`

7. 登陆mysql：`mysql -uroot -p 3306 -S /tmp/mysql-dev.sock`,不用输入密码

8. 设置密码：`update mysql.user set authentication_string='xx' where user='root';`

## 允许远程登陆

1. `update mysql.user  set host = '%' where user = 'root';`

2. `FLUSH PRIVILEGES;`

## 故障排除

1. 报错：mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
   
   ```
   apt install libncurses5
   ```

2. 报错：
   ```
   root@VM-8-13-ubuntu:/opt# /opt/mysql-8.0.11/bin/mysqld --defaults-file=/opt/mysql-8.0.11/mysql-dev/conf/my.cnf --initialize --user=root
   /opt/mysql-8.0.11/bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
   ```

   https://www.cnblogs.com/baozixiaoge/p/18277347
