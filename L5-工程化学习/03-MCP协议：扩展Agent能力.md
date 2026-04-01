# MCP 协议：扩展 Agent 能力

## 本章目标

- 理解 MCP（Model Context Protocol）协议
- 了解 MCP 的工作原理
- 学会构建自己的 MCP Server
- 掌握用 MCP 连接外部工具

---

## 什么是 MCP？

**MCP = Model Context Protocol（模型上下文协议）**

MCP 是一种标准协议，让 AI 模型能够连接外部数据源和工具。就像 USB 让电脑连接各种设备，MCP 让 AI 连接各种外部系统。

```
┌─────────────┐         MCP          ┌─────────────┐
│   Claude    │◄────────────────────►│  MCP Server │
│   Code      │                       │  (你的服务)  │
└─────────────┘                       └─────────────┘
                                              │
                                              ▼
                                      ┌─────────────┐
                                      │   外部资源   │
                                      │ 数据库/文件  │
                                      │   API/服务   │
                                      └─────────────┘
```

---

## MCP 的核心概念

### 1. MCP Host（宿主）

运行 Claude Code 等 AI 应用的主程序

### 2. MCP Client（客户端）

在 Host 内运行，与 Server 保持 1:1 连接

### 3. MCP Server（服务器）

独立的进程，通过 MCP 协议提供工具和数据

### 4. Resources（资源）

MCP Server 暴露的可读取数据（如文件内容、数据库记录）

### 5. Tools（工具）

MCP Server 提供的可执行功能

---

## MCP 协议结构

### 通信方式

```
MCP 使用 JSON-RPC 2.0 进行通信

请求格式：
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search",
    "arguments": {"query": "..."}
  }
}

响应格式：
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": "..."
  }
}
```

### 核心方法

| 方法 | 说明 |
|------|------|
| `initialize` | 建立连接，交换能力 |
| `tools/list` | 列出可用工具 |
| `tools/call` | 调用工具 |
| `resources/list` | 列出可用资源 |
| `resources/read` | 读取资源 |

---

## 构建 MCP Server

### 项目结构

```
mcp-server-example/
├── server.py          # MCP Server 主程序
├── requirements.txt   # 依赖
└── README.md
```

### server.py

```python
"""
MCP Server 示例
提供文件搜索和内容读取功能
"""

from mcp.server import Server
from mcp.types import Tool, Resource
from pydantic import AnyUrl
import json

# 创建 Server 实例
server = Server("file-explorer")

# ============ 工具定义 ============

@server.list_tools()
async def list_tools() -> list[Tool]:
    """列出所有可用工具"""
    return [
        Tool(
            name="search_files",
            description="在指定目录中搜索文件",
            inputSchema={
                "type": "object",
                "properties": {
                    "directory": {
                        "type": "string",
                        "description": "要搜索的目录路径"
                    },
                    "pattern": {
                        "type": "string",
                        "description": "文件名匹配模式，如 *.py"
                    }
                },
                "required": ["directory"]
            }
        ),
        Tool(
            name="read_file",
            description="读取文件内容",
            inputSchema={
                "type": "object",
                "properties": {
                    "path": {
                        "type": "string",
                        "description": "文件的完整路径"
                    }
                },
                "required": ["path"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> str:
    """执行工具调用"""
    if name == "search_files":
        return await search_files(**arguments)
    elif name == "read_file":
        return await read_file(**arguments)
    else:
        raise ValueError(f"未知工具: {name}")

# ============ 工具实现 ============

async def search_files(directory: str, pattern: str = "*") -> str:
    """搜索文件"""
    import glob
    import os

    # 安全检查：防止目录遍历攻击
    directory = os.path.abspath(directory)

    search_pattern = f"{directory}/**/{pattern}" if pattern != "*" else f"{directory}/**/*"

    files = []
    for path in glob.glob(search_pattern, recursive=True):
        if os.path.isfile(path):
            files.append(path)

    return json.dumps(files[:50], indent=2)  # 最多返回 50 个

async def read_file(path: str) -> str:
    """读取文件内容"""
    import os

    # 安全检查
    path = os.path.abspath(path)

    if not os.path.exists(path):
        return f"文件不存在: {path}"

    try:
        with open(path, 'r', encoding='utf-8') as f:
            content = f.read()
        return content
    except Exception as e:
        return f"读取错误: {e}"

# ============ 资源定义 ============

@server.list_resources()
async def list_resources() -> list[Resource]:
    """列出可用资源"""
    return [
        Resource(
            uri=AnyUrl("file:///current-dir"),
            name="Current Directory",
            description="当前工作目录"
        )
    ]

@server.read_resource()
async def read_resource(uri: AnyUrl) -> str:
    """读取资源"""
    import os
    if str(uri) == "file:///current-dir":
        return os.getcwd()
    raise ValueError(f"未知资源: {uri}")

# ============ 启动服务器 ============

if __name__ == "__main__":
    import mcp.server.stdio

    async def main():
        async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
            await server.run(
                read_stream,
                write_stream,
                server.create_initialization_options()
            )

    import asyncio
    asyncio.run(main())
```

