---
title: Agent 四大核心组件
date: 2026-04-23
tags:
  - Agent基础
source: https://mp.weixin.qq.com/s/FoC5Y4VWe2PaGCtdkcDhTw
---

在小说阅读器读本章

去阅读

大家好，我是小林。

这次来学习读者跟我反馈小红书二面遇到的面试题：「 **Agent 的基本架构由哪些核心组件构成？** 」

## 💡 简要回答

我理解 Agent 的基本架构有四个核心组件：LLM、工具、记忆、规划模块。

LLM 是整个系统的大脑，负责理解任务和做决策。

工具让 Agent 能跟外部世界交互，搜索、执行代码、调 API 都靠它。

记忆让 Agent 在任务执行过程中保持状态，不会「失忆」。

规划模块负责把复杂目标拆解成可执行的步骤。

这四个组合在一起，才让 Agent 具备了自主完成任务的能力。

## 📝 详细解析

理解了 Agent 是什么之后，我们来看它的内部结构，一个完整的 Agent 系统，到底由哪几个核心部件组成。

你可以把整个 Agent 系统类比成一家公司： **LLM 是老板** ，所有决策都经过它拍板； **工具系统是外包执行团队** ，老板说「去搜这个」「去发这封邮件」，他们负责真正干活； **记忆系统是公司档案室** ，各种信息的存档和调档都靠它； **规划模块是项目经理** ，拿到一个大目标后负责拆解成可执行的任务单。四个角色各司其职，才撑起了 Agent 的自主运行能力。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zLRM1IicjS8jnwib2h5uO42rQ27pWKV5YicV4SZ1OJRLdWibRlaMcEKKXkzfh1M8zClLSIyhwxV0KL9gWZFBLCaiaKPkaxFGNX6ibAjvSBLbOggZw/640?wx_fmt=png&from=appmsg)

先来说 **LLM 核心** 。它是整个 Agent 的大脑，所有的输入，不管是用户的指令、工具返回的结果还是记忆里调出来的内容，最终都要经过 LLM 来理解和决策。它负责判断：下一步该做什么？是继续思考、调用某个工具、还是已经可以给出最终答案了？没有 LLM，其他三个组件就是一堆零件，没有人来统一指挥。

然后是 **工具系统** ，这是 Agent 和外部世界交互的唯一入口。LLM 本身是个纯粹的「语言处理器」，它不能上网、不能读文件、不能执行代码，但这些限制都可以通过工具来突破。工具可以是搜索引擎、数据库查询、代码执行器、发邮件的 API，任何你能用函数封装的能力都可以变成工具。

工具是怎么定义的？我给你看一个最标准的格式：

```
# 定义工具的结构（以 OpenAI function calling 格式为例）
# 你只需要告诉模型三件事：工具叫什么名、能做什么事、需要哪些参数
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

# LLM 决定调用工具时，会返回类似这样的结构：
# {"tool_call": {"name": "search_web", "arguments": {"query": "2024年大模型最新进展"}}}
# 然后你的代码负责真正执行这个搜索，把结果再塞回给 LLM
```

你看，工具定义里没有一行执行逻辑，只有「名字、描述、参数说明」。模型读了这份说明书，决定要调哪个工具、参数填什么，把决策以 JSON 格式告诉你，你的代码去真正执行，结果再反馈给模型。整个分工很清晰： **模型负责「决定做什么」，程序负责「真正执行」** 。

接下来是 **记忆系统** ，它分两层，你可以类比人的记忆方式来理解。短期记忆就是当前这轮对话的上下文，装在 context window 里，Agent 在一次任务执行过程中靠它记住中间状态，比如第一步搜索到了什么、第二步执行结果是什么。这就像人的「工作记忆」，容量有限，任务一结束就清空了。所以还需要长期记忆，通常用向量数据库来实现，把重要信息 embedding 之后存起来，下次用的时候做语义检索拿回来。这就像人的「长期记忆」，容量大、可以跨天保留，但需要主动「回忆」才能调出来。

最后是 **规划模块** ，它决定了 Agent 能不能应对复杂任务。简单任务一步就搞定了，但如果你让 Agent「帮我写一份竞品分析报告」，它需要先把这个目标拆解：搜索竞品资料 -> 整理关键数据 -> 对比分析 -> 撰写报告。规划模块就是做这件事的，有些实现是让 LLM 先输出一个完整计划再逐步执行，有些是边执行边规划，根据每步结果动态调整。

这四个组件合在一起，到底是怎么跑起来的？我用一段伪代码来还原整个运行过程，看完你就能理解它们是怎么协作的：

```
# Agent 运行的核心 loop（伪代码）
def agent_run(user_goal: str):
    # 第一步：规划模块上场，把目标拆成步骤列表
    plan = llm.plan(user_goal)

    memory = []  # 短期记忆，用来存每一步的中间结果

    for step in plan:
        # 第二步：LLM 核心做决策，这一步该怎么做？
        action = llm.decide(
            step=step,
            history=memory,                    # 把短期记忆传进去，让它知道之前做了什么
            long_term=vector_db.search(step)   # 从长期记忆里捞出相关历史
        )

        if action.type == "tool_call":
            # 第三步：工具系统负责真正执行
            result = tools.execute(action.tool_name, action.args)
            memory.append({"step": step, "result": result})  # 执行结果存入短期记忆

        elif action.type == "final_answer":
            return action.content  # LLM 判断任务完成，返回最终答案
```

看完这段伪代码，你会发现 Agent 的核心节奏其实很简单：规划 -> 决策 -> 执行 -> 结果存入记忆 -> 再决策，循环往复，直到任务完成。LLM 始终是那个做决策的角色，工具系统是执行者，记忆系统让它不会「失忆」，规划模块帮它把大目标拆成小步骤。

LangChain、LlamaIndex、AutoGen 这些主流框架，本质上都是围绕这四个组件来设计的，只是封装方式和侧重点各有不同。

往期AI Agent面试题：

[字节一面秒挂！问“大模型是怎么查天气的？”，我答“它自己去调 API”，面试官冷笑...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483764&idx=1&sn=6e80c70f995acba3fa05d2c1b3a88b0c&scene=21#wechat_redirect)

[字节二面：被问“大模型知识过时了怎么解？”，我答“微调”，面试官当场黑脸：“听说过 RAG 吗？”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483658&idx=1&sn=02673313e69b2a6a304ad7462e4e4a29&scene=21#wechat_redirect)

[腾讯面试官怒怼：“连 MCP 协议都没玩过，还敢在简历上写精通 Agent 开发？”，只会调包的我懵了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483673&idx=1&sn=0010e78aece8b05388b94d0396444a03&scene=21#wechat_redirect)

[阿里技术官拷问：“简历上写着精通 Agent 开发，那你说明白 A2A 和 MCP 到底啥区别？”，我：“呃... 不都是协议吗？”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483728&idx=1&sn=8feb775f055f697517d0a3ad424a8c7b&scene=21#wechat_redirect)

[腾讯面试官一句追问："做了这么多Agent项目，大模型怎么学会调用外部工具都说不清？" 我当场卡壳，二面凉了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483763&idx=1&sn=b3f2abee33ad0cf37282f8814b0960a5&scene=21#wechat_redirect)