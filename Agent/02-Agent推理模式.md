---
title: Agent 推理模式：ReAct 与 CoT
date: 2026-04-23
tags:
  - Agent基础
  - ReAct
  - CoT
source: https://mp.weixin.qq.com/s/-8oS3sLTizJw2GgLtXRfPQ
---


大家好，我是小林。

前阵子有位林友，去面快手 AI Agent 开发岗，面完第一时间就来找我复盘面试踩坑的经历。

他说前面聊项目细节都很顺畅，结果面试官话锋一转，直接来了个连环炮：“除了 ReAct，你还知道哪些 Agent 推理范式？”

他当时满脑子还在死磕 Prompt 优化的技巧，当场就被问愣住了，支支吾吾半天没说出个所以然。

也正是他的这次真实踩坑，让我决定把 Agent 推理模式这个面试高频核心考点，一次性给大家讲透。

快手面试题真题：「 **Agent 推理模式有哪些？ReAct 是啥？具体是怎么实现的？** 」

## 💡 简要回答

Agent 的推理模式我用过几种。

最基础的是直接输出答案，没有中间推理；CoT 是让 LLM 先把推理过程写出来再给答案，准确率更高；ReAct 是在 CoT 基础上加了「行动」，让 LLM 交替输出思考和工具调用，每次行动后再根据结果继续思考，形成一个循环。

我觉得 ReAct 是目前 Agent 用得最广的模式，因为它推理过程可见，又能动态利用外部工具，两个优点都有。

## 📝 详细解析

### 什么是推理模式？

要理解「推理模式」这个词，得先说清楚 LLM 面临的一个根本困境。

LLM 的工作原理，是根据你给它的输入，一个 token 一个 token 地往后预测。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zLRM1IicjS8jWZWW7OF7Eo5ajVYjroPMicEU2Y1E0iaJvRBvibGJwWwFQ3MF2LJsVkEXV5aK8BbteSA0b2MeGj2BicMVaOwZvLlGUjnaM5juteCE/640?wx_fmt=png&from=appmsg)

你问它一个简单问题，它可以直接说出答案。但如果你问的是一个需要多步推导的问题，比如「A 公司的市值是 B 公司的 1.2 倍，B 公司比 C 公司高 30%，请问 A 和 C 谁更高，差多少？」，LLM 在没有任何辅助的情况下，往往直接给你一个「感觉对」的答案，而这个答案可能是错的。

原因在于，当它「一口气」预测答案时，中间的推导步骤都是隐式的，没有办法强制自己在每一步都做出正确的推断。误差会在中间某个暗处悄悄累积，最终暴露在答案里。

![](https://mmbiz.qpic.cn/mmbiz_png/zLRM1IicjS8hcZUFSvrRIhYssBkOVgmJRLSP9JkK6utEOAcayiasia1ORLiclUKY50Pfg4OTicAT7aia7UOVqMrkrAUFhm6KQGEQibibO1M5Dc0jDA0/640?wx_fmt=png&from=appmsg)

你可以把它类比成心算和笔算的区别。让你心算「123 × 456」，你可能算错；但如果你把每一步都写在纸上，「123 × 6 = 738、123 × 50 = 6150……」，一步一步来，算错的概率就会大大降低。原因不是你突然变聪明了，而是「写下来的过程」本身帮助你避免在中间某步跳跃出错。

LLM 也一样，把推导过程写出来，就等于在每一步都有了一个可以依赖的「前文」，下一步的预测建立在一个已经写清楚的正确基础之上。

「推理模式」存在的根本原因就是这个：通过不同的方式，让 LLM 把隐式的思考过程显式化出来，从而减少多步推理中的累积误差。CoT、ReAct 就是这个方向上的两种解法，每一个都在解决前一个的局限性。

### CoT是什么？

**CoT** ，全称 Chain of Thought（思维链），是最早也最简单的解法。

核心想法极其朴素：在 prompt 里加一句「 **让我们一步步思考** 」， **LLM 就会先把推理步骤写出来，再给答案，而不是直接蹦出结论** 。为什么加一句话就有效？

本质是因为 LLM 的输出是顺序生成的，先写出来的推理内容会进入上下文，成为后续生成的依据。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zLRM1IicjS8jCrw0nauntiafRTTdY03ibeofXGlcuicL8LJcnv0KhxqGOrBF3wDBr6jtnWsrwJGeb9tHASjtLP3PsnFXHJ7L9OTSxfdhLoibHOhs/640?wx_fmt=png&from=appmsg)

当 LLM 先写出「第一步：A 市值是 B 的 1.2 倍，所以 A > B」这句话之后，这个推导结论就进入了上下文，下一步的预测建立在这个明确写出的正确基础上，而不是靠它在脑子里「暗中维持」这个中间状态。就像笔算，纸上的每一行数字都在帮你记住上一步算到哪了。

