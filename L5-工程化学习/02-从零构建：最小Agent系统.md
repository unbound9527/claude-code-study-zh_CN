# 从零构建：最小 Agent 系统

## 本章目标

- 理解 Agent 的核心组件
- 用 Python 从零构建一个可运行的 Agent
- 实现 ReAct 循环
- 添加基本的 Memory 机制
- 掌握构建 Agent 的基本思路

---

## Agent 的四大组件

```
Agent = LLM（大脑）+ Tools（手脚）+ Memory（记忆）+ Loop（循环）
```

| 组件 | 作用 | 类比 |
|------|------|------|
| LLM | 理解意图、做出决策 | 大脑 |
| Tools | 执行具体操作 | 手脚 |
| Memory | 存储对话历史 | 记忆 |
| Loop | 控制执行流程 | 神经系统 |

---

## 项目结构

```
mini-agent/
├── agent.py          # Agent 主类
├── tools.py          # 工具定义
├── memory.py         # 记忆管理
└── main.py           # 入口
```

---

## 第一步：定义 Tools

### tools.py

```python
"""
工具定义模块
每个工具包含：名称、描述、参数、执行函数
"""

from typing import Callable

# 工具注册表
REGISTRY: dict[str, dict] = {}

def tool(name: str, description: str):
    """装饰器：注册工具"""
    def decorator(func: Callable):
        REGISTRY[name] = {
            "name": name,
            "description": description,
            "func": func
        }
        return func
    return decorator

# ============ 内置工具 ============

@tool(
    name="calculator",
    description="进行数学计算。使用格式：表达式如 2+2、10*5 等"
)
def calculator(expression: str) -> str:
    """计算数学表达式"""
    try:
        # 安全计算（只用 eval 做简单演示，生产环境不要用）
        result = eval(expression, {"__builtins__": {}}, {})
        return str(result)
    except Exception as e:
        return f"计算错误: {e}"

@tool(
    name="search",
    description="搜索信息。需要关键词作为参数"
)
def search(keyword: str) -> str:
    """模拟搜索功能"""
    # 这里可以接入真实搜索 API
    knowledge = {
        "python": "Python 是一种高级编程语言，创始人是 Guido van Rossum",
        "人工智能": "AI 是 Artificial Intelligence 的缩写，指人造的智能系统",
        "agent": "Agent 是能够自主决策和执行任务的 AI 系统"
    }
    return knowledge.get(keyword, f"没有找到关于 '{keyword}' 的信息")

@tool(
    name="get_time",
    description="获取当前时间"
)
def get_time() -> str:
    """获取当前时间"""
    from datetime import datetime
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# ============ 工具列表 ============

def get_available_tools() -> list[dict]:
    """获取所有可用工具"""
    return [
        {
            "name": name,
            "description": info["description"]
        }
        for name, info in REGISTRY.items()
    ]
```

---

## 第二步：实现 Memory

### memory.py

```python
"""
记忆管理模块
管理对话历史和上下文窗口
"""

from typing import list

class Memory:
    def __init__(self, max_tokens: int = 4000):
        self.max_tokens = max_tokens
        self.messages: list[dict] = []
        self.token_count = 0

    def add(self, role: str, content: str):
        """添加消息"""
        self.messages.append({"role": role, "content": content})
        # 简单估算 token 数（实际应用中需要精确计算）
        self.token_count += len(content) // 4

    def get_messages(self) -> list[dict]:
        """获取消息列表，自动压缩如果太长"""
        if self.token_count > self.max_tokens:
            self.compact()
        return self.messages

    def compact(self):
        """压缩记忆：保留最近的和系统提示"""
        # 保留第一条（系统提示）和最后 N 条
        system_msg = self.messages[0] if self.messages else None
        recent = self.messages[-4:]  # 保留最近 4 条

        self.messages = [system_msg] + recent if system_msg else recent
        self.token_count = sum(len(m["content"]) // 4 for m in self.messages)

    def clear(self):
        """清空记忆"""
        self.messages = []
        self.token_count = 0
```

