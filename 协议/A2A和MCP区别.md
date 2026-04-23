---
title: A2A 和 MCP 区别
date: 2026-04-23
tags:
  - 协议
  - A2A
  - MCP
  - 面试
source: https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483728&idx=1&sn=8feb775f055f697517d0a3ad424a8c7b
image: assets/images/a2a-vs-mcp.png
---

# A2A 和 MCP 区别

> [!abstract] 一句话回答
> **MCP** = Agent 用工具（类比：人怎么用电脑）**A2A** = Agent 之间聊天协作（类比：员工之间怎么沟通）。两者互补，完整的 Agent 系统通常需要同时使用。

---

## 🆚 对比

| | MCP | A2A |
|---|---|---|
| **全称** | Model Context Protocol | Agent to Agent Protocol |
| **解决的问题** | Agent 如何调用外部工具 | 多个 Agent 之间如何协作 |
| **定位** | Agent ↔ 工具/数据源 | Agent ↔ Agent |
| **类比** | 人怎么用电脑 | 员工之间怎么开会 |
| **典型场景** | 连接搜索API/数据库/邮件 | 多 Agent 协作完成复杂任务 |

---

## 🔧 MCP：Agent 的"工具箱"

```
Agent ──── MCP ──── 搜索API
         │
         ├── MCP ──── 数据库
         │
         └── MCP ──── 邮件系统
```

---

## 🤝 A2A：Agent 的"通讯录"

```
Agent A ←── A2A ──→ Agent B
  │                    │
  └── 共享上下文/任务 ──┘
```

---

## 💡 两者关系

> **MCP + A2A = 完整的 Agent 协作体系**

- MCP 让单个 Agent 能"干活"（调用工具）
- A2A 让多个 Agent 能"配合"（互相通信）

一个复杂系统里，既有 MCP 接入外部工具，也有 A2A 让 Agent 之间分工协作。

---

## 🎯 面试考察点

1. **说出区别** — MCP 是 Agent→工具，A2A 是 Agent→Agent
2. **类比记忆** — MCP=用电脑，A2A=员工沟通
3. **两者关系** — 互补协议，配合使用

---

## 📚 相关面试题

- [[协议/MCP协议是什么]] — MCP 协议详解
- [[Agent基础/Agent四大核心组件]] — Agent 完整架构
