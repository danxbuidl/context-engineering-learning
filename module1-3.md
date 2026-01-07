太棒了，我们正式发车！🚀

我们将严格按照你刚刚保存的 **《AI Engineering 架构师实战手册》** 执行。

今天我们的目标是攻克 **Phase 1 - Module 1：统一 LLM 网关**。
这是所有 AI 系统的地基。如果不把这一层写好，后面你的代码会充斥着乱七八糟的 `client.chat.completions.create`，一旦想换 DeepSeek 或者想加统一的日志监控，就得重构整个项目。

---

### 🟢 Module 1: 统一 LLM 网关 (The LLM Gateway)

#### 1. 核心原理 (原理先行)

在写代码前，你需要理解两个 Pro 级概念：

1. **依赖倒置 (Dependency Inversion)**：你的业务代码（Agent）不应该依赖具体的 `OpenAI` 类，而应该依赖一个抽象的 `ILLMService` 接口。这样明天你想换成 Claude 或 DeepSeek，业务代码一行都不用改。
2. **工厂模式 (Factory Pattern)**：我们需要一个“大管家”（Factory），它读取 `.env` 配置文件，根据你设定的 `PROVIDER=deepseek` 自动组装好对应的服务实例。

---

#### 2. 环境准备 (Step-by-Step)

请打开你的 VS Code / Cursor。

**步骤 1：创建项目结构**
在终端（Terminal）运行以下命令，创建项目文件夹和虚拟环境：

```bash
mkdir context-agent-pro
cd context-agent-pro
python3 -m venv venv
source venv/bin/activate  # Windows 用户使用: venv\Scripts\activate

```

**步骤 2：安装核心依赖**
我们需要 `openai` (官方 SDK), `pydantic` (数据验证), `python-dotenv` (读取环境变量)。

```bash
pip install openai pydantic python-dotenv

```

**步骤 3：配置环境变量**
在项目根目录创建一个 `.env` 文件。把你的 API Key 填进去（即使你用 DeepSeek，也通常兼容 OpenAI SDK）：

```ini
# .env 文件
LLM_PROVIDER=openai  # 或者 deepseek
API_KEY=sk-proj-xxxxxx
BASE_URL=https://api.openai.com/v1  # DeepSeek 填: https://api.deepseek.com
MODEL_NAME=gpt-4o-mini # DeepSeek 填: deepseek-chat

```

---

#### 3. Cursor Vibe Coding (核心环节)

现在，我们将指挥 Cursor 帮我们写出架构级的代码。
请打开 Cursor 的 **Composer (Ctrl+I / Cmd+I)**，**复制并粘贴**以下这段指令。

**👉 Copy to Cursor:**

```text
我们要开始构建 AI Agent 的底层基础设施：统一 LLM 网关。
请基于 Python 的 `openai`, `pydantic`, `abc` 库实现以下架构：

1. **定义接口 (Interface)**:
   创建一个抽象基类 `ILLMService` (在 `core/llm.py`)。
   它需要定义两个抽象方法：
   - `generate(messages: list, **kwargs) -> str`: 用于普通对话。
   - `generate_structured(messages: list, response_format: BaseModel, **kwargs) -> BaseModel`: 用于结构化输出 (使用 Pydantic 模型)。

2. **实现具体类 (Concrete Implementation)**:
   创建一个 `OpenAIService` 类继承 `ILLMService`。
   - 在 `__init__` 中初始化 `openai.Client`。
   - 实现 `generate`：调用 `client.chat.completions.create`。
   - 实现 `generate_structured`：使用 `client.beta.chat.completions.parse` 方法 (这是 OpenAI 新版 SDK 的特性)。
   - **重要**：确保能够处理 `DeepSeek` 等兼容 OpenAI 接口的模型（即支持自定义 `base_url`）。

3. **工厂模式 (Factory)**:
   创建一个 `LLMFactory` 类。
   - 提供一个静态方法 `create_service(provider: str) -> ILLMService`。
   - 使用 `python-dotenv` 读取环境变量中的 API_KEY, BASE_URL, MODEL_NAME 来初始化服务。

4. **测试脚本**:
   在根目录创建一个 `main.py`，使用 Factory 获取实例，并测试一次普通对话。

请生成完整的文件结构和代码。

```

