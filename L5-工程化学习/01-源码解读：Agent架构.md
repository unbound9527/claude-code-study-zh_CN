# 源码解读：Claude Code Agent 架构

## 本章目标

- 了解 Claude Code 源码泄露事件
- 理解 Claude Code 的核心架构
- 学会阅读源码学习 Agent 实现
- 理解 Tool Executor、ReAct 循环的代码原理

---

## 源码泄露事件

### 发生了什么

2024年底，Claude Code 的内部源码被意外泄露到公共平台。这次泄露让开发者社区有机会深入了解 Claude Code 的实际实现。

### 泄露的内容

泄露的源码揭示了 Claude Code 的核心架构：
- Agent 循环机制
- Tool 定义和执行
- 上下文管理
- 与 Claude API 的交互方式

### 这意味着什么

> 以前我们只能猜测 Claude Code 怎么工作，现在可以直接看它的"配方"了。

---

## Claude Code 核心架构

### 架构总览

```
┌─────────────────────────────────────────────────────┐
│                   Claude Code                        │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐               │
│  │   User      │───►│   Parser    │               │
│  │   Input     │    │   (解析)     │               │
│  └─────────────┘    └──────┬──────┘               │
│                            ▼                        │
│  ┌─────────────────────────────────────────────┐   │
│  │              Agent Loop (Agent 循环)        │   │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐  │   │
│  │  │ Claude  │───►│ Planner │───►│ Tool    │  │   │
│  │  │ (大脑)   │    │ (规划)   │    │ Executor│  │   │
│  │  └─────────┘    └─────────┘    │ (执行)   │  │   │
│  │                              └────┬────┘  │   │
│  └────────────────────────────────────┼───────┘   │
│                                       ▼            │
│  ┌─────────────────────────────────────────────┐   │
│  │              Memory (记忆)                   │   │
│  │         对话历史 + 上下文管理                 │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### 各组件职责

| 组件 | 职责 | 代码中的位置 |
|------|------|-------------|
| Parser | 解析用户输入，提取意图 | input processing |
| Agent Loop | 核心循环：思考→决策→执行 | main loop |
| Planner | 决定使用哪个工具 | decision logic |
| Tool Executor | 执行具体操作 | tool handlers |
| Memory | 管理对话历史和上下文 | context management |

---

## Tool Executor 详解

### 源码中的工具定义

Claude Code 的工具定义采用结构化方式：

```python
# 伪代码示例（基于泄露源码重构）
class Tool:
    name: str           # 工具名称
    description: str    # 工具描述（让 LLM 理解何时使用）
    parameters: dict    # 参数定义
    handler: callable   # 实际执行函数

# 内置工具示例
TOOLS = [
    Tool(
        name="Read",
        description="读取文件内容",
        parameters={"file_path": "string"},
        handler=read_file
    ),
    Tool(
        name="Write",
        description="写入文件内容",
        parameters={"file_path": "string", "content": "string"},
        handler=write_file
    ),
    Tool(
        name="Bash",
        description="执行命令行",
        parameters={"command": "string"},
        handler=execute_command
    ),
    # ... 更多工具
]
```

### 工具调用流程

```
1. LLM 决定需要使用工具
   ↓
2. 构造工具调用请求
   {
     "name": "Read",
     "parameters": {"file_path": "src/app.js"}
   }
   ↓
3. Tool Executor 查找并调用对应 handler
   ↓
4. Handler 执行实际操作
   ↓
5. 将结果返回给 LLM
```

---

## ReAct 循环的代码实现

### 循环结构

```python
# 伪代码示例（基于泄露源码重构）
async def agent_loop(user_input: str, tools: list[Tool]):
    # 1. 初始化
    messages = [{"role": "user", "content": user_input}]
    memory = Memory()

    while True:
        # 2. LLM 思考
        response = await claude.complete(messages)

        # 3. 解析 LLM 响应
        if response.stop_reason == "end_turn":
            # 直接回答用户，循环结束
            return response.content

        # 4. 提取工具调用
        tool_calls = response.content.get("tool_calls", [])

        for tool_call in tool_calls:
            # 5. 查找工具
            tool = find_tool(tool_call.name, tools)

            # 6. 执行工具
            result = await tool.handler(**tool_call.arguments)

            # 7. 添加到记忆
            memory.add(tool_call, result)
            messages.append({
                "role": "tool",
                "content": result
            })

        # 8. 检查是否应该停止
        if should_stop(memory):
            return format_response(memory)
```

### 关键设计决策

| 设计 | 说明 |
|------|------|
| 异步执行 | 工具调用是异步的，提高效率 |
| 循环终止条件 | LLM 决定何时停止（end_turn） |
| 错误处理 | 工具失败时允许重试或跳过 |
| 上下文窗口 | 受限于 LLM 的上下文长度 |

---

## 上下文管理实现

### 泄露源码中的上下文策略

```python
# 伪代码示例
class ContextManager:
    def __init__(self, max_tokens: int):
        self.max_tokens = max_tokens
        self.messages = []

    def add(self, message: dict):
        self.messages.append(message)

        # 如果超出上下文限制，进行压缩
        if self.count_tokens() > self.max_tokens * 0.8:
            self.compact()

    def compact(self):
        """上下文压缩策略"""
        # 1. 保留系统提示
        # 2. 保留最近 N 条对话
        # 3. 压缩或删除中间的历史
        self.messages = self.summarize(self.messages)
```

### 压缩策略

| 策略 | 何时使用 | 效果 |
|------|----------|------|
| 保留最近 | 对话较长时 | 遗忘早期对话 |
| 摘要 | 需要保留概览时 | 丢失细节 |
| 删除低价值 | 高频重复操作 | 丢失具体步骤 |

---

## 如何阅读源码学习

### 学习路径

1. **从入口开始**：找到 main.py 或 index.ts
2. **追踪流程**：跟随用户输入看代码流程
3. **理解抽象**：每个组件的接口是什么
4. **深入实现**：具体算法和数据结构

### 阅读技巧

```
┌────────────────────────────────────────┐
│  1. 先看注释和文档字符串                │
│     了解代码意图而非实现细节             │
├────────────────────────────────────────┤
│  2. 用 "断点调试" 代替 "通读代码"        │
│     设置断点，跟踪实际执行流程           │
├────────────────────────────────────────┤
│  3. 画流程图而非抄代码                 │
│     理解架构比记住代码更重要             │
├────────────────────────────────────────┤
│  4. 带着问题阅读                       │
│     "这个函数什么时候被调用？"           │
│     "返回值是什么格式？"                 │
└────────────────────────────────────────┘
```

### 推荐的学习顺序

1. Agent Loop（理解主流程）
2. Tool Executor（理解工具调用）
3. Memory（理解上下文管理）
4. Parser（理解输入处理）

---

## 练一练

1. 分析 Claude Code 的 Tool 定义，找出至少 5 个内置工具
2. 画出一个请求从输入到输出的完整流程图
3. 思考：如果要添加一个新工具，需要修改哪些部分？

---

## 小测验

**1. Claude Code 的核心循环是？**
- A. Input → Output 直接响应
- B. Think → Act → Observe 的循环
- C. Request → Response 单次交互

**2. Tool Executor 的作用是？**
- A. 解析用户输入
- B. 执行具体的文件操作、命令等
- C. 管理对话历史

**3. 上下文压缩发生在什么时候？**
- A. 每次对话开始
- B. 上下文接近上限时
- C. 对话结束时

---

## 答案

1-B | 2-B | 3-B

---

## 下一步

- [从零构建：最小 Agent 系统](02-从零构建：最小Agent系统.md)
