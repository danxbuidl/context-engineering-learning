# AI Engineering 进阶之路：从架构到实战 (Context Engineering & Agent Architecture)

**目标**：掌握构建生产级（Production-Level）AI 应用的核心能力，从单纯的 Prompt 调用进阶为能设计高可用、低成本、高智能 Agent 系统的 AI 架构师。

**核心方法论**：
- **Infrastructure First**：不写面条代码，先搭工厂和接口。
- **Context Economics**：把 Token 当作金钱来管理（压缩、缓存）。
- **Data-Driven Eval**：不靠感觉，靠评估系统说话。

---

## 🟢 第一阶段：入门 · 核心链路 (Foundation Phase)
**目标**：构建一个稳健、可运行的代码库搜索 Agent，跑通 LLM 调用、工具使用和基础上下文管理的闭环。

### Module 1: 基础设施层 (The Infrastructure)
* **核心任务**：构建统一的 LLM 网关。
* **关键技术点**：
    * [cite_start]**Factory Pattern**：统一封装 OpenAI/DeepSeek/Claude 接口，实现模型热切换 [cite: 82-108]。
    * **Structured Output Pipeline**：实现“JSON 修复器”，处理 Markdown 包裹、转义错误等 Dirty Data。
    * **Error Handling**：API 重试机制与错误回退策略。
* **交付物**：`LLMService` 基类与 `LLMFactory` 工厂类。

### Module 2: 主动搜索代理 (Search Agent Loop)
* **核心任务**：让 AI 学会使用工具探索代码库。
* **关键技术点**：
    * [cite_start]**Tool Implementation**：实现 `GrepTool` (正则搜索), `GlobTool` (文件查找), `ListDirectoryTool` (树形结构输出) [cite: 293-370]。
    * **Agent Loop**：构建 `While` 循环，实现 "思考-工具调用-结果处理" 的自主决策流。
    * **Context Efficiency**：优化工具输出格式（如树形目录）以节省 Token。
* **交付物**：一个能自主回答 "requirements.txt 里有哪些依赖" 的 CLI Agent。

### Module 3: 基础上下文管理 (Basic Context Management)
* **核心任务**：防止 Token 爆炸。
* **关键技术点**：
    * [cite_start]**Tool Pruning (工具裁剪)**：只保留最近一轮工具输出，历史工具调用压缩为摘要 [cite: 173-198]。
    * **Sliding Window**：基础的滑动窗口策略。
* **交付物**：支持多轮对话不报错的 `ContextManager`。

---

## 🔴 第二阶段：精进 · Pro 级架构 (Pro Phase)
**目标**：引入高阶优化技术，解决“慢、笨、贵”的问题，对标业界顶尖实践（如 ClaudeCode, Gemini）。

### Module 4: 动态自适应压缩 (Adaptive Compression)
* **核心任务**：智能决定“遗忘什么”。
* **关键技术点**：
    * [cite_start]**Adaptive Strategy**：基于“置信度”算法，动态选择“中间移除”或“最旧移除”策略 [cite: 199-225]。
    * **System 2 Attention (S2A)**：实现去噪中间件，用小模型清洗 Context 中的无关信息。
    * [cite_start]**State Snapshot**：复刻 Gemini 的 `<scratchpad>` 和 `<state_snapshot>` XML 摘要策略 [cite: 168-172]。

### Module 5: 高级 RAG 与混合检索 (Advanced RAG)
* **核心任务**：提升检索准确率与语义理解。
* **关键技术点**：
    * **Contextual Retrieval**：切片前为每个 Chunk 生成背景说明（Contextual Header）。
    * [cite_start]**Parent Document Retrieval**：检索子块，投喂父文档，保证上下文连贯 [cite: 161-162]。
    * [cite_start]**HyDE**：假设性文档嵌入，提升概念性问题的检索效果 [cite: 161]。

### Module 6: 生产级高并发模式 (Production Patterns)
* **核心任务**：高并发下的成本与性能优化。
* **关键技术点**：
    * [cite_start]**Redis Caching**：实现 Write-Through（写穿）和 Read-Through（读穿）的多级缓存 [cite: 226-265]。
    * [cite_start]**Single-flight**：实现同进程去重，防止缓存击穿导致的数据库雪崩 [cite: 266-271]。

### Module 7: 评估与生态 (Evaluation & Ecosystem)
* **核心任务**：数据化衡量 Agent 表现。
* **关键技术点**：
    * [cite_start]**EventBus**：实现事件总线，解耦监控与业务逻辑 [cite: 13-29]。
    * [cite_start]**Automated Eval**：编写测试用例，自动计算 Tool Selection 的准确率与召回率 [cite: 52-75]。
    * [cite_start]**MCP Protocol**：(扩展) 实现 Model Context Protocol 客户端，连接外部生态工具 [cite: 280-288]。

## 🏗️ 通过这份学习，你能具备从 0 搭建的系统性能力吗？
答案是肯定的。
这不仅仅是一份“教程”，它实际上是带你复刻一个微型的企业级 AI 框架。

为什么说它是“系统性”的？因为我们从这份 wakeup-jin-practical-guide-to-context-engineering.txt 文档中提取的不仅仅是代码片段，而是一套完整的工程闭环。

完成这份计划后，你将拥有以下 5 个维度的从 0 到 1 的搭建能力：

(1) 架构设计能力 (Architecture)
你不再是写一个几百行的 main.py 脚本，而是能搭建分层架构。

你将学会：如何通过 工厂模式 (Factory Pattern)  统一管理 OpenAI、DeepSeek 等不同模型，实现配置与逻辑解耦。
系统性体现：你的系统可以随时更换底层模型，而无需重写业务逻辑。

(2) 扩展性能力 (Scalability)
你不仅能写死几个工具，还能搭建通用的插件系统。

你将学会：定义标准的 Tool 接口 ，并构建 ToolManager  来自动注册、查找和执行工具。
系统性体现：当需要增加新功能（比如增加一个“发邮件”的工具）时，你只需要写一个新类，系统会自动加载，无需修改核心循环。

(3) 鲁棒性与成本控制 (Reliability & Cost)
你将学会如何让系统在生产环境中稳定、便宜地运行。

你将学会：构建 Redis 缓存层 ，实现“读穿/写穿”策略，并利用 Token 压缩策略  处理超长上下文。
系统性体现：你的系统不会因为用户聊了 100 句就崩溃（Context 溢出），也不会因为重复提问而浪费 API 费用。

(4) 智能决策能力 (Agentic Reasoning)
你将超越简单的“一问一答”，构建能自主决策的 Agent。

你将学会：构建 Search Agent ，让 LLM 在 while 循环中自主决定“下一步做什么” 。
系统性体现：你具备了构建 Copilot 级别应用的核心逻辑——让 AI 像人类工程师一样查文档、读代码、解决问题。

(5) 质量保障能力 (QA & Evaluation)
你知道如何证明你的系统是好的。

你将学会：搭建 EventBus (事件总线) 来解耦监控，并编写 评估函数  来自动化测试 Agent 的准确率。
系统性体现：你拥有了数据驱动迭代的能力，这是从“Demo 开发者”通过“Pro 工程师”的关键门槛。

总结： 这份计划涵盖了 接口层 (Interface) -> 逻辑层 (Agent Loop) -> 工具层 (Tools) -> 数据层 (Cache) -> 监控层 (Eval)。这就是一个 AI 应用的全系统生命周期。