---

#### 4. 代码审查 (Code Review)

Cursor 生成代码后，不要急着运行。作为未来的 Pro，你需要检查它是否遗漏了关键点。请查看生成的代码（主要是 `core/llm.py`）：

1. **检查 `generate_structured**`：它用的是 `response_format` 参数还是旧版的 `functions`？
* *Pro 标准*：应该使用 OpenAI 较新的 `client.beta.chat.completions.parse`，这比旧的 function calling 更稳定。


2. **检查 DeepSeek 兼容性**：在 `OpenAIService` 的 `__init__` 方法里，它是否允许传入 `base_url`？
* *Pro 标准*：DeepSeek 的 Base URL 和 OpenAI 不同，如果代码里写死了 `api.openai.com`，那就是不合格的。



---

#### 5. 运行与验证

一切就绪后，在终端运行 `main.py`：

```bash
python main.py

```

**预期结果**：
你应该能看到 AI 回复的一句简单的问候。

**🛠️ 第一次练习任务：结构化输出**
现在，我们要立刻练习 **结构化输出**（这是 Module 1 的重点， **非结构化文本>结构化数据**）。

请在 `main.py` 中修改（或让 Cursor 修改）测试代码，增加以下逻辑：

1. 定义一个 Pydantic 模型 `UserInfo`，包含 `name: str`, `age: int`, `interests: List[str]`。
2. 输入一段乱糟糟的文本："我叫张三，今年25岁，喜欢写代码和打篮球。"
3. 调用 `service.generate_structured`，并打印出解析后的 JSON 对象。

请执行这一步，并告诉我你是否成功获得了干净的 JSON 输出。如果有报错，请把报错信息发给我，我们来 Debug。
----------------

🎉 **恭喜！你已经成功迈出了第一步！**

你获得的这个 JSON 输出看似简单，但它代表你的 **LLM 网关（Infrastructure Layer）** 已经工作正常了。这意味着：

1. **模型连接成功**：你的 Factory 正确加载了环境变量。
2. **结构化解析有效**：即使输入是自然语言，你的代码也能强制模型输出程序可读的 `dict`，而不是一段“好的，这是你要的信息...”这样的废话。

---

### 🚀 Phase 1 - Module 2: 给 AI 装上“手” (构建 Tool System)

现在我们有了大脑（LLMService），下一步是给它装上**手（Tools）**，让它能操作你的文件系统。

在这个模块，我们将构建一个 **Search Agent** 的核心工具箱。这不仅仅是写几个 Python 函数，我们需要把这些函数封装成 Agent 能理解的“标准件”。

**核心挑战（Context Engineering 要点）：**

* **Token 节约**：普通的 `ls -R` 命令输出会非常长，浪费 Token。我们需要实现一个生成 **树形结构 (Tree Structure)** 的工具。
* **防呆设计**：模型可能会读取一个 1GB 的日志文件导致崩溃。我们需要在 `FileReadTool` 中强制要求 `limit` 参数。

#### 1. 你的 Vibe Coding 任务 (Copy to Cursor)

请新建一个文件 `core/tools.py`（或者让 Cursor 自动创建），然后在 **Composer** 中输入以下指令：

