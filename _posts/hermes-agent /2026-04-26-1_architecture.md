---
title: 一、架构
date: 2026-04-26 00:00:00 +0800
categories: [ai,hermes-agent]
tags: [ai,hermes-agent]
author: ahern
---

## 系统概述

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Entry Points                                 │
│                                                                     │
│  CLI (cli.py)    Gateway (gateway/run.py)    ACP (acp_adapter/)     │
│  Batch Runner    API Server                  Python Library         │
└──────────┬──────────────┬───────────────────────┬───────────────────┘
           │              │                       │
           ▼              ▼                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     AIAgent (run_agent.py)                          │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Prompt       │  │ Provider     │  │ Tool         │               │
│  │ Builder      │  │ Resolution   │  │ Dispatch     │               │
│  │ (prompt_     │  │ (runtime_    │  │ (model_      │               │
│  │  builder.py) │  │  provider.py)│  │  tools.py)   │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                 │                 │                       │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐               │
│  │ Compression  │  │ 3 API Modes  │  │ Tool Registry│               │
│  │ & Caching    │  │ chat_compl.  │  │ (registry.py)│               │
│  │              │  │ codex_resp.  │  │ 61 tools     │               │
│  │              │  │ anthropic    │  │ 52 toolsets  │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────┴─────────────────┴─────────────────┴───────────────────────┘
           │                                    │
           ▼                                    ▼
┌───────────────────┐              ┌──────────────────────┐
│ Session Storage   │              │ Tool Backends        │
│ (SQLite + FTS5)   │              │ Terminal (7 backends)│
│ hermes_state.py   │              │ Browser (5 backends) │
│ gateway/session.py│              │ Web (4 backends)     │
└───────────────────┘              │ MCP (dynamic)        │
                                   │ File, Vision, etc.   │
                                   └──────────────────────┘
```

### 接入层

包括 CLI、Messaging Gateway、ACP、API Server、Python Library等组件。

### Agent 引擎层

核心是以 ReAct 模式循环，负责提示词构建、模型调用、工具执行、重试、fallback、上下文压缩和持久化。

- Prompt（Context） 构建：组装系统提示词，来源包括 会话历史、memory、skills、AGENTS.md / .hermes.md、工具使用规则（Schema）、模型等。
- Memory 管理：包含长期记忆，会话历史召回和外部记忆 provider。
- LLM Provider：适配多家大模型提供商。

### 存储层
- SQLite + FTS5：SQLite 保存会话历史，FTS5提供Memory全文检索召回
- 文件系统：保存长期记忆（MEMORY.md、 USER.md）等

### 工具层
负责工具注册，工具集分组，多 backend 调用等，覆盖 web、terminal、file、browser、vision、memory、cron、delegation、MCP 等。

### Skills 系统
兼容 agentskills.io 标准，并能在使用中**自我改进**。

## Hermes VS OpenClaw

## 参考
- [hermes-agent](https://github.com/nousresearch/hermes-agent)
- [openclaw](https://github.com/openclaw/openclaw)