# AI Engineering 架构师实战手册 (The Master Playbook)

**文档说明**：
* **用途**：这是您的长期学习路线图和知识库，旨在帮助您从零构建生产级 AI 系统。
* **核心方法论**：Infrastructure First（基建优先）、Context Economics（上下文经济学）、Data-Driven Eval（数据驱动评估）。
* **如何使用**：当您开启一个新的 AI 会话（Cursor/ChatGPT/Claude）时，**直接将本文档的相关章节粘贴给 AI**，并输入：“我是 AI 初学者，请根据这份文档的 [当前模块]，指导我进行开发。”

---

# Phase 1: 基础设施与核心链路 (Foundation)

## Module 1: 统一 LLM 网关 (The LLM Gateway)
**🎯 学习目标**：构建企业级的大模型调用层，解决模型碎片化、API 不稳定和输出格式不可控的问题。

### 1.1 核心原理 (Theory)
* [cite_start]**Factory Pattern**：通过工厂模式统一封装 OpenAI、DeepSeek 或 Claude 接口，实现配置与逻辑解耦，支持模型热切换 [cite: 82-108]。
* **Structured Output**：
    * [cite_start]**JSON vs TSV**：对于列表型数据，TSV (Tab-Separated Values) 比 JSON 节省 Token，且流式输出更友好 [cite: 459-465]。
    * [cite_start]**Sanitization**：大模型常输出不合法的 JSON（如包含 ` ```json ` 标记），需要编写清洗层 [cite: 453-455]。

### 1.2 架构规范 (Architecture Spec)
* **Interface**: `ILLMService`
    * `generate(messages: list, stream: bool) -> str`
    * `generate_structured(messages: list, schema: BaseModel) -> object`
* **Class**: `LLMFactory`
    * 支持配置 `provider`, `api_key`, `base_url`。
    * [cite_start]处理 DeepSeek 与 OpenAI 兼容接口的差异 [cite: 91-95]。

### 1.3 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "我们要开始 Module 1 开发。请基于 Python `pydantic` 和 `openai` 库：
> 1. 定义 `ILLMService` 抽象基类。
> 2. 实现 `OpenAIService`，支持重试机制（Retry）和错误处理。
> 3. 实现 `LLMFactory`，根据环境变量动态返回服务实例。
> 4. 编写一个 `parse_json_markdown` 工具函数，能自动去除字符串中的 markdown 代码块标记并解析 JSON。"

---

## Module 2: 主动搜索代理 (Search Agent)
**🎯 学习目标**：实现一个能自主探索代码库的 Agent（而非被动问答）。

### 2.1 核心原理 (Theory)
* **Agent Loop**：Agent 是一个 `While` 循环（思考 -> 决定工具 -> 执行 -> 观察 -> 再思考）。
* **Search Strategy**：
    * [cite_start]`ListDirectory`：查看目录结构（树形输出）[cite: 360-370]。
    * [cite_start]`Glob`：按文件名模式匹配 [cite: 324-336]。
    * [cite_start]`Grep`：按内容正则搜索 [cite: 293-305]。
    * [cite_start]`FileRead`：读取文件内容（支持 `offset/limit` 防止 Token 爆炸）[cite: 341-353]。

### 2.2 架构规范 (Architecture Spec)
* **Tool Interface**: `name`, `description`, `parameters`, `execute(**kwargs)`。
* [cite_start]**Tool Manager**: 负责注册工具，并将 Python 函数转换为 OpenAI Tools Schema [cite: 407-418]。

### 2.3 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 2：构建 Search Agent。
> 1. 定义标准的 `BaseTool` 类。
> 2. 实现以下工具：
>    - `ListDirectoryTool`: 输出树形结构（Tree Structure）的目录字符串。
>    - `GrepTool`: 封装 `ripgrep` 或 `grep` 命令，支持正则。
>    - `FileReadTool`: 读取文件，参数包含 `file_path`, `start_line`, `end_line`。
> 3. 实现一个 `AgentRunner` 类，包含一个 `run(user_query)` 循环，能够自动解析 LLM 的 `tool_calls` 并执行。"

---

## Module 3: 基础上下文管理 (Basic Context)
**🎯 学习目标**：让 Agent 拥有“短期记忆”，且不撑爆内存。

### 3.1 核心原理 (Theory)
* **Context Rot (上下文腐烂)**：历史消息越多，噪音越大，模型越笨。
* [cite_start]**Tool Pruning (工具裁剪)**：只保留**最近一轮**的工具调用详细结果。将旧的工具结果替换为 `[Tool output hidden]`，只保留 Agent 的总结 [cite: 173-186]。

### 3.2 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 3：上下文优化。
> 请为 `AgentRunner` 增加一个 `HistoryManager` 模块：
> 1. 维护 `messages` 列表。
> 2. 实现 `prune_tools()` 方法：遍历历史消息，找到所有非最新一轮的 `role='tool'` 消息，将其 content 替换为摘要占位符。
> 3. 确保 System Prompt 始终保留在第一位。"

---

# Phase 2: Pro 级架构与深度优化 (Mastery)

## Module 4: 动态自适应压缩 (Adaptive Compression)
**🎯 学习目标**：实现类似 ClaudeCode 的智能记忆管理。

### 4.1 核心原理 (Theory)
* **Adaptive Strategy**：根据 Token 占比动态决定策略。
    * **Middle-Out**：保留开头和结尾，切掉中间（适合长文档）。
    * [cite_start]**Oldest-Removal**：先进先出（适合闲聊）[cite: 199-225]。
* [cite_start]**XML State Snapshot**：不存流水账，而是让 AI 定期生成 `<state_snapshot>` XML，记录 `overall_goal`, `completed_tasks` [cite: 168-169]。

### 4.2 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 4：高级压缩。
> 1. 实现一个 `ContextCompressor` 类。
> 2. 编写逻辑：当 Token > 阈值（如 8000）时，触发压缩。
> 3. 实现 'XML Snapshot' 策略：调用 LLM 生成当前对话状态的 XML 摘要（包含 goals, done_tasks），并用此摘要替换旧的历史记录。"

---

## Module 5: 高级 RAG 与混合检索 (Advanced RAG)
**🎯 学习目标**：超越简单的“关键词搜索”，让 Agent 理解语义和概念。

### 5.1 核心原理 (Theory)
* **Contextual Retrieval (上下文补全)**：
    * *痛点*：代码切片后往往失去上下文。
    * *解法*：在存入向量库前，使用 LLM 为每个切片生成一段“背景描述”（Contextual Header）。
* **Parent Document Retrieval (父文档检索)**：
    * [cite_start]*策略*：检索时匹配精准的“小子块”，但在构建 Prompt 时，投喂该子块所属的完整“父文档块” [cite: 162]。
* **HyDE (假设性文档嵌入)**：
    * [cite_start]*策略*：不检索用户的问题，而是让 LLM 先生成一个假想的答案，然后去检索这个假想答案的向量 [cite: 163]。

### 5.2 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 5：高级检索。
> 1. 引入 `chromadb` 或 `faiss` 向量库。
> 2. 实现 'Contextual Indexer'：读取代码文件，按函数切分。对每个函数，调用 LLM 生成一句 'This function is used for X' 的注释，与源码合并后存入向量库。
> 3. 升级 Search Agent：新增 `SemanticSearchTool`。
> 4. 实现 'Hybrid Search'：同时执行 `GrepTool` (关键词) 和 `SemanticSearchTool` (向量)，使用 RRF 算法合并结果。"

---

## Module 6: 生产级高并发模式 (Production Patterns)
**🎯 学习目标**：高并发、低成本、高可用。

### 6.1 核心原理 (Theory)
* **Read-Through / Write-Through**：
    * 写：同时写 Redis 和 DB。
    * [cite_start]读：先读 Redis，没有读 DB 并回填 [cite: 226-246]。
* [cite_start]**Single-Flight**：防止缓存击穿。当多个请求同时查同一个失效 Key 时，只允许 1 个请求去查 DB [cite: 266-271]。

### 6.2 架构规范 (Architecture Spec)
* **Class**: `UnifiedStorage`
    * Backend: `RedisCache`, `FilePersistentStore`。
    * Features: TTL (Time To Live), SingleFlight 锁。

### 6.3 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 6：生产级存储。
> 1. 实现 `UnifiedStorage` 类，支持 `get`, `set`。
> 2. 集成 Redis（或使用内存字典模拟）。
> 3. 实现 'SingleFlight' 模式：确保同一时间对同一 Key 的数据库查询只有一个在执行。
> 4. 实现 'Read-Through' 缓存逻辑。"

---

## Module 7: 评估体系 (Evaluation)
**🎯 学习目标**：数据驱动优化。

### 7.1 核心原理 (Theory)
* [cite_start]**EventBus**: 解耦业务逻辑和监控。Agent 只管跑，EventBus 负责记录 `tool:call`, `agent:call` [cite: 13-29]。
* [cite_start]**Eval Metrics**: 计算 `Agent Match`（是否调用了正确的 Agent）和 `Tool Match`（是否调用了正确的工具）[cite: 52-64]。

### 7.2 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 7：评估系统。
> 1. 实现一个单例 `EventBus`。
> 2. 在 Agent 关键节点（Start, ToolCall, Result）埋点发射事件。
> 3. 编写一个测试脚本 `eval_agent.py`，运行一组预设的 Query（如 '查一下 redis 配置'），断言 Agent 是否调用了 `grep` 工具。"

---

## Module 8: 生态标准与 MCP (Ecosystem & MCP)
**🎯 学习目标**：不再闭门造车，让 Agent 连接外部世界（Github, Slack, Postgres）。

### 8.1 核心原理 (Theory)
* **MCP (Model Context Protocol)**：
    * [cite_start]*概念*：Anthropic 推出的开放标准。任何服务只要实现了 MCP Server 接口，你的 Agent 就能直接调用其工具，无需手动写 Tool Wrapper [cite: 280-288]。
    * *价值*：一次开发，连接所有数据源。

### 8.2 ⚡ Vibe Coding 任务指令 (Prompt for Cursor)
> "进入 Module 8：MCP 集成。
> 1. 修改 `ToolManager`，使其支持加载外部 MCP Server。
> 2. 使用 `mcp-sdk` (Python)，实现一个 `MCPClient`。
> 3. 任务：连接一个本地的 'FileSystem MCP Server' 或 'Git MCP Server'，让 Agent 可以直接操作 Git 提交代码，而不仅仅是修改文件。"

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
