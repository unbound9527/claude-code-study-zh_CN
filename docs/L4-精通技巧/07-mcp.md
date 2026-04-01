# MCP：连接外部世界

## 本章目标

- 理解什么是 MCP（Model Context Protocol）
- 知道 MCP 能连接什么
- 能使用 MCP 扩展 Claude Code 的能力

---

## 什么是 MCP？

**MCP = Model Context Protocol（模型上下文协议）**

MCP 就像给 Claude Code 装上**插件**，让它能连接外部工具和数据源。

```
没有 MCP 时：
Claude Code → 只能处理对话

有 MCP 时：
Claude Code → 连接 → 外部工具
              → 连接 → 数据库
              → 连接 → API
              → 连接 → 文件系统
```

---

## MCP 能做什么？

| MCP 类型 | 能做的事 |
|----------|----------|
| **文件系统** | 读取/搜索你电脑上的文件 |
| **GitHub** | 查看仓库、创建 Issue、提交代码 |
| **数据库** | 查询数据、生成报表 |
| **网页** | 获取网页内容、抓取数据 |
| **API** | 连接各种在线服务 |

---

## 常用 MCP 场景

### 1. 文件系统 MCP

**场景**：让 AI 直接读取你电脑上的文件

```
"请看一下我桌面上的 report.docx"
"帮我搜索 Documents 文件夹里的所有 PDF"
```

### 2. GitHub MCP

**场景**：管理 GitHub 仓库

```
"查看我的仓库列表"
"帮我创建一个新 Issue"
"提交这段代码到 GitHub"
```

### 3. 数据库 MCP

**场景**：查询和分析数据

```
"查询上个月的销售数据"
"帮我生成一份数据报表"
```

### 4. 网页 MCP

**场景**：获取网页信息

```
"帮我抓取这个电商网站的商品价格"
"获取新闻网站的头条"
```

---

## 安装 MCP

### 方法一：通过命令行安装

```bash
# 安装官方 MCP
claude mcp install filesystem
claude mcp install github

# 查看已安装的 MCP
claude mcp list
```

### 方法二：通过配置文件

在 `~/.claude/settings.json` 中配置：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/你的/目录路径"]
    }
  }
}
```

---

## 使用 MCP

### 和 AI 对话时直接使用

```
"请用文件系统 MCP 帮我读取 /Users/me/desktop/notes.txt"
```

### 主动指定 MCP

```
"请用 GitHub MCP 帮我查看我的仓库"
```

### 让 AI 自动选择

AI 会根据你的问题自动选择合适的 MCP（如果有配置的话）。

---

## 创建自己的 MCP

如果你会编程，可以创建自己的 MCP：

### 基本结构

```javascript
// my-mcp-server.js
const { MCPServer } = require('@modelcontextprotocol/sdk/server');
const { StdioServerTransport } = require('@modelcontextprotocol/sdk/server/stdio');

const server = new MCPServer({
  name: 'my-mcp-server',
  version: '1.0.0',
});

// 定义一个工具
server.tool('getWeather', {
  description: '获取城市天气',
  schema: {
    city: { type: 'string', required: true }
  },
  handler: async ({ city }) => {
    // 你的逻辑
    return { weather: '晴天', temperature: '25°C' };
  }
});

const transport = new StdioServerTransport();
server.run(transport);
```

---

## MCP 安全注意

| ✅ 安全做法 | ❌ 风险行为 |
|-------------|-------------|
| 只连接可信的 MCP | 安装来源不明的 MCP |
| 限制文件访问范围 | 给 MCP 完全的文件权限 |
| 定期检查已安装的 MCP | 不知道 MCP 在干什么 |

---

## 跟做

1. 查看你已安装的 MCP：
   ```
   claude mcp list
   ```

2. 如果有文件系统 MCP，尝试让 AI 读取一个文件

---

## 练一练

1. 了解有哪些官方 MCP 可用
2. 安装一个你感兴趣的 MCP
3. 尝试使用它

---

## 小测验

**1. MCP 是什么？**
- A. 一种聊天格式
- B. 让 AI 连接外部工具的协议
- C. 一种编程语言

**2. MCP 可以连接什么？**
- A. 只有网页
- B. 只有文件
- C. 各种外部工具和数据源

**3. 使用 MCP 时应该注意？**
- A. 随意安装
- B. 注意安全，只用可信的 MCP
- C. 不需要注意事项

---

## 答案

1-B | 2-C | 3-B

---

## 下一步

- [Hooks：自动化钩子](08-hooks.md)
