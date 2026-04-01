# Hooks 机制：自定义 Agent 行为

## 本章目标

- 理解 Hooks 的概念和作用
- 了解 Claude Code 的 Hooks 工作原理
- 学会编写自定义 Hook
- 掌握常用 Hooks 场景

---

## 什么是 Hooks？

**Hook = 钩子 = 在特定时机插入的自定义逻辑**

Hooks 让你在 Claude Code 执行过程中"钩入"自定义代码，改变或扩展默认行为。

```
普通流程：
用户输入 → Claude Code 处理 → 输出

Hooks 流程：
用户输入 → [Before Hook] → Claude Code 处理 → [After Hook] → 输出
```

---

## Hooks vs Tools

| 特性 | Tools | Hooks |
|------|-------|-------|
| 触发方式 | LLM 决定调用 | 事件自动触发 |
| 执行者 | Agent | Claude Code |
| 用途 | 执行任务 | 拦截/修改行为 |
| 返回值 | 结果给 LLM | 影响后续流程 |

---

## Claude Code 的 Hooks 类型

### 可用 Hooks

| Hook | 触发时机 | 常见用途 |
|------|----------|----------|
| `pre_tool_use` | 工具执行前 | 验证参数、记录日志 |
| `post_tool_use` | 工具执行后 | 处理结果、修改输出 |
| `pre_task` | 任务开始前 | 准备环境、检查前置条件 |
| `post_task` | 任务完成后 | 清理资源、发送通知 |
| `on_tool_error` | 工具出错时 | 错误处理、降级方案 |

---

## Hooks 实现示例

### 基本结构

```javascript
// .claude/hooks/
// hooks.md 中定义钩子

## pre_tool_use

当工具执行前触发。

你可以检查：
- 工具名称和参数
- 当前上下文

你可以做：
- 记录日志
- 修改参数
- 阻止执行
- 添加提示

---
```

### Python 实现示例

```python
# hooks_impl.py
"""
Claude Code Hooks 实现示例
"""

class HooksManager:
    def __init__(self):
        self.hooks = {
            "pre_tool_use": [],
            "post_tool_use": [],
            "on_tool_error": []
        }

    def register(self, event: str, hook_func: callable):
        """注册钩子"""
        if event in self.hooks:
            self.hooks[event].append(hook_func)

    async def trigger(self, event: str, context: dict) -> dict:
        """触发钩子链"""
        results = []

        for hook in self.hooks.get(event, []):
            result = await hook(context)
            results.append(result)

            # 如果钩子返回阻止信号，停止执行
            if result.get("blocked"):
                return result

        return {"allowed": True, "results": results}

# ============ 常用钩子实现 ============

async def log_tool_use(context: dict) -> dict:
    """记录工具使用日志"""
    tool_name = context.get("tool_name")
    args = context.get("arguments", {})

    print(f"[LOG] 工具调用: {tool_name}")
    print(f"[LOG] 参数: {args}")

    return {"allowed": True}

async def validate_file_path(context: dict) -> dict:
    """验证文件操作路径，防止越界"""
    tool_name = context.get("tool_name")
    args = context.get("arguments", {})

    # 只检查文件操作
    if tool_name in ["Read", "Write", "Edit", "Bash"]:
        path = args.get("file_path") or args.get("command", "")

        # 简单检查：路径不能包含 ..
        if ".." in path:
            return {
                "blocked": True,
                "message": "禁止路径遍历攻击"
            }

    return {"allowed": True}

async def block_dangerous_commands(context: dict) -> dict:
    """阻止危险命令"""
    tool_name = context.get("tool_name")
    args = context.get("arguments", {})

    if tool_name == "Bash":
        command = args.get("command", "").lower()

        # 危险命令关键词
        danger_patterns = [
            "rm -rf /",
            "format c:",
            "del /f /s /q",
            ":(){ :|:& };:",  # Fork bomb
        ]

        for pattern in danger_patterns:
            if pattern in command:
                return {
                    "blocked": True,
                    "message": f"危险命令已阻止: {pattern}"
                }

    return {"allowed": True}

# ============ 使用示例 ============

async def main():
    manager = HooksManager()

    # 注册钩子
    manager.register("pre_tool_use", log_tool_use)
    manager.register("pre_tool_use", validate_file_path)
    manager.register("pre_tool_use", block_dangerous_commands)

    # 触发钩子
    context = {
        "tool_name": "Bash",
        "arguments": {"command": "rm -rf /tmp/test"}
    }

    result = await manager.trigger("pre_tool_use", context)

    if result.get("blocked"):
        print(f"操作被阻止: {result.get('message')}")
    else:
        print("允许执行")

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

## 实用 Hooks 场景

### 场景 1：自动备份文件

```python
async def backup_before_edit(context: dict) -> dict:
    """编辑文件前自动备份"""
    if context["tool_name"] == "Edit":
        import shutil
        from datetime import datetime

        file_path = context["arguments"]["file_path"]

        # 创建备份
        backup_path = f"{file_path}.backup.{datetime.now().strftime('%Y%m%d%H%M%S')}"
        shutil.copy(file_path, backup_path)

        print(f"[BACKUP] 已备份到: {backup_path}")

    return {"allowed": True}