---

## 第三步：构建 Agent

### agent.py

```python
"""
Agent 主模块
实现 ReAct 循环
"""

from .tools import REGISTRY, get_available_tools
from .memory import Memory

class Agent:
    def __init__(self, llm_client):
        """
        初始化 Agent

        Args:
            llm_client: LLM 客户端（需要实现 complete 方法）
        """
        self.llm = llm_client
        self.memory = Memory()

    def set_system_prompt(self, prompt: str):
        """设置系统提示"""
        self.memory.add("system", prompt)

    def think(self, user_input: str) -> str:
        """
        核心方法：处理用户输入，执行 ReAct 循环

        ReAct 循环：
        1. Thought - LLM 思考需要什么
        2. Action - 执行工具
        3. Observation - 获取结果
        4. 重复直到完成
        """
        # 添加用户消息
        self.memory.add("user", user_input)

        max_iterations = 10  # 防止无限循环
        iteration = 0

        while iteration < max_iterations:
            iteration += 1

            # 1. 调用 LLM 获取响应
            messages = self.memory.get_messages()
            tools = get_available_tools()

            response = self.llm.complete(messages, tools)

            # 2. 检查是否包含工具调用
            if "tool_calls" in response:
                tool_results = []

                for tool_call in response["tool_calls"]:
                    tool_name = tool_call["name"]
                    tool_args = tool_call["arguments"]

                    # 3. 执行工具
                    if tool_name in REGISTRY:
                        result = REGISTRY[tool_name]["func"](**tool_args)
                    else:
                        result = f"未知工具: {tool_name}"

                    tool_results.append({
                        "tool": tool_name,
                        "result": result
                    })

                    # 4. 添加到记忆
                    self.memory.add(
                        "tool",
                        f"[{tool_name}] {result}"
                    )

                # 打印执行过程（方便观察）
                for tr in tool_results:
                    print(f"🔧 执行: {tr['tool']}")
                    print(f"   结果: {tr['result']}\n")

            else:
                # 5. 没有工具调用，直接返回响应
                final_answer = response.get("content", "")
                self.memory.add("assistant", final_answer)
                return final_answer

        return "抱歉，问题太复杂，我无法在限制次数内完成。"

    def reset(self):
        """重置 Agent，清空记忆"""
        self.memory.clear()
```

---

## 第四步：创建 LLM 客户端

### llm_client.py

```python
"""
LLM 客户端
支持多种后端
"""

import anthropic

class ClaudeClient:
    """Anthropic Claude API 客户端"""

    def __init__(self, api_key: str, model: str = "claude-sonnet-4-20250514"):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.model = model

    def complete(self, messages: list[dict], tools: list[dict]) -> dict:
        """调用 Claude API"""

        # 转换消息格式
        api_messages = []
        for msg in messages:
            if msg["role"] == "system":
                continue  # system 消息单独处理
            api_messages.append(msg)

        response = self.client.messages.create(
            model=self.model,
            max_tokens=1024,
            system=messages[0]["content"] if messages and messages[0]["role"] == "system" else "",
            messages=api_messages,
            tools=[{
                "name": t["name"],
                "description": t["description"],
                "input_schema": {"type": "object", "properties": {}}
            } for t in tools]
        )

        # 解析响应
        result = {
            "content": ""
        }

        for block in response.content:
            if block.type == "text":
                result["content"] = block.text
            elif block.type == "tool_use":
                result["tool_calls"] = [{
                    "name": block.name,
                    "arguments": block.input
                }]

        return result


class MockClient:
    """测试用 Mock 客户端"""

    def complete(self, messages: list[dict], tools: list[dict]) -> dict:
        """返回模拟响应"""

        user_msg = ""
        for msg in reversed(messages):
            if msg["role"] == "user":
                user_msg = msg["content"]
                break

        # 简单规则：如果用户说"计算"就调用计算器
        if "计算" in user_msg or "算" in user_msg:
            # 提取数字和运算符
            import re
            nums = re.findall(r'\d+', user_msg)
            ops = re.findall(r'[+\-*/]', user_msg)

            if nums and len(nums) >= 2:
                expr = f"{nums[0]}{ops[0] if ops else '+'}{nums[1]}"
                return {
                    "tool_calls": [{
                        "name": "calculator",
                        "arguments": {"expression": expr}
                    }]
                }

        return {
            "content": f"我理解了你的问题：{user_msg}\n\n这是一个测试回复。"
        }
```