### requirements.txt

```
mcp>=1.0.0
pydantic>=2.0.0
```

---

## MCP Server 配置

### 在 Claude Code 中配置

1. 创建或编辑 `CLAUDE.md` 文件
2. 添加 MCP Server 配置：

```markdown
## MCP Servers

### file-explorer
```json
{
  "command": "python",
  "args": ["/path/to/your/mcp-server-example/server.py"]
}
```
```

### 启动方式

Claude Code 启动时会自动发现并连接配置的 MCP Server。

---

## 实战：用 MCP 构建知识库查询

### 场景

希望 Claude Code 能够查询你的私人笔记

### 实现步骤

```python
# notes_server.py

from mcp.server import Server
from mcp.types import Tool
import json
from pathlib import Path

server = Server("knowledge-base")

NOTES_DIR = Path.home() / "notes"

@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="search_notes",
            description="搜索笔记内容",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词"
                    }
                },
                "required": ["query"]
            }
        ),
        Tool(
            name="read_note",
            description="读取指定笔记",
            inputSchema={
                "type": "object",
                "properties": {
                    "filename": {
                        "type": "string",
                        "description": "笔记文件名"
                    }
                },
                "required": ["filename"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> str:
    if name == "search_notes":
        query = arguments["query"].lower()
        results = []

        for note_file in NOTES_DIR.glob("*.md"):
            content = note_file.read_text().lower()
            if query in content:
                results.append(note_file.name)

        return json.dumps(results, indent=2)

    elif name == "read_note":
        note_path = NOTES_DIR / arguments["filename"]
        if note_path.exists():
            return note_path.read_text()
        return "笔记不存在"

    raise ValueError(f"未知工具: {name}")
```

---

## MCP 安全考虑

### 信任来源

```
⚠️ 只运行来自可信来源的 MCP Server
⚠️ 避免运行未审查的第三方 Server
```

### 安全最佳实践

| 实践 | 说明 |
|------|------|
| 沙箱运行 | MCP Server 在隔离环境中运行 |
| 最小权限 | 只授予必要的文件/网络访问 |
| 输入验证 | 验证所有传入的参数 |
| 日志审计 | 记录工具调用历史 |

---

## 练一练

1. 修改上面的 `server.py`，添加一个新工具 `count_lines` 统计文件行数
2. 构建一个 MCP Server，提供天气查询功能（调用外部 API）
3. 为你的私人项目配置 MCP Server

---

## 小测验

**1. MCP 协议的全称是？**
- A. Model Compute Protocol
- B. Model Context Protocol
- C. Machine Communication Protocol

**2. MCP Server 的作用是？**
- A. 存储 Claude Code 的数据
- B. 通过标准协议提供工具和数据
- C. 运行 AI 模型

**3. MCP 使用什么协议进行通信？**
- A. REST API
- B. GraphQL
- C. JSON-RPC 2.0

---

## 答案

1-B | 2-B | 3-C

---

## 下一步

- [Hooks 机制：自定义 Agent 行为](04-Hooks机制：自定义Agent行为.md)