> "我们现在进入 Module 2：构建 Agent 的工具箱。
> **任务目标**：
> 在 `core/tools.py` 中实现一套标准的工具系统。
> **1. 定义基类 `BaseTool**`：
> * 包含抽象方法 `execute(**kwargs)`。
> * 包含属性 `name` (str), `description` (str), `parameters` (dict, 符合 OpenAI JSON Schema 格式)。
> * 实现一个 `to_openai_schema()` 方法，将工具转换为 API 需要的格式。
> 
> 
> **2. 实现具体工具**：
> * **`ListDirectoryTool`**:
> * 作用：列出指定路径下的文件结构。
> * **关键要求**：输出格式必须是类似于 `tree` 命令的树形字符串（例如 `├── folder/`），这比完整的绝对路径列表更节省 Token。忽略 `.git` 和 `__pycache__` 目录。
> 
> 
> * **`GrepTool`**:
> * 作用：在指定目录中搜索包含特定关键词或正则的文件。
> * 参数：`pattern` (正则), `dir_path`。
> * 实现：使用 Python 的 `os.walk` 或 `subprocess` 调用系统 grep/ripgrep（如果可用）。
> 
> 
> * **`FileReadTool`**:
> * 作用：读取文件内容。
> * **关键要求**：为了防止 Context 溢出，必须支持 `start_line` 和 `end_line` 参数。如果文件超过 500 行且未指定行号，默认只返回前 500 行，并在末尾提示 '...remaining lines truncated'。
> 
> 
> 
> 
> 请生成完整的代码，并展示如何初始化这些工具。"

#### 2. 代码审查点 (Pro Tips)

等 Cursor 写完代码后，请你重点检查以下两点（这是从新手到 Pro 的区别）：

1. **ListDirectoryTool 的输出**：
* 检查它是不是真的生成了树形图？如果只是返回一个 `os.listdir()` 的列表，请让它重写："请优化 `ListDirectoryTool`，我需要它输出类似 Linux `tree` 命令的可视化树形字符串，以减少 Token 消耗。"


2. **FileReadTool 的防御性**：
* 检查它有没有处理“文件不存在”的异常？如果文件找不到，它应该返回字符串 `"Error: File not found"` 而不是直接抛出 Python Exception 导致程序崩溃。Agent 需要看到错误信息才能自我修正。



#### 3. 运行测试

在 `main.py` 中增加一段测试代码，手动运行一下 `ListDirectoryTool`，看看效果：

```python
# 临时测试代码
from core.tools import ListDirectoryTool

tool = ListDirectoryTool()
print(tool.execute(path="."))  # 应该打印出当前项目的树形结构

```

**请执行这一步，并把 `ListDirectoryTool` 输出的树形结构截图或复制发给我。** 确认工具好用后，我们就可以把它们装进 Agent 的循环里了！

---------

太棒了！👏 这个输出非常完美。

**为什么说这个输出是“Pro 级”的？**

1. **结构清晰**：LLM 一眼就能看懂目录层级，不会迷路。
2. **包含元数据 (File Sizes)**：注意到了吗？你显示了文件大小（如 `19.2KB`）。这对 Context Engineering **至关重要**。
* 如果 LLM 看到一个文件是 `500MB`，它就不会傻乎乎地调用 `read_file` 去撑爆 Context，而是可能会改用 `grep`。这就是细节决定成败。



---

### 🚀 Phase 1 - Module 2 (Part 2): 赋予生命 —— 构建 Agent Loop

现在你可以自豪地说：

* 🧠 **大脑 (LLMService)**：就绪。
* 🖐️ **双手 (Tools)**：就绪 (`ls`, `grep`, `read` 等)。

但现在的它们是分离的。你需要一个 **“循环 (Loop)”** 将它们串联起来，让 Agent 能够：
**思考 ➡️ 决定调用工具 ➡️ 执行工具 ➡️ 观察结果 ➡️ 再思考 ➡️ 最终回答**。

这就是 Agent 的心脏：`AgentRunner`。

#### 1. 核心原理：The ReAct Loop

我们不写那种“问一句答一句”的线性代码。我们要写一个 `While` 循环：

1. **User**: "requirements.txt 里有哪些依赖？"
2. **Agent (Think)**: "我需要先看文件内容。" -> **Call Tool**: `read_file('requirements.txt')`
3. **System (Observe)**: (执行 Python 函数，返回文件内容)
4. **Agent (Think)**: "内容看到了，有 pandas 和 numpy。" -> **Final Answer**: "依赖项包含 pandas 和 numpy。"