```

### 场景 2：敏感信息检查

```python
async def check_secrets(context: dict) -> dict:
    """检查敏感信息泄露"""
    if context["tool_name"] in ["Write", "Edit"]:
        content = context["arguments"].get("content", "")

        secret_patterns = [
            (r'api[_-]?key["\']?\s*[:=]\s*["\']?[a-zA-Z0-9]{20,}', "API Key"),
            (r'password["\']?\s*[:=]\s*["\']?[^"\s]{8,}', "Password"),
            (r'sk-[a-zA-Z0-9]{20,}', "OpenAI Secret Key"),
        ]

        import re
        for pattern, secret_type in secret_patterns:
            if re.search(pattern, content, re.IGNORECASE):
                return {
                    "blocked": True,
                    "message": f"检测到可能的 {secret_type}，请使用环境变量"
                }

    return {"allowed": True}
```

### 场景 3：自动生成文档

```python
async def auto_document(context: dict) -> dict:
    """修改代码后自动更新文档"""
    if context["tool_name"] == "Edit":
        file_path = context["arguments"]["file_path"]

        # 如果是 Python 文件，更新 docstring
        if file_path.endswith(".py"):
            print(f"[DOC] 建议更新 {file_path} 的文档")

    return {"allowed": True}
```

---

## Claude Code 官方 Hooks 配置

### 配置文件位置

```
项目根目录/
├── CLAUDE.md           # 项目配置
├── .claude/
│   └── hooks/
│       └── hooks.md    # Hooks 定义
```

### hooks.md 示例

```markdown
# Claude Code Hooks Configuration

## pre_tool_use

When: Every tool execution
Check: Validate file paths are within project directory

```javascript
export function pre_tool_use({ tool_name, tool_args }) {
  // 检查路径安全
  if (tool_args.file_path) {
    const path = tool_args.file_path;
    if (path.includes("..") || path.startsWith("/etc")) {
      return {
        blocked: true,
        message: "不允许访问项目外的路径"
      };
    }
  }
}
```
```

---

## Hooks 执行顺序

```
用户输入
    │
    ▼
┌─────────────────────┐
│  pre_task Hooks     │  ← 任务开始前的处理
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Claude 处理        │
└──────────┬──────────┘
           │
           ▼
    ┌──────────────┐
    │ Tool 1       │
    ├──────────────┤
    │ pre_tool     │──→ 阻止
    ├──────────────┤
    │ 执行         │
    ├──────────────┤
    │ post_tool    │
    └───────┬──────┘
           │
           ▼
    ┌──────────────┐
    │ Tool 2       │
    │ ...          │
    └───────┬──────┘
           │
           ▼
┌─────────────────────┐
│  post_task Hooks   │  ← 任务完成后的处理
└──────────┬──────────┘
           │
           ▼
        输出
```

---

## 练一练

1. 编写一个 Hook，在每次执行 `Bash` 命令前记录日志
2. 编写一个 Hook，检查文件写入是否包含 `TODO` 注释
3. 设计一个 Hook，自动在每次编辑后更新 CHANGELOG

---

## 小测验

**1. Hook 的主要作用是？**
- A. 执行具体任务
- B. 在特定时机插入自定义逻辑
- C. 存储数据

**2. `pre_tool_use` 触发于？**
- A. 任务开始前
- B. 工具执行前
- C. 工具执行后

**3. 如何阻止一个工具执行？**
- A. 删除工具
- B. Hook 返回 `blocked: true`
- C. 关闭电脑

---

## 答案

1-B | 2-B | 3-B

---

---

## 本章小结

**L5 工程化学习**带你从"使用者"进化为"构建者"：

| 主题 | 核心知识点 |
|------|----------|
| Agent 架构 | Agent = LLM(大脑) + Tools(手脚) + Memory(记忆) |
| ReAct 循环 | Thought → Action → Observation → 循环直到完成 |
| 最小 Agent | while loop + 工具调用 + 结果评估 = 自主 Agent |
| MCP 协议 | 通过标准化协议连接外部服务，实现能力扩展 |
| Hooks 机制 | 在工具执行前后插入自定义逻辑，实现安全审计和自动化 |

**ReAct 循环图示**：
```
┌────────────────────────────────────┐
│  Thought: 分析当前情况，决定下一步   │
├────────────────────────────────────┤
│  Action: 执行工具（Read/Write/Bash）│
├────────────────────────────────────┤
│  Observation: 获取执行结果          │
├────────────────────────────────────┤
│  判断：任务完成？ → 否 → 回到 Thought│
│       → 是 → 返回最终答案            │
└────────────────────────────────────┘
```

**本章重点记住**：L5 不仅是"学习"源码，更是理解 AI 系统的工作原理。理解了 Agent 架构，你就能设计自己的 AI 工作流，而不是只会"调用"现成功能。

---

## 下一步

- [回到 L4：精通技巧](../L4-精通技巧/08-hooks.md)
- [L6 实践经验](../L6-实践经验/01-高效使用模式.md)
