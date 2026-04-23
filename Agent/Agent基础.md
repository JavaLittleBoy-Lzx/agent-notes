---
title: Agent 四大核心组件
date: 2026-04-23
tags:
  - Agent基础
  - 面试
source: https://mp.weixin.qq.com/s/FoC5Y4VWe2PaGCtdkcDhTw
image: assets/images/agent-workflow.png
---

# Agent 四大核心组件

> [!abstract] 一句话回答
> Agent = **LLM（大脑）** + **工具（手脚）** + **记忆（存档）** + **规划（项目经理）**

---

## 🏢 类比：公司四角色

把 Agent 系统类比成一家公司：

| 组件 | 公司角色 | 职责 |
|------|----------|------|
| **LLM** | 老板 | 所有决策必经它拍板 |
| **工具系统** | 外包团队 | 真正执行干活 |
| **记忆系统** | 档案室 | 信息存取，不丢上下文 |
| **规划模块** | 项目经理 | 大目标拆成小步骤 |

---

## 1. LLM 核心 — 的大脑

LLM 是整个 Agent 的**唯一决策者**，其他三个组件都是它的"工具"。

**它判断三件事：**
1. 下一步该做什么？
2. 调用哪个工具？
3. 还是直接给最终答案？

> 没有 LLM，其他三个组件就是一堆零件，无人指挥。

---

## 2. 工具系统 — Agent 的手脚

LLM 不能上网、不能读文件、不能执行代码，工具就是它的"外挂"。

### 可以封装成工具的：
- 🌐 搜索引擎
- 💻 代码执行器
- 📧 发邮件 API
- 🗄️ 数据库查询
- 任何能用函数封装的能力

### 工具调用流程：

```
LLM 返回 {tool_name, arguments}
        ↓
程序执行真正的 API 调用
        ↓
结果反馈给 LLM
        ↓
LLM 生成最终回答
```

### 标准格式（OpenAI Function Calling）：

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "搜索互联网信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词"
                    }
                },
                "required": ["query"]
            }
        }
    }
]

# LLM 返回的是 JSON，不是真正执行
# {"tool_call": {"name": "search_web", "arguments": {"query": "..."}}}
```

> [!tip] 核心分工
> **模型决定"做什么"，程序负责"真正执行"**

---

## 3. 记忆系统 — Agent 的存档

| 类型 | 作用 | 生命周期 |
|------|------|----------|
| **短期记忆** | 当前对话上下文，存中间结果 | 任务结束即清空 |
| **长期记忆** | 向量数据库，语义检索 | 跨任务持久化 |

### 短期记忆例子：

```
第1步：搜索"苹果市值" → 记住结果
第2步：搜索"谷歌市值" → 记住结果
第3步：比较两者 → 用到前两步的结果
```

### 长期记忆实现：

- 用**向量数据库**存储 embedding
- 下次任务时，语义检索相关记忆调出来
- 像人的长期记忆，需要"主动回忆"

---

## 4. 规划模块 — Agent 的项目经理

**作用：** 把大目标拆成可执行的小步骤

### 例子：目标 = "写一份竞品分析报告"

```
1. 搜索竞品资料
2. 整理关键数据
3. 对比分析
4. 撰写报告
```

### 两种实现方式：

| 方式 | 描述 |
|------|------|
| **Plan-and-Execute** | 先让 LLM 输出完整计划，再按计划一步步执行 |
| **Error-Recovery** | 边执行边规划，根据每步结果动态调整 |

---

## 🔄 Agent 核心运行循环

```python
def agent_run(user_goal: str):
    plan = llm.plan(user_goal)          # 规划：拆解目标
    memory = []                          # 初始化短期记忆

    for step in plan:
        action = llm.decide(
            step=step,
            history=memory,              # 短期记忆：前几步的结果
            long_term=vector_db.search(step)  # 长期记忆：相关历史
        )

        if action.type == "tool_call":
            result = tools.execute(action.tool_name, action.args)
            memory.append({"step": step, "result": result})  # 存入记忆

        elif action.type == "final_answer":
            return action.content  # 任务完成
```

### 核心节奏：

```
规划 → 决策 → 执行 → 存入记忆 → 再决策 → 循环直到完成
```

---

## 🏗️ 主流框架

| 框架 | 特点 |
|------|------|
| **LangChain** | 全功能封装，灵活度高 |
| **LlamaIndex** |专注知识检索和 RAG |
| **AutoGen** | 多 Agent 协作 |

本质都是围绕**四大组件**设计，只是封装方式不同。

---

## 🎯 面试考察点

1. **说出四大组件** — LLM / 工具 / 记忆 / 规划
2. **工具调用流程** — LLM 返回 JSON → 程序执行 → 结果反馈
3. **短期 vs 长期记忆** — 上下文窗口 vs 向量数据库
4. **规划模块的作用** — 拆解复杂目标
5. **Agent 循环** — 规划→决策→执行→记忆→循环

---

## 📚 相关面试题

- [[Function-Calling/大模型是怎么查天气的]]
- [[Function-Calling/大模型怎么学会调用外部工具]]
- [[协议/MCP协议是什么]]
