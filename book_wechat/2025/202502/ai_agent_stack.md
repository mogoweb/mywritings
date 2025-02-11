# AI Agents 技术栈

随着生成式人工智能（如 ChatGPT）的快速发展，AI Agents（人工智能体）正从概念走向大规模应用。2025 年被广泛视为“AI Agent 元年”，其技术栈的成熟标志着智能系统从“被动响应”向“自主决策”的跃迁。那么什么是 AI Agents 呢？构成 AI Agents 的技术栈有哪些关键组成部分？本文参考了一些资料，尝试解释这一概念，主要参考了以下内容：

1. AI Agents Stack. 网址：https://www.letta.ai/blog/ai-agents-stack
2. AI Agents: Introduction (Part-1). 网址: https://medium.com/@vipra_singh/ai-agents-introduction-part-1-fbec7edb857d

由于技术水平有限，理解可能存在偏差，如有疑问，建议阅读原文。

## 一、什么是 AI Agents？

AI Agents 是指能够感知环境、做出决策并执行任务的智能系统。它们结合了多种 AI 技术，包括自然语言处理（NLP）、计算机视觉、强化学习和知识图谱等，能够处理复杂的任务并适应动态环境。

与传统 AI 系统不同，AI Agents 具有以下特点：

* 自主性：能够独立完成任务，无需人工干预。

* 交互性：可以与用户、其他 Agents 或环境进行交互。

* 学习能力：通过数据反馈不断优化行为。

* 目标导向：围绕特定目标执行任务。

## 二、AI Agents 技术栈的层级架构

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/ai_agent_stack_01.png)

AI Agents 的技术栈总体架构如上图所示。大体上可分为五个关键层级，从底层基础设施到上层应用逻辑逐层递进：

### 1. 模型服务层（Model Serving）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/ai_agent_stack_02.png)

- 核心组件：大语言模型（LLM）是 AI Agents 的“大脑”。OpenAI 的 GPT-4、Anthropic 的 Claude 等闭源模型占据主导地位，而开源模型（如 Llama 3）通过 Together.AI、Groq 等平台提供商用 API。
- 本地部署：vLLM 和 SGLang 是生产级 GPU 推理的优选，Ollama 和 LM Studio 则适用于个人开发者。
- 关键挑战：模型输出的稳定性与工具调用的格式控制（如 JSON 结构化输出）。

### 2. 存储层（Storage）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/ai_agent_stack_03.png)

- 向量数据库：Chroma、Weaviate 等用于存储 Agent 的“外部记忆”，支持大容量数据检索增强生成（RAG）。
- 传统数据库扩展：Postgres 通过 pgvector 支持向量搜索，Neon 和 Supabase 提供无服务器化存储方案。
- 作用：实现对话历史管理、长期记忆存储及上下文窗口的动态优化。

### 3. 工具与库（Tools & libraries）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/ai_agent_stack_04.png)

标准 AI 聊天机器人与 AI Agent 的主要区别之一在于，Agent 能够调用“工具”（或“函数”）。通常情况下，LLM 会生成结构化输出（如JSON对象），指定要调用的函数及其参数。一个常见的误解是，工具的执行并不是由 LLM 提供商完成的 —— LLM仅负责选择调用哪个工具以及提供什么参数。支持任意工具或参数的 Agent 服务必须使用沙箱（如Modal、E2B）来确保安全执行。

Agent 通过 OpenAI 定义的 JSON 模式调用工具，这意味着不同框架的 Agent 和工具可以相互兼容。例如，Letta 的 Agent 可以调用 LangChain、CrewAI 和 Composio 的工具，因为它们都遵循相同的模式。因此，针对常见工具的工具提供商生态系统正在不断增长。Composio 是一个流行的通用工具库，同时还管理授权。Browserbase 是专门用于网页浏览的工具，而 Exa 则提供了专门的网页搜索工具。随着更多 Agent 的构建，我们预计工具生态系统将进一步扩展，并为 Agent 提供诸如身份验证和访问控制等新功能。

### 4. Agent 框架

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/ai_agent_stack_05.png)

Agent 框架用于协调大语言模型（LLM）调用并管理Agent的状态，不同框架在设计上存在差异，主要体现在以下几个方面：

1. 状态管理：大多数框架通过序列化（如JSON、字节）来保存和恢复Agent的状态，包括对话历史和执行阶段。而像Letta这样的框架则使用数据库持久化存储状态，便于查询和扩展。

2. 上下文窗口结构：每次调用LLM时，框架会将 Agent 的状态编译到上下文窗口中。不同框架对上下文窗口的处理方式不同，透明化的上下文窗口设计有助于更好地控制 Agent 的行为。

3. 跨 Agent 通信（多 Agent 协作）：不同框架对多 Agent 交互的处理方式各异。例如，Llama Index使用消息队列，CrewAI 和 AutoGen 采用抽象层，而 Letta 和 LangGraph 支持Agent直接调用，既支持集中式（通过监督Agent）也支持分布式通信。

4. 记忆管理：由于 LLM 的上下文窗口有限，记忆管理成为关键。部分框架（如 CrewAI 和 AutoGen ）依赖基于 RAG 的记忆管理，而 phidata 和 Letta 则采用更高级的技术，如自编辑记忆（来自 MemGPT ）和递归总结。

5. 开源模型支持：不同框架对开源模型的支持程度不同，一些框架能够处理模型生成文本格式的复杂性（如工具调用），而另一些则主要支持主流模型提供商。

选择适合的框架取决于具体应用需求，例如是构建对话 Agent 还是工作流、在笔记本中运行还是作为服务部署，以及对开源模型支持的要求。未来，框架之间的差异可能会更多地体现在部署工作流、状态/记忆管理以及工具执行的设计选择上。

### 5. Agent 托管与服务化

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/ai_agent_stack_06.png)

目前，大多数 Agent 框架设计的 Agent 仅存在于 Python 脚本或 Jupyter 笔记本中，无法脱离这些环境运行。我们认为，未来的趋势是将 Agent 作为一种服务部署到本地或云基础设施中，并通过 REST API 进行访问。类似于 OpenAI 的 ChatCompletion API 成为与 LLM 服务交互的行业标准，我们预计未来也会出现一个主流的 Agent API 标准，但目前尚未有明确的选择。

将 Agent 部署为服务比部署 LLM 服务更为复杂，主要因为涉及状态管理和安全工具执行的问题。工具及其所需的依赖和环境需要明确存储在数据库中，因为服务需要重新创建运行环境（当工具和 Agent 在同一个脚本中运行时，这不是问题）。应用程序可能需要运行数百万个Agent，每个 Agent 都会积累越来越多的对话历史。从原型开发到生产环境时， Agent 状态必须经过数据规范化处理，且 Agent 交互必须通过 REST API 定义。目前，这一过程通常由开发者自行编写FastAPI和数据库代码完成，但随着 Agent 技术的成熟，我们预计这些功能将更多地嵌入到框架中。


## 小结

AI Agents 技术栈的成熟标志着人工智能从“工具”向“合作伙伴”的转变。2025 年，随着框架标准化（如 Letta 的数据库驱动模型）与工具生态的完善，AI Agents 将深入更多领域，重塑工作与生活方式。