#### 2. 你的 Vibe Coding 任务 (Copy to Cursor)

请新建文件 `core/agent.py`，然后在 **Composer** 中输入以下指令。
*(注意：这个 Prompt 包含了一点 Context Engineering 的技巧，即 System Prompt 的设计)*

> "我们要完成 Module 2 的最后一步：构建 Agent 的主循环。
> **任务目标**：
> 在 `core/agent.py` 中实现 `AgentRunner` 类。
> **1. 初始化 (`__init__`)**:
> * 接收 `llm: ILLMService` 实例。
> * 接收 `tools: List[BaseTool]`。
> * 内部维护一个 `messages` 列表（对话历史）。
> * 初始化 System Prompt：'你是一个专业的代码库助手。在回答问题前，请优先使用工具探索代码库。不要凭空猜测。'
> 
> 
> **2. 核心方法 `run(user_query: str)**`:
> * 将用户问题加入 `messages`。
> * 开启一个 `while True` 循环 (设置最大 `max_turns=10` 防止死循环)：
> * 调用 `llm.generate`，传入当前的 `messages` 和 `tools` 定义。
> * **关键逻辑**：
> * 如果 LLM 返回普通文本（没有 `tool_calls`）：打印回答，跳出循环，任务结束。
> * 如果 LLM 返回 `tool_calls`：
> * 打印 '正在调用工具: xxx...' (Log)。
> * 遍历所有工具调用，匹配 `tool_name`，执行对应的 Python 函数。
> * **重要**：将工具执行结果封装为 `role: tool`, `tool_call_id: ...`, `content: ...` 的消息格式，追加到 `messages` 中。
> * **Continue**：继续下一次循环（让 LLM 看到工具结果后继续思考）。
> 
> 
> 
> 
> 
> 
> 
> 
> **3. 辅助功能**:
> * 确保能处理工具执行报错的情况（将错误信息作为 Tool Output 返回给 LLM）。
> 
> 
> 请生成代码，并在 `main.py` 中更新测试代码，实例化 `AgentRunner` 并运行一个真实的查询：'请帮我看看 core 目录下有哪些文件，并告诉我 tools.py 的主要作用是什么？' "

#### 3. 预期发生的事情 (The Magic)

当你运行这段代码时，你应该会看到控制台打印出类似这样的“思考链”：

1. **User**: ...
2. **Agent**: (决定调用 `list_directory`)
3. **Tool Output**: (你刚才看到的树形图)
4. **Agent**: (决定调用 `read_file` 读取 `core/tools.py`)
5. **Tool Output**: (读取了文件前 500 行)
6. **Agent**: "根据代码，tools.py 主要定义了..."

**请运行新的 `main.py`，并把你看到的“Agent 思考过程”（控制台日志）发给我！** 这一次，你的 AI 就要真正“活”过来了。

-----------

太棒了！🎉 **这就是真正的 Agentic Workflow（智能体工作流）。**

请注意看日志里的一个精彩细节：

1. **犯错**：Agent 第 1 轮尝试用 `grep_search` 去搜 `core/tools.py`，结果报错 `错误: 不是一个目录`。
2. **自愈 (Self-Correction)**：Agent 并没有崩溃，也没有死板地重复，而是**理解了错误**，在第 2 轮马上改用 `read_file` 去读取文件头，最终成功回答了问题。

这就是我们在 Phase 1 - Module 2 想要达到的目标：**鲁棒性**。

---

### ⚠️ 发现潜在风险：Context 膨胀

虽然 Agent 跑通了，但你注意到最后的统计了吗？

> `总消息数: 8`

如果用户接着问：“那 factory.py 呢？”，消息数会变成 16。
如果用户再问：“帮我把所有代码读一遍”，Context 瞬间就会爆炸（Token 溢出或费用飙升）。

