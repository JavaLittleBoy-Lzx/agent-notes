---
title: MCP 协议是什么？
date: 2026-04-23
tags:
  - 协议
  - MCP
  - 面试
source: https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483673&idx=1&sn=0010e78aece8b05388b94d0396444a03
---

![配图](assets/images/mcp-protocol-gpt2.png)


## 🏗️ MCP 三个核心组件

```
┌─────────────────┐
│   AI 应用       │  ← Host（发起操作的 AI，如 Claude Desktop）
│  (AI 应用)      │
└───────┬─────────┘
        │
┌───────▼─────────┐
│  MCP Client     │  ← 每个工具保持 1:1 连接
│ (和每个工具通信) │
└───────┬─────────┘
        │
┌───────▼─────────┐
│  MCP Server     │  ← 暴露特定能力的轻量程序
│ (Slack/GitHub)  │
└─────────────────┘
```

---

## ✨ MCP 解决的问题

**之前：** 每个工具提供商有自己的 API 标准
```python
# Slack 工具
slack_client.chat_postMessage(...)

# GitHub 工具
github_client.issues.create(...)
```
每次换工具都要重写集成代码。

**有了 MCP：** 统一协议，一次集成，永久使用
```python
# 无论什么工具，都是同样的 MCP 调用方式
mcp_client.call_tool("slack", "chat_postMessage", {...})
mcp_client.call_tool("github", "issues_create", {...})
```

---

## 🎯 面试考察点

1. **MCP 是什么** — Anthropic 提出的开放协议
2. **解决什么问题** — 标准化 Agent 与工具的连接
3. **三个组件** — Host / Client / Server
4. **USB 类比** — 统一接口，一次集成，永久使用

---

## 📚 相关面试题

- [[协议/A2A和MCP区别]] — A2A vs MCP
- [[Agent基础/Agent四大核心组件]] — 工具系统在 Agent 中的角色
