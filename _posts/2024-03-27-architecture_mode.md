---
title: 常见架构模式
date: 2024-03-27 00:00:00 +0800
categories: [root, architecture]
tags: [architecture]
author: ahern
---

### 分层模式
![img.png](./assets/images/img_6.png){:height="10%" width="50%"}
每一层有特定的角色和职责；请求逐层向下传递，并逐层向上返回
- 展示层（View）：用户UI页面，请求输入和响应展示
- 控制层（Control）：执行业务逻辑
- 应用层（Service）：控制层和数据层的桥梁
- 数据层（Dao）：操作数据库

场景：
- 适合小型web服务
优点：
- 前期开发成本低，可快速完成应用
- 分层职责分明，可维护性高
缺点：
- 业务请求需要经过多层处理，性能低

### 微内核
如 [Opentelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector)
![img.png](./assets/images/img_7.png){:height="10%" width="50%"}
- 核心部分：核心模块保证程序最小可运行部分
- 插件部分：插件模块则按需接入和可自定义开发

场景：
- 程序拥有固定核心逻辑，追求高扩展性，适合开源场景和多团队开发
优点：
- 可扩展，可移植
缺点：

### 事件驱动
![img.png](./assets/images/img_8.png){:height="10%" width="50%"}
事件产生并发送到Channel，由事件调度器调度到不同的处理器执行
- 事件产生：各种业务场景下触发生成事件
- Channel：事件队列
- 调度器：从Channel中取出事件，并调度到不同的处理器
- 处理器：处理特定类型的事件
场景：
- 电商下单场景
优点：
- 大型复杂系统下，处理异步事件

### 阶段事件驱动
![img.png](./assets/images/img.png){:height="10%" width="50%"}
简称SEDA（stage event driver architecture），各个stage之间通过event通讯，stage内处理event使用线程池异步处理。结合了事件驱动和多线程模式优点。

场景：
- Pipeline处理
- 可异步

优点：
- 易扩展
- 事件驱动，异步，解耦
- 充分利用多线程模型

### Pipeline管道-过滤器
![img.png](./assets/images/img_1.png){:height="10%" width="50%"}
统一过滤器之间通讯协议，每个管道都是非定向的和点对点，接收一个源的输入和输出到另一个源

场景：
- 流水线处理
- 语法分词

优点：
- 可插拔、易扩展

缺点：
- 不适合交互性系统
- 过滤器之间频繁解析和反解析导致性能损失

### 微服务
![img.png](./assets/images/img_3.png){:height="10%" width="50%"}
将系统划分多个独立服务，每个服务可以独立部署，拥有自己的api边界，管理自己的数据库，可以是不同的开发语言。

场景：
- 适合大型系统

优点：
- 容灾性好
- 配合k8s扩缩容

缺点：
- 系统设计必须容忍服务失败
- 分布式事务问题
- 运维复杂

### 基于空间
![img.png](./assets/images/img_4.png){:height="10%" width="50%"}

### 参考
- [架构师必须了解的 5 种最佳软件架构模式](https://www.infoq.cn/article/vrquohwkwjjghb5wvx1y)
- [程序员必知的几种软件架构模式](https://www.infoq.cn/article/6rx047oohjlrdipd1bc2)
- [阶段式服务器模型](https://zh.wikipedia.org/wiki/%E9%98%B6%E6%AE%B5%E5%BC%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%A8%A1%E5%9E%8B)
- [SEDA架构实现](https://blog.51cto.com/SoyTechnology/3346495)
- [基于空间的架构](http://www.uml.org.cn/zjjs/202212164.asp)