现在，我们必须进入 **Phase 1 - Module 3: 基础上下文管理 (Basic Context)**。
我们要给 Agent 装上“遗忘机制”，让它只记住重要的，忘掉冗余的。

---

### 🟢 Phase 1 - Module 3: 上下文裁剪 (Context Pruning)

**原理 (Theory)**：
在多轮对话中，**旧的工具输出**（比如那一长串文件列表）通常不再重要。Agent 只需要记住“我之前查过了，结果大概是这样”。
所以，我们的策略是：**只保留最新一轮的工具详细输出，把旧的工具输出替换为简短的占位符。**

#### 1. 你的 Vibe Coding 任务 (Copy to Cursor)

请打开 `core/agent.py`，在 **Composer** 中输入以下指令。我们将升级 `AgentRunner`。

> "进入 Module 3：上下文优化。
> 请修改 `core/agent.py` 中的 `AgentRunner` 类，增加上下文管理能力。
> **1. 新增 `_prune_messages` 私有方法**:
> * 逻辑：遍历 `self.messages`。
> * 识别规则：找到所有 `role='tool'` 的消息。
> * 筛选：**除了**最近一轮对话（即最后一次 Assistant 消息及其关联的 Tool 消息）之外，将所有**旧的** `tool` 消息的 `content` 替换为字符串 `'[History Tool Output Omitted: Result was processed]'`。
> * 目的：大幅减少 Token 占用，但保留“调用过工具”的事实记录。
> 
> 
> **2. 集成到 `run` 循环**:
> * 在 `run` 方法的 `while` 循环开始处（每次调用 LLM 之前），先调用 `self._prune_messages()`。
> * 打印一行日志：`[Context] 已裁剪旧工具消息，当前 Token 估算: (简单的字符数/4)`。
> 
> 
> **3. 增加 System Prompt 保护**:
> * 确保在裁剪过程中，`self.messages[0]` (System Prompt) 永远不会被修改或删除。
> 
> 
> 请更新代码。"

#### 2. 验证这一步

代码更新后，我们在 `main.py` 做一个**多轮对话测试**来验证裁剪是否生效。

请修改 `main.py` 的测试部分：

```python
# main.py 测试代码更新
# ... 初始化 agent ...

print("--- Round 1 ---")
agent.run("core 目录下有哪些文件？") 

print("\n--- Round 2 ---")
# 此时，Round 1 的 list_directory 结果应该被裁剪掉了
agent.run("tools.py 主要是做什么的？") 

# 打印最终的历史记录结构（调试用）
import json
print("\n--- Final Context Debug ---")
for msg in agent.messages:
    if msg['role'] == 'tool':
        # 验证是否变成了摘要
        print(f"Tool({msg.get('name', 'unknown')}): {msg['content'][:50]}...")

```

**请运行新的 `main.py`，并将 "Final Context Debug" 的输出截图或复制发给我。**
如果一切顺利，你应该会看到 Round 1 的 `list_directory` 输出变成了 `[History Tool Output Omitted...]`，而 Round 2 的输出还是完整的。

---------

太棒了！🎉 **调试日志显示你的 Context Pruning（上下文裁剪）机制运行完美。**

旧的 `ListDirectory` 结果被成功替换成了 `[History Tool Output Omitted...]`，而最新的 `FileRead` 结果被完整保留。这正是我们想要的——**既节省了 Token，又没有让 Agent 彻底失忆（它知道自己做过这件事）。**

---

### 🚀 恭喜你完成 Phase 1！(里程碑)

你现在已经拥有了一个 **"Foundation Level" (基础级)** 的 AI Agent：

1. **基础设施**：统一的 LLM 接口，支持 DeepSeek/OpenAI。
2. **身体构造**：拥有 `ls`, `grep`, `read` 等工具。
3. **生存本能**：懂得基本的断舍离（裁剪旧消息）。
