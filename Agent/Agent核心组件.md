# Agent 核心组件

> 本文讲解 Agent 的四大核心组件及其协作方式

- **来源**：[小林图解 - Agent 的基本架构由哪些核心组件构成？](https://mp.weixin.qq.com/s/FoC5Y4VWe2PaGCtdkcDhTw)
- **作者**：小林
- **标签**：#Agent #核心概念 #面试

---

## 🎯 一句话总结

Agent 由 **LLM（大脑） + 工具（执行） + 记忆（状态） + 规划（拆解）** 四大组件构成，类比公司：LLM 是老板、工具是外包团队、记忆是档案室、规划是项目经理。

---

## 🏛️ 四大核心组件

### 1. LLM 核心

**角色**：整个 Agent 的**大脑**，负责所有决策。

**职责**：
- 理解用户指令
- 判断下一步该做什么（继续思考 / 调用工具 / 给出答案）
- 整合工具返回结果、记忆中的信息做综合决策

**特点**：没有 LLM，其他三个组件就是一堆零件，没有人统一指挥。

---

### 2. 工具系统

**角色**：Agent 与**外部世界交互的唯一入口**。

**本质**：把任何能用函数封装的能力变成工具——搜索引擎、数据库查询、代码执行器、发邮件 API 等。

**工作原理**（Function Calling 格式）：

```python
# 工具定义 = 名字 + 描述 + 参数说明（无执行逻辑）
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "搜索互联网上的信息",
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

# LLM 返回决策：{"tool_call": {"name": "search_web", "arguments": {"query": "..."}}}
# 代码负责真正执行，结果反馈给 LLM
```

**分工**：
- 模型 → 决定**做什么**
- 程序 → **真正执行**

---

### 3. 记忆系统

**角色**：让 Agent 在任务执行过程中保持状态，**不会「失忆」**。

**两层结构**：

| 类型 | 类比 | 特点 |
|------|------|------|
| 短期记忆 | 人的"工作记忆" | context window，容量有限，任务结束就清空 |
| 长期记忆 | 人的"长期记忆" | 向量数据库，容量大，跨天保留，语义检索召回 |

---

### 4. 规划模块

**角色**：把复杂目标**拆解**成可执行的步骤。

**职责**：
- 将大目标拆成小任务单（搜索竞品 → 整理数据 → 对比分析 → 撰写报告）
- 两种实现方式：
  1. **先规划后执行**：LLM 先输出完整计划，再逐步执行
  2. **边执行边规划**：根据每步结果动态调整

---

## ⚙️ 核心运行 Loop

```
规划 → 决策 → 执行 → 结果存入记忆 → 再决策 → 循环 → 任务完成
```

**伪代码**：

```python
def agent_run(user_goal: str):
    plan = llm.plan(user_goal)           # 规划模块拆解目标
    memory = []                           # 短期记忆
    
    for step in plan:
        action = llm.decide(
            step=step,
            history=memory,               # 短期记忆（之前做了什么）
            long_term=vector_db.search(step)  # 长期记忆（相关历史）
        )
        
        if action.type == "tool_call":
            result = tools.execute(action.tool_name, action.args)
            memory.append({"step": step, "result": result})  # 存入短期记忆
        elif action.type == "final_answer":
            return action.content
```

---

## 🔗 与主流框架的关系

LangChain、LlamaIndex、AutoGen 等主流框架，本质都是围绕这 **四个组件** 设计，只是封装方式和侧重点不同。

---

## 📝 面试要点

1. **必背**：LLM + 工具 + 记忆 + 规划 四个组件及其作用
2. **类比记忆**：公司模型 — LLM 是老板、工具是外包、记忆是档案室、规划是项目经理
3. **区分**：短期记忆（context window）vs 长期记忆（向量数据库）
4. **理解 Loop**：规划→决策→执行→记忆→再决策 的循环机制