CoT 有两种触发方式。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zLRM1IicjS8guPV8xBE0W7JkWCsAZury4EoDsVRsOeapuibpe2aLESqqcmDJrgZIzVwZMlmdxAd2HicFOicQer9icU4clwlXftegHWTspDtLZY60/640?wx_fmt=png&from=appmsg)

• 一种是 Zero-shot CoT，直接在 prompt 末尾加上「让我们一步步思考」，LLM 自己展开推理，不需要额外示例；
• 另一种是 Few-shot CoT，给几个带有完整推理过程的例子让 LLM 模仿，效果更稳定，适合格式要求比较固定的场景。

但 CoT 有一个根本性的局限： **它是纯文字推理，没有办法和外部世界交互** 。推理过程再完整，也拿不到实时数据，不能执行计算，不能访问数据库。

如果你问 LLM「现在苹果公司的市值是多少？」，它只能根据训练数据里的知识回答，而那些知识可能已经过时好几个月了。这就是 CoT 不够用的地方，你需要的不只是一个能把推理写出来的 LLM，而是 **一个能在推理过程中「出去拿数据」「执行工具」再「回来继续推理」的系统。于是有了 ReAct** 。

### ReAct 是什么？

**ReAct** 是 Reasoning and Acting 的缩写，核心思路是在 CoT 的推理链里，插入真实的「行动」。

它让 LLM 按照「思考 -> 行动 -> 观察」这个循环来推进任务：先思考当前该怎么做，然后调用一个工具去获取信息或执行操作，把工具返回的结果作为新的「观察」接收回来，再进入下一轮思考，直到 LLM 判断任务完成。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zLRM1IicjS8hHANgN9nwD0iaJWGaCOtwVGLCoEEemxpG3rGn3I7icCC2kwrKiaFuFtjWuMaM1lKmQ7QrZJlGE6ic5VWb8DCO3bOM1Tm3ZiamQT66E/640?wx_fmt=png&from=appmsg)

用一个具体例子来感受这个循环。假设你问 Agent「2024 年苹果公司和谷歌的市值谁更高？差多少？」，如果只靠 CoT，LLM 只能说出它训练时知道的数字，可能已经不准了。但用 ReAct，整个过程会是这样的：

```
Thought: 这道题需要两家公司的实时市值数据，我得先查苹果的市值
Action: search
Action Input: 苹果公司 2024 年市值
Observation: 苹果公司 2024 年市值约为 3.5 万亿美元

Thought: 好，苹果的数字有了，再查谷歌的
Action: search
Action Input: 谷歌 2024 年市值
Observation: 谷歌 2024 年市值约为 2.1 万亿美元

Thought: 两个数字都有了，苹果 3.5 万亿，谷歌 2.1 万亿，苹果更高，差距是 1.4 万亿
Final Answer: 苹果公司 2024 年市值约 3.5 万亿美元，谷歌约 2.1 万亿美元，苹果更高，差距约 1.4 万亿美元
```

每一个 Thought 是 LLM 的推理，每一个 Action 是它决定调什么工具，每一个 Observation 是工具执行后系统填进去的真实结果，最后 Final Answer 是任务完成的终止信号。推理和真实数据的获取是交织在一起的，这才让 Agent 能处理「需要实时信息」或「需要执行操作」的任务。

**ReAct 的实现原理** ，是通过 prompt 格式来约束 LLM 的输出结构，但这个循环不是 LLM 自己在转，而是由你的代码来驱动的。

LLM 每次只做一件事：根据当前的历史，输出下一步的 Thought 加上 Action。你的代码负责检测它的输出，判断「有没有 Final Answer」，如果没有就解析出 Action、执行对应的工具、把工具结果作为 Observation 填回历史，再次调用 LLM，一轮一轮地转。

一个典型的 ReAct prompt 长这样：

```
你是一个 AI 助手，可以使用以下工具：
- search(query): 搜索互联网获取最新信息
- calculator(expr): 计算数学表达式

回答时请严格按照以下格式：
Thought: 你的思考过程（分析当前情况，决定下一步）
Action: 工具名称
Action Input: 工具的输入参数
Observation: （此行由系统填入工具返回的结果，你不用写）
... 以上可以重复多轮 ...
Final Answer: 当你确定可以回答时，在这里给出最终答案

问题：2024 年苹果公司的市值是多少？和谷歌相比谁更高？
```

然后你的代码跑一个循环，不断地「调 LLM、检查输出、执行工具、把结果填回去」：

