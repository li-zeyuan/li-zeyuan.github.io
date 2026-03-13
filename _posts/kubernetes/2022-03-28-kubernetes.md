---
title: Kubernetes系统架构和原理
date: 2022-03-28 00:00:00 +0800
categories: [middleware, kubernetes]
tags: [kubernetes]
author: ahern
---

## 架构
```
    +-------------------+
    |   kube-apiserver  |
    +-------------------+
    |           |      |
scheduler  controller  etcd
    |
+-----------+
|   Node    |
+-----------+
| kubelet   |
| kubeproxy |
| container |
```

master 节点实现对集群管理，有 4 个组件：
- api-server：提供restful接口，实现整个k8s集群通信
- scheduler：过滤节点和对节点打分，为pod选择最优的节点
- controller：集群的管理控制中心，对集群的资源进行管理
- etcd：非关系型数据库，集群内资源的存储


<br>
node 节点实现对pod的管理，有 3 个组件：
- kubelet：节点的管理组件，通过 CRI 与 container runtime通讯，监控节点状态和容器状态。
> CRI(Container Runtime Interface): Kubelet 与 container runtime 通讯接口规范。
> 
> kubelet /kyoo-blet/ 与 kubectl /kube-control/
>
> kubelet是节点管理组件，kubectl是用户命令行工具。
- kube-proxy：节点的网络代理组件，通过 iptables 或 IPVS 维护网络规则，将请求转发到 pod 。
- container runtime：是负责实际运行容器的软件组件，Kubelet 通过 CRI 调用 Runtime 完成镜像拉取、容器创建、启动和生命周期管理。

## 创建一个pod的流程

- ![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210310100538.png)

- api-server处理用户请求，将pod信息存储到etcd中
- kube-scheduler预选和优选为pod选择最优的节点
- 节点的kubelet从节点点中获取pod清单，下载镜像启动容器

## Q & A

### k8s为什么不用redis，而用etcd？

| 特性    | etcd     | Redis |
| ----- | -------- | ----- |
| 设计目标  | 分布式一致性存储 | 高性能缓存 |
| 一致性   | 强一致      | 最终一致  |
| 共识算法  | Raft     | 无     |
| Watch | 原生支持     | 不支持   |
| 版本控制  | revision | 无     |
| 事务    | 分布式事务    | 简单事务  |
| 持久化   | 强持久化     | 内存为主  |

### k8s网络不通排查
- 先确认 物理/节点网络可达
- 再确认 Pod 网络是否创建成功
- 再确认 Service 虚拟 IP 是否可达
- 最后检查 策略、DNS、kube-proxy 规则

## 参考

- 从零开始入门 K8s：详解 K8s 核心概念：https://www.infoq.cn/article/knmavdo3jxs3qpkqtzbw
- 官方文档：https://kubernetes.io/zh/docs/home/
- https://www.jianshu.com/p/2de643caefc1
- 学习路线：https://www.infoq.cn/article/9dtx*1i1z8hsxkdrpmhk
- minikube：https://github.com/kubernetes/minikube
- minikube docs：https://minikube.sigs.k8s.io/docs/start/
- kubectl安装：https://github.com/caicloud/kube-ladder/blob/master/tutorials/lab1-installation.md