---

## 第五步：主程序入口

### main.py

```python
"""
Mini Agent - 入口文件
"""

from agent import Agent
from llm_client import ClaudeClient, MockClient

def main():
    # 选择客户端（测试用 Mock，生产用 Claude）
    # llm = MockClient()
    llm = ClaudeClient(api_key="your-api-key")

    # 创建 Agent
    agent = Agent(llm)

    # 设置系统提示
    agent.set_system_prompt(
        """你是一个有帮助的 AI 助手。
你有以下工具可以使用：
- calculator: 进行数学计算
- search: 搜索信息
- get_time: 获取当前时间

当需要使用工具时，明确说明要使用的工具和参数。"""
    )

    print("=" * 50)
    print("Mini Agent 已启动！输入 'quit' 退出")
    print("=" * 50)

    # 对话循环
    while True:
        user_input = input("\n你: ").strip()

        if user_input.lower() in ["quit", "退出"]:
            print("再见！")
            break

        if not user_input:
            continue

        response = agent.think(user_input)
        print(f"\nAgent: {response}")

if __name__ == "__main__":
    main()
```

---

## 运行测试

### 安装依赖

```bash
pip install anthropic
```

### 运行

```bash
python main.py
```

### 测试对话

```
你: 2+2 等于多少？
Agent: 🔧 执行: calculator
   结果: 4

Agent: 2+2 等于 4。

你: 现在几点了？
Agent: 🔧 执行: get_time
   结果: 2026-04-01 10:30:00

Agent: 现在是 2026-04-01 10:30:00。

你: 给我讲讲 Python 是什么
Agent: Python 是一种高级编程语言，创始人是 Guido van Rossum...
```

---

## 扩展练习

### 初级扩展

1. **添加新工具**：实现一个 `read_file` 工具读取文件
2. **改进 Memory**：使用更精确的 token 计算
3. **错误处理**：添加超时和重试机制

### 中级扩展

1. **流式输出**：支持打字机效果的流式响应
2. **多模态**：添加图片理解能力
3. **持久化**：保存对话历史到文件

### 高级扩展

1. **Agent Teams**：实现多 Agent 协作
2. **MCP 支持**：接入 MCP 协议连接外部服务
3. **自定义 LLM**：支持本地部署的 LLM（如 Ollama）

---

## 完整代码结构

```
mini-agent/
├── __init__.py
├── agent.py        # Agent 主类（ReAct 循环）
├── tools.py        # 工具定义和注册
├── memory.py       # 记忆管理
├── llm_client.py   # LLM 客户端（Claude / Mock）
└── main.py         # 入口
```

---

## 小测验

**1. Agent 的四大组件是？**
- A. 输入、输出、存储、显示
- B. LLM、Tools、Memory、Loop
- C. 用户、助手、文件、网络

**2. ReAct 循环中的 "Thought" 是什么？**
- A. 用户的问题
- B. LLM 分析情况并决定下一步
- C. 工具的执行结果

**3. Memory 的主要作用是？**
- A. 存储用户密码
- B. 管理对话历史和上下文
- C. 保存文件

---

## 答案

1-B | 2-B | 3-B

---

## 下一步

- [MCP 协议：扩展 Agent 能力](03-MCP协议：扩展Agent能力.md)
