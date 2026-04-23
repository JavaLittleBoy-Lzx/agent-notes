# 腾讯面试官：MCP 协议是什么？

> 面试题来源：微信公众号 - 小林coding

## 题目

连 MCP 协议都没玩过，还敢在简历上写精通 Agent 开发？

## 核心知识点

**MCP (Model Context Protocol) 协议**

## 答案要点

1. **MCP 是 Anthropic 提出的开放协议**，用于标准化 Agent 与外部工具/数据源的连接
2. **解决的问题**：
   - 之前每个工具提供商都有自己的 API 标准，互不兼容
   - MCP 提供统一标准，Agent 只需实现一次，就能连接所有支持 MCP 的工具
3. **MCP 的三个核心组件**：
   - **Host**：发起操作的 AI 应用（如 Claude Desktop）
   - **Client**：与每个工具保持 1:1 连接的客户端
   - **Server**：暴露特定能力的轻量级程序（如 Slack Server、GitHub Server）

## 类比理解

**MCP 就像 USB 接口规范**——有了统一标准，任何 USB 设备都能插在任何 USB 接口上。

---

tags: #AI-Agent #面试 #MCP #协议 #Anthropic
