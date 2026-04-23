---
title: A2A 和 MCP 区别
date: 2026-04-23
tags:
  - 协议
  - A2A
  - MCP
  - 面试
source: https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483728&idx=1&sn=8feb775f055f697517d0a3ad424a8c7b
---

# A2A 和 MCP 区别

## 题目

简历上写着精通 Agent 开发，那你说明白 A2A 和 MCP 到底啥区别？

## 核心知识点

**A2A vs MCP 定位差异**

## 答案要点

| 协议 | 全称 | 解决的问题 | 定位 |
|------|------|------------|------|
| **MCP** | Model Context Protocol | Agent 如何调用外部工具 | Agent ↔ 工具/数据源 |
| **A2A** | Agent to Agent Protocol | 多个 Agent 之间如何协作 | Agent ↔ Agent |

## 大白话理解

- **MCP**：Agent 怎么用工具（类比：人怎么用电脑）
- **A2A**：Agent 之间怎么聊天协作（类比：员工之间怎么沟通）

## 两者关系

互补协议，完整的 Agent 系统通常需要两者配合：
- MCP 处理 Agent 与外部工具的交互
- A2A 处理多 Agent 协作场景

---

tags: #AI-Agent #面试 #A2A #MCP #协议
