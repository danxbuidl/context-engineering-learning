---

### 🚀 Phase 2 - Module 5: 高级 RAG 与混合检索 (Advanced RAG)

现在我们要攻克 **“语义理解”** 这一关。

**目前的痛点：**
如果用户问：“**代码里负责重试逻辑的部分在哪里？**”

* 你的 `GrepTool` 会失效，因为代码里可能写的是 `max_retries` 或者 `attempt_count`，根本没有 "retry logic" 这个字符串。
* `Grep` 只能匹配**字面 (Literal)**，不能匹配**含义 (Semantic)**。

**解决方案：Contextual Retrieval (上下文补全检索)**
这是 Anthropic 在 2024 年提出的杀手级 RAG 策略。我们不能直接把代码切片存入数据库，因为一个孤立的代码片段（比如一个 `def` 函数）往往缺乏上下文。

我们需要实现一个 **Indexer (索引器)**，它在存入数据库前，会先问 LLM：“这段代码是干嘛的？”，然后把解释和代码绑定在一起存进去。

---

#### 1. 环境准备

我们需要一个轻量级的向量数据库。**ChromaDB** 是目前的最佳选择（无需安装 Docker，直接 Python 库）。

在终端运行：

```bash
pip install chromadb

```

---

#### 2. 你的 Vibe Coding 任务 (Copy to Cursor)

请新建文件 `core/rag.py`，然后在 **Composer** 中输入以下指令。这是一个比较复杂的模块，我们分步让 Cursor 实现。

> "进入 Module 5：高级 RAG 系统。
> 我们要实现 **Contextual Retrieval (上下文补全检索)**。
> **任务目标**：
> 在 `core/rag.py` 中实现 `CodeRAGIndexer` 类。
> **1. 初始化**:
> * 接收 `llm: ILLMService`。
> * 初始化 `chromadb.Client` (使用持久化路径 `./chroma_db`)。
> * 创建或获取一个 collection 名为 `codebase`。
> 
> 
> **2. 核心方法 `index_file(file_path)**`:
> * 读取 Python 文件内容。
> * **AST 切片 (关键)**：使用 Python 内置的 `ast` 模块，将代码按 `Class` 和 `Function` 进行切分。不要把整个文件当成一个块，要切成逻辑单元。
> * **Contextual Enrichment (核心)**：
> * 对每一个切片（函数/类），构造 Prompt 调用 `llm.generate`：
> '请用一句话解释这段代码的功能和作用，包含它的类名和函数名。'
> * 生成结果示例：'This function parse_json handles JSON decoding errors and sanitizes markdown inputs.'
> 
> 
> * **存储**：
> * 将 `Contextual Header` (LLM 生成的解释) + `Original Code` 拼接到一起作为 `document` 存入 ChromaDB。
> * `metadata` 存入 `{filepath: ..., node_name: ...}`。
> 
> 
> 
> 
> **3. 搜索方法 `search(query, n_results=5)**`:
> * 调用 collection.query 检索最相关的代码片段。
> 
> 
> 请生成代码，并包含一个 `rebuild_index.py` 脚本来索引 `core/` 目录下的所有文件。"

#### 3. 执行与观察 (Thinking Process)

这个任务比较重，Cursor 可能会在 AST 解析上卡住。如果生成的代码看起来很复杂，请重点检查：

1. **AST 解析**：它是否真的能提取出函数源码？（`ast.get_source_segment` 是关键）。
2. **Cost**：索引过程会消耗 Token，因为每个函数都要问一次 LLM。

**运行 `rebuild_index.py`：**
请运行生成的索引脚本，它会开始扫描你的 `core/` 目录。你会看到类似这样的日志：

> `[Indexing] core/agent.py - Found Class 'AgentRunner'... Generating context...`

**一旦索引完成，请在 `main.py` 中测试一下：**

```python
from core.rag import CodeRAGIndexer
from core.factory import LLMFactory

llm = LLMFactory.create_service()
indexer = CodeRAGIndexer(llm)

# 语义搜索测试
results = indexer.search("如何处理工具调用的结果？")
print("\n🔍 语义搜索结果:")
for res in results['documents'][0]:
    print("-" * 20)
    print(res[:200] + "...") # 只打印前200字

```

