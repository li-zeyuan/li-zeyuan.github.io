---
title: 网卡性能指标
date: 2023-09-06 00:00:00 +0800
categories: [linux]
tags: [linux]
author: ahern
---

## 性能指标
- 带宽：链路的最大传输速率
- 延时：一个数据包往返所需的时间延迟
- 吞吐率：单位时间内成功传输数据大小
- PPS：单位时间传输包数量
- 连接数：TCP连接数
- 丢包率：所丢失数据包数量占所发送数据组的比率
- 重传率：重传网络包的比例

## 带宽
ethtool eth0 | grep Speed

## 延时
ping

## 吞吐率/PPS
sar -n DEV 1
```
18:30:49        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
18:30:50           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
18:30:50         eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```
- 1：表示每1秒输出结果
- EDEV：网络错误的统计数据
- rxpck/s 和 txpck/s 分别是接收和发送的 PPS，单位为包 / 秒。
- rxkB/s 和 txkB/s 分别是接收和发送的吞吐率，单位是 KB/ 秒。
- rxcmp/s 和 txcmp/s 分别是接收和发送的压缩数据包数，单位是包 / 秒

## 连接数

