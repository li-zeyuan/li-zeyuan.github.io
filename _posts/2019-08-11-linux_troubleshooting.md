---
title: Linux故障排查
date: 2019-08-11 00:00:00 +0800
categories: [root, linux]
tags: [linux, troubleshooting]
author: ahern
---

## 查看内存使用情况
top、htop

## 查看CPU使用情况
top、htop

## systemctl status查看service不断重启
#### 原因
服务发生Panic

#### 解决
- `journalctl -xe | grep ${service}`
- `journalctl -u ${service} -r`
- 文档：https://wangchujiang.com/linux-command/c/journalctl.html

## 磁盘空间不足

#### 原因
1. docker日志文件占用空间
2. 服务日志文件占用空间

#### 解决
- 查看分区
```
fdisk -l
```

- 查看分区可使用占比
```
df -lh
```

- 查看当前目录最大的10个文件
```
du --block-size=MiB --max-depth=1 | sort -rn | head -10
```

- 查看大于50的文件，按大小倒序
```
find ./ -type f -size "+50M" -exec du -h {} + | sort -rh|head -n 60
```