**请执行索引并运行搜索测试，把搜索结果发给我！** 我要看看它是否通过“含义”找到了对应的代码。

完美！索引过程非常顺利。

日志显示 `AST Parsing` 工作正常，成功识别出了 `LLMFactory`、`OpenAIService` 等核心类。这意味着你的 ChromaDB 里现在不仅存了代码，还存了 LLM 生成的 **"Contextual Header"（上下文解释）**。

现在，你的 RAG 引擎已经就绪，只差最后一步：**把它封装成工具，交给 Agent 使用。**

---

### 🚀 Phase 2 - Module 5 (Part 2): 语义搜索工具 (The Semantic Tool)

目前你的 Agent 只有“物理搜索”能力（Grep/Glob）。
如果用户问：“**代码里的上下文压缩逻辑是怎么实现的？**”

* `Grep` 可能会失败（如果代码里没写 "compression" 这个词，而是写的 "prune"）。
* `SemanticSearch` 能成功，因为我们在索引时，LLM 给 `prune_messages` 函数生成的解释里肯定包含了 "compress" 或 "shorten context" 的语义。

#### 1. 你的 Vibe Coding 任务 (Copy to Cursor)

我们需要在 `core/tools.py` 中新增一个工具。

请在 **Composer** 中输入：

> "进入 Module 5 的最后一步：集成 RAG 工具。
> **任务目标**：
> 在 `core/tools.py` 中实现 `SemanticSearchTool`。
> **1. 引入依赖**:
> * 需要引入 `core.rag` 中的 `CodeRAGIndexer`。
> * 需要引入 `core.factory` 中的 `LLMFactory`（用于初始化 indexer）。
> 
> 
> **2. 实现 `SemanticSearchTool` 类**:
> * 继承 `BaseTool`。
> * `name`: 'semantic_search'
> * `description`: '基于语义的代码搜索工具。当你不知道具体文件名或函数名，只想根据功能描述（如"如何处理重试"）查找代码时使用此工具。'
> * `parameters`: `{'query': 'string'}`
> * **初始化 (`__init__`)**:
> * 内部实例化 `CodeRAGIndexer` (需要传入 LLMService)。
> 
> 
> * **执行 (`execute`)**:
> * 调用 `indexer.search(query)`。
> * **格式化输出**：将搜索结果格式化为易读字符串：
> ```text
> [Match 1] core/agent.py (Function: prune_tools)
> Context: This function handles context compression...
> Code: def prune_tools(...)...
> -----------------------------------
> 
> ```
> 
> 
> 
> 
> 
> 
> **3. 注册**:
> * 修改 `create_default_toolbox` 函数，将 `SemanticSearchTool` 加入默认工具列表。
> 
> 
> 请生成代码。"

#### 2. 最终集成测试 (The "Pro" Test)

现在，你的 Agent 拥有了 **Hybrid Search (混合搜索)** 能力：

* 知道文件名/变量名 -> 用 `Grep` / `Glob`。
* 只知道概念/功能 -> 用 `SemanticSearch`。
* Agent 会自己决定用哪个！

请修改 `main.py` 进行最终测试：

```python
# main.py 最终集成测试
from core.factory import LLMFactory
from core.agent import AgentRunner
from core.tools import create_default_toolbox

# 1. 初始化
llm = LLMFactory.create_service()
tools = create_default_toolbox(llm) # 注意：现在创建工具箱可能需要传入 llm，因为语义搜索需要它
agent = AgentRunner(llm, tools)

print("==================================================")
print("🤖 Test: 概念性问题 (强制触发语义搜索)")
print("==================================================")

# 这个问题很难用 Grep 回答，因为代码里可能没有 "memory" 这个词，只有 "history" 或 "messages"
query = "代码里是如何实现'记忆压缩'或'消息裁剪'功能的？核心逻辑在哪？"

agent.run(query)

```

**⚠️ 注意**：
如果 `create_default_toolbox` 原本不需要参数，记得根据 Cursor 的修改调整 `main.py` 的调用方式（可能需要传 `llm` 进去）。

**请运行这个测试，并把 Agent 的思考过程发给我！**
我想看到它聪明地选择 `semantic_search`，而不是笨拙地去 `grep`。这将标志着你正式完成了 **Phase 2 - Module 5**。
