### 🔴 Phase 2: Mastery (精进阶段) —— 让 Agent 拥有 "超级大脑"

现在我们要进入 **Phase 2 - Module 4: 动态自适应压缩 (Adaptive Compression)**。

**为什么要进阶？**
目前的裁剪策略太简单粗暴（只保留最近一轮）。如果我问：“第一步查到的文件名里，那个以 `f` 开头的是什么？”

* **现状**：Agent 会回答“我看不到具体文件名了，因为被裁剪了”。❌
* **Pro 级目标**：Agent 能回答“根据之前的记录，虽然详细列表已折叠，但我记得状态快照里提到了 `factory.py`。” ✅

我们将实现 **"XML State Snapshot" (状态快照)** 策略。这是 Gemini 和 ClaudeCode 等高级 Agent 都在用的技术。

#### 1. 核心原理：State Snapshot

我们不再简单地“删除”或“占位”，而是**调用一个小模型（或者 Agent 自己）**，把过去的对话历史**压缩**成一段结构化的 XML 摘要。

**变换前：**

> User: 查 core 目录
> Tool: [file1, file2, file3...]
> Agent: 找到了 file1...
> User: 查 file1
> Tool: [content of file1...]

**变换后 (Context 结构)：**

> **System**: ...
> **User**: (Hidden prompt) 请总结当前状态。
> **Assistant**:
> ```xml
> <state_snapshot>
>   <completed_tasks>
>     1. Listed 'core' directory.
>     2. Read 'factory.py'.
>   </completed_tasks>
>   <key_findings>
>     - 'core' contains agent.py, tools.py.
>     - 'factory.py' implements LLMFactory class.
>   </key_findings>
>   <current_goal>Analyze specific files.</current_goal>
> </state_snapshot>
> 
> ```
> 
> 
> **User**: (New Query) 那 tools.py 呢？

---

#### 2. 你的 Vibe Coding 任务 (Copy to Cursor)

请新建文件 `core/compression.py`，然后在 **Composer** 中输入以下指令。

> "进入 Module 4：高级上下文压缩。
> 我们要实现一个基于 LLM 的 **State Snapshot** 压缩器。
> **任务目标**：
> 在 `core/compression.py` 中实现 `ContextCompressor` 类。
> **1. 初始化**:
> * 接收 `llm: ILLMService`。
> * 设置 `token_threshold` (默认 2000，方便测试)。
> 
> 
> **2. 核心方法 `compress_history(messages)**`:
> * **检测**：计算 `messages` 的总字符数（简单模拟 Token 数）。如果低于阈值，直接返回原列表。
> * **执行压缩**：
> * 提取当前所有的对话历史（除了 System Prompt）。
> * 构造一个专门的 Prompt，发给 LLM：
> '请阅读上述对话历史，生成一个简短的 XML 状态快照。
> 格式要求：
> <state>
> <completed_tasks>...</completed_tasks>
> <knowledge_graph>提取的关键信息点</knowledge_graph>
> <next_plan>...</next_plan>
> </state>'
> * 调用 `llm.generate` 获取摘要。
> 
> 
> * **重组 Context**：
> * 新的 `messages` = [System Prompt] + [User Message: 'System Notification: Previous context summary'] + [Assistant Message: (LLM 生成的 XML)] + [最近 2 轮的原始对话]。
> 
> 
> 
> 
> **3. 集成**：
> * 请告诉我如何在 `AgentRunner.run` 循环中集成这个 compressor，替换掉之前的简单 `prune` 逻辑。
> 
> 
> 请生成代码。"

#### 3. 执行与验证

Cursor 生成代码后，它可能会提示你修改 `core/agent.py`。
请确保在 `agent.py` 中：

1. 实例化 `ContextCompressor`。
2. 在 `run` 循环里，把 `self._prune_messages()` 替换为 `self.messages = self.compressor.compress_history(self.messages)`。

**测试步骤：**
为了触发压缩（假设阈值设为 2000 字符），我们需要进行一场“长对话”。在 `main.py` 中：

```python
# main.py 测试
# 强制把阈值设低一点，方便触发
from core.compression import ContextCompressor
agent.compressor = ContextCompressor(agent.llm, token_threshold=500) # 设为 500 字符就触发

print("--- Round 1: List Files (产生大量 Token) ---")
agent.run("请详细列出 core 目录下的文件，并读取 agent.py 的前 50 行。")

print("\n--- Round 2: Trigger Compression ---")
agent.run("总结一下你刚才看到了什么？")

# 验证压缩结果
print("\n--- Context Debug ---")
for msg in agent.messages:
    if "<state>" in str(msg.get('content', '')):
        print("🎉 成功生成 XML 快照:\n", msg['content'])

```

**请运行代码，并把生成的 `<state>...</state>` XML 快照发给我！** 一旦看到这个，你就掌握了 Gemini/Claude 背后的记忆原理。

