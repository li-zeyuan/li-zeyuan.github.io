---
title: 常见架构模式
date: 2024-03-27 00:00:00 +0800
categories: [root, architecture]
tags: [architecture]
---

### 分层模式
每一层有特定的角色和职责；请求逐层向下传递，并逐层向上返回
![img.png](./assets/images/img_6.png){:height="10%" width="50%"}
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
事件产生并发送到Channel，由事件调度器调度到不同的处理器执行
![img.png](./assets/images/img_8.png){:height="10%" width="50%"}
- 事件产生：各种业务场景下触发生成事件
- Channel：事件队列
- 调度器：从Channel中取出事件，并调度到不同的处理器
- 处理器：处理特定类型的事件
场景：
- 电商下单场景
优点：
- 大型复杂系统下，处理异步事件

### 阶段事件驱动
### Pipeline管道-过滤器
### 微服务
### 基于空间

### 参考
- [阶段式服务器模型](https://zh.wikipedia.org/wiki/%E9%98%B6%E6%AE%B5%E5%BC%8F%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%A8%A1%E5%9E%8B)
- [架构师必须了解的 5 种最佳软件架构模式](https://www.infoq.cn/article/vrquohwkwjjghb5wvx1y)
- [程序员必知的几种软件架构模式](https://www.infoq.cn/article/6rx047oohjlrdipd1bc2)