```
def react_agent(question: str, tools: dict, max_steps: int = 10):
    # 把 ReAct 格式约束和问题拼在一起，作为初始 prompt
    prompt = build_react_prompt(question, tools)
    # 用来存每一轮的对话历史，每次调 LLM 都把完整历史带上
    history = []

    for _ in range(max_steps):
        # 调 LLM，让它输出下一步的 Thought + Action
        # 注意：每次调用都把完整历史拼进去，LLM 才知道之前做了什么
        response = llm.generate(prompt + "\n".join(history))

        if"Final Answer:"in response:
            # LLM 输出了 Final Answer，说明它判断任务完成了
            return response.split("Final Answer:")[-1].strip()

        # 从 LLM 输出里解析出 Action 名称和 Action Input
        # 例如：Action: search，Action Input: 苹果公司市值 -> ("search", "苹果公司市值")
        action, action_input = parse_action(response)

        # 执行对应的工具，拿到真实结果
        if action in tools:
            observation = tools[action](action_input)
        else:
            # 如果 LLM 填了一个不存在的工具名，给它一个错误反馈
            observation = f"工具 {action} 不存在，请选择可用工具"

        # 把这一轮的 LLM 输出（含 Thought+Action）和 Observation 都追加进历史
        # 下次调 LLM 时这些内容会成为它的「记忆」
        history.append(response)
        history.append(f"Observation: {observation}")

    return"超过最大步数，任务未完成"
```

整个 loop 里，真正的「智能」全在 LLM 每次输出的 Thought 里，它在分析当前情况、做出下一步决策。

你的代码框架做的事是：管理对话历史、执行工具、检测循环终止条件。两件事分工很清楚，理解了这个分工，ReAct 就不再神秘了。

需要补充一点：上面描述的是 ReAct 的 **经典实现** ，靠 prompt 格式约束加文本解析来驱动工具调用。

现代 LLM（GPT-4、Claude 3 之后）基本都原生支持 **Function Calling / Tool Use** ，模型可以直接输出结构化的 JSON 工具调用，不再需要靠解析 `Action: xxx` 这种文本格式。

这让 ReAct 的实现更干净，也更可靠，不容易因为 LLM 输出格式不规范而解析失败。本质上「思考 -> 行动 -> 观察」的循环没变，只是「行动」这一步从解析文本变成了解析结构化 JSON。

ReAct 的主要代价是 token 消耗随步骤数线性增长，因为每次调 LLM 都要把完整历史带上，任务步骤越多，输入就越长，成本可观。

推荐阅读：

[腾讯三面面试官刚想拿“Agent和Workflow 的区别”难倒我，我反手甩出一张架构对比图，他当场让我等 HR 面。](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483898&idx=1&sn=90101573e94272a1f4ceb6f3b5596758&scene=21#wechat_redirect)

[小红书二面秒挂！问“Agent 核心组件有哪些”，我答“大模型+Prompt”，面试官冷笑...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483788&idx=1&sn=45c1ec9d5a578a689d27179f06833b14&scene=21#wechat_redirect)

[美团面试官：“既然大模型已经这么强了，为什么还要做 Agent？”，我给出了满分回答，他眼睛亮了。](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483777&idx=1&sn=0fee73150fd3c92d666391a2e0797260&scene=21#wechat_redirect)

[字节二面：被问“大模型知识过时了怎么解？”，我答“微调”，面试官当场黑脸：“听说过 RAG 吗？”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483658&idx=1&sn=02673313e69b2a6a304ad7462e4e4a29&scene=21#wechat_redirect)

[腾讯面试官怒怼：“连 MCP 协议都没玩过，还敢在简历上写精通 Agent 开发？”，只会调包的我懵了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483673&idx=1&sn=0010e78aece8b05388b94d0396444a03&scene=21#wechat_redirect)

[阿里技术官拷问：“简历上写着精通 Agent 开发，那你说明白 A2A 和 MCP 到底啥区别？”，我：“呃... 不都是协议吗？”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483728&idx=1&sn=8feb775f055f697517d0a3ad424a8c7b&scene=21#wechat_redirect)

[腾讯面试官一句追问："做了这么多Agent项目，大模型怎么学会调用外部工具都说不清？" 我当场卡壳，二面凉了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483763&idx=1&sn=b3f2abee33ad0cf37282f8814b0960a5&scene=21#wechat_redirect)

💪面试突击资源推荐：

✅刷题闯关+模拟面试： 牛面Offer小程序

✅后端训练营：   
 ✅大模型训练营： [转行去做大模型开发了！](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247554501&idx=2&sn=85911a9a90cb323d397cf1ca65364d58&scene=21#wechat_redirect)   
 ✅做项目：





微信扫一扫   
 使用小程序

： ， ， ， ， ， ， ， ， ， ， ， ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言 收藏 听过 