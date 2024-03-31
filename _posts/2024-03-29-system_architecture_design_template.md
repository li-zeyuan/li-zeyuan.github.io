---
title: 系统架构设计模板
date: 2024-03-29 00:00:00 +0800
categories: [root, architecture]
tags: [architecture]
author: ahern
---

### 步骤
- 步骤1：功能架构分层
- 步骤2：产品架构图
- 步骤3：应用架构图
- 步骤4：技术架构图
- 步骤5：部署架构图

### 功能架构分层
![img.png](./assets/images/img_9.png){:height="10%" width="50%"}
- 同一个产品范围，同一组产品功能模块放在同一层

### 产品架构图
![img.png](./assets/images/img_10.png){:height="10%" width="50%"}
- 展示产品核心功能
- 产品架构图至少分三层：**用户感知层、功能模块层、数据层**
- 层级内子模块功能独立，界限分明，架构分层合理
- 参考：https://www.infoq.cn/article/5A8LiWThDdHpkjeKgWLk

### 应用架构图
参考：https://www.infoq.cn/article/ZzI05OBgks2kspUWa5y7
#### 单体应用
![img.png](./assets/images/img_11.png){:height="10%" width="50%"}
- 分为4层：表现层，业务层，数据层，基础通用层
- 表现层：面向用户，用户通过表现层与系统交互
- 业务层：处理具体业务逻辑
- 数据层：与数据库交互
- 基础通用层：不包含业务逻辑，通用能力，如：工具类，中间件
#### 分布式应用
![img.png](./assets/images/img_12.png){:height="10%" width="50%"}
- 清晰应用边界
- 明确应用之间调用关系
- 分为输入、输出系统，或划分数据输入、输出、控制流

### 技术架构图
![img.png](./assets/images/img_13.png){:height="10%" width="50%"}
- 以产品架构图，应用架构图为基础
- 每个功能点的技术选型
- 子系统之间的交互技术细节
- 参考：https://www.infoq.cn/article/RQDwWxDcwbxtwU8LBFSG

### 部署架构图
![img.png](./assets/images/img_14.png){:height="10%" width="50%"}
- 侧重网络设计、集群设计、数据存储设计
- 展示服务部署细节
- 参考：https://www.infoq.cn/article/RQDwWxDcwbxtwU8LBFSG

