# Anthropic 官方 MCP 指南

> 本章节基于 Anthropic Engineering 博客关于 Code Execution with MCP 的官方说明。

---

## MCP 是什么？

**MCP = Model Context Protocol**

MCP 是一个开放协议，允许 AI 模型与外部工具和数据源连接。

```
┌──────────────┐     MCP      ┌──────────────┐
│ Claude Code  │◀───────────▶│ MCP Server   │
│   (客户端)   │             │  (服务)   │
└──────────────┘             └──────────────┘
                                   │
                                   ▼
                            ┌──────────────┐
                            │  外部资源    │
                            │ 数据库/API等  │
                            └──────────────┘
```

---

## MCP 官方架构

### 核心组件

| 组件 | 说明 |
|------|------|
| **MCP Host** | Claude Code 等 AI 应用 |
| **MCP Client** | 与 Server 通信的客户端 |
| **MCP Server** | 提供特定功能的服务器 |
| **Transport** | 通信方式（stdio、WebSocket） |

### 通信流程

```
1. Host 启动 MCP Client
       ↓
2. Client 连接到 Server
       ↓
3. Server 注册可用工具
       ↓
4. Host 调用工具
       ↓
5. Client 转发请求到 Server
       ↓
6. Server 执行并返回结果
       ↓
7. Client 返回结果给 Host
```

---

## Claude Code 中的 MCP

### 支持的工具类型

Claude Code 通过 MCP 支持：

| 类型 | 工具 | 用途 |
|------|------|------|
| **Filesystem** | 读取、写入、搜索文件 | 文件操作 |
| **Git** | 提交、分支、日志 | 版本控制 |
| **Database** | 查询、更新数据 | 数据操作 |
| **API** | HTTP 请求 | 外部服务 |

---

## 官方 MCP 服务器

### 1. Filesystem Server

```bash
# 安装
npm install -g @modelcontextprotocol/server-filesystem

# 配置
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"]
    }
  }
}
```

### 2. GitHub Server

```bash
# 安装
npm install -g @modelcontextprotocol/server-github

# 配置
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token"
      }
    }
  }
}
```

### 3. PostgreSQL Server

```bash
# 安装
npm install -g @modelcontextprotocol/server-postgres

# 配置
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    }
  }
}
```

---

## Code Execution with MCP

> 以下内容来自 Anthropic Engineering Blog

### 为什么需要 MCP？

传统的 AI 对话只能处理文字，而 MCP 让 AI 能够：
- 读取真实的文件系统
- 执行代码
- 访问真实数据库
- 与外部 API 交互

### MCP 的安全模型

```
┌─────────────────────────────────────┐
│           Claude Code                │
├─────────────────────────────────────┤
│  用户权限决定 MCP 能做什么           │
│                                     │
│  allowedDirectories:               │
│  - /project/src  ✓                  │
│  - /System      ✗                   │
└─────────────────────────────────────┘
```

---

## MCP 最佳实践

### 1. 限制访问范围

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem",
               "/home/user/projects"]  // 只允许访问项目目录
    }
  }
}
```

### 2. 使用环境变量

```json
{
  "mcpServers": {
    "api": {
      "command": "node",
      "args": ["./mcp-server.js"],
      "env": {
        "API_KEY": "from-env"
      }
    }
  }
}
```

### 3. 错误处理

```javascript
// 在 MCP Server 中
server.tool('fetchData', {
  handler: async ({ query }) => {
    try {
      const result = await externalAPI(query);
      return { content: result };
    } catch (error) {
      throw new Error(`API Error: ${error.message}`);
    }
  }
});
```

---

## 官方资源

- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)
- [MCP SDK 文档](https://github.com/modelcontextprotocol)
- [官方 MCP Servers](https://github.com/modelcontextprotocol/servers) ⭐ 3.2k+

## 相关开源项目

### MCP 生态

| 项目 | Stars | 说明 |
|------|-------|------|
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | ⭐ 3.2k+ | 官方 MCP 服务器集合 |
| [MCP-Servers (Cline)](https://github.com/cline/mcp-servers) | ⭐ 1k+ | Cline 社区维护的服务器 |
| [mcp-go](https://github.com/ModelContextProtocol/mcp-go) | ⭐ 200+ | Go 语言 MCP 框架 |
| [mcp-langchain](https://github.com/mcp-langchain/mcp-langchain) | ⭐ 500+ | LangChain MCP 集成 |

### 多 Agent 框架

| 项目 | Stars | 说明 |
|------|-------|------|
| [MetaGPT](https://github.com/geekan/MetaGPT) | ⭐ 38k+ | 多 Agent 软件开发框架 |
| [CrewAI](https://github.com/crewAI/crewAI) | ⭐ 20k+ | 多 Agent 编排框架 |
| [AutoGen](https://microsoft.github.io/autogen/) | ⭐ 15k+ | 微软多 Agent 对话框架 |
| [ChatDev](https://github.com/open-compass/ChatDev) | ⭐ 12k+ | 虚拟软件公司多 Agent |

### Claude Code 生态

| 项目 | Stars | 说明 |
|------|-------|------|
| [Claude Code](https://docs.anthropic.com/claude-code) | - | 官方文档 |
| [claude-code (GitHub)](https://github.com/anthropics/claude-code) | ⭐ 8k+ | Claude Code CLI 源码 |

---

## 跟做

1. 安装一个官方 MCP Server
2. 配置到 Claude Code
3. 尝试使用它

---

## 下一步

- [安全最佳实践](09-security.md)
