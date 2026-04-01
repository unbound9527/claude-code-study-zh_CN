# MCP：连接外部世界

MCP（Model Context Protocol）让 Claude Code 连接外部工具和数据源，像给 AI 安装插件一样。

---

## 核心概念

```
无 MCP：Claude Code → 只能对话

有 MCP：Claude Code → 文件系统 / GitHub / 数据库 / 网页 / API
```

MCP 采用客户端-服务器架构：
- **MCP 客户端**：Claude Code 内置
- **MCP 服务器**：你需要安装和配置

---

## 常用 MCP 服务器

### 1. Filesystem MCP

读写和搜索本地文件系统。

**安装**：
```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/directory
```

**配置**：
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**功能**：
- 读取文件内容
- 搜索文件
- 写入/编辑文件
- 列出目录

### 2. GitHub MCP

集成 GitHub API。

**安装**：
```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

**环境变量**：
```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
```

**功能**：
- 查看仓库和目录
- 创建 Issue 和 PR
- 提交代码
- 审查代码

### 3. Brave Search MCP

网页搜索。

**安装**：
```bash
claude mcp add brave-search -- npx -y @modelcontextprotocol/server-brave-search
```

**环境变量**：
```bash
export BRAVE_API_KEY=your_api_key
```

**功能**：
- 搜索网页
- 获取页面内容

### 4. Slack MCP

Slack 消息通知。

**安装**：
```bash
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
```

**环境变量**：
```bash
export SLACK_BOT_TOKEN=xoxb-xxxx
export SLACK_TEAM_ID=Txxxx
```

### 5. PostgreSQL MCP

数据库查询。

**安装**：
```bash
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
```

**环境变量**：
```bash
export DATABASE_URL=postgresql://user:password@localhost:5432/dbname
```

---

## 配置位置

| 位置 | 作用域 | 用途 |
|------|--------|------|
| `~/.claude/settings.json` | 用户级 | 全局 MCP |
| `.claude/settings.json` | 项目级 | 项目专用 MCP |

---

## 完整配置示例

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

---

## MCP 管理命令

```bash
# 添加 MCP
claude mcp add <name> -- <command>

# 列出已配置的 MCP
claude mcp list

# 删除 MCP
claude mcp remove <name>

# 重启 MCP
claude mcp restart <name>

# 检查 MCP 状态
claude mcp status
```

---

## 环境变量使用

在配置中引用环境变量：

```bash
export GITHUB_TOKEN=ghp_xxxx
export DATABASE_URL=postgresql://...
```

配置中引用：
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

## 多 MCP 配置

### 项目级 MCP

在项目 `.claude/settings.json` 中配置项目专用的 MCP：

```json
{
  "mcpServers": {
    "project-db": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${PROJECT_DB_URL}"
      }
    }
  }
}
```

### 本地覆盖

创建 `.claude/settings.local.json` 覆盖默认配置：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/custom/path"]
    }
  }
}
```

---

## 实战示例

### 示例 1：代码审查工作流

```
你：用 GitHub MCP 查看仓库中的 PR #123

Claude：[调用 GitHub MCP 获取 PR 信息]

你：审查这个 PR 的代码质量

Claude：[调用 GitHub MCP 获取代码差异，进行审查]
```

### 示例 2：数据库查询

```
你：用 PostgreSQL MCP 查询过去一周的新用户数

Claude：[调用 PostgreSQL MCP 执行查询]

结果：
| 日期 | 新用户数 |
|------|----------|
| 2024-01-01 | 145 |
| 2024-01-02 | 203 |
...
```

### 示例 3：综合搜索

```
你：用 Brave Search MCP 搜索 Claude Code 最新更新

Claude：[执行搜索，返回结果]
```

---

## 安全最佳实践

| ✅ 推荐 | ❌ 避免 |
|--------|---------|
| 只安装可信来源的 MCP | 安装来源不明的 MCP |
| 限制文件访问路径 | 允许访问整个文件系统 |
| 使用环境变量存储密钥 | 硬编码密钥在配置中 |
| 定期检查已安装的 MCP | 安装后不再过问 |
| 最小权限原则 | 给不必要的完全权限 |

---

## 安全配置示例

### 限制文件访问

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem",
               "/home/user/projects",  // 只允许访问项目目录
               "/home/user/documents"]
    }
  }
}
```

### 只读 MCP

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_READONLY_TOKEN}"  // 只读 token
      }
    }
  }
}
```

---

## 常见问题

**Q：MCP 安装后不生效？**

A：
1. 检查 `claude mcp list` 是否显示
2. 重启 Claude Code
3. 检查配置 JSON 格式是否正确

**Q：MCP 提示权限不足？**

A：
1. 检查环境变量是否设置
2. 确认 Token/API Key 有足够权限
3. 检查 MCP 服务器文档

**Q：可以同时用多个 MCP 吗？**

A：可以，Claude 会自动选择合适的 MCP 处理请求。

**Q：自定义 MCP 怎么开发？**

A：参考官方 MCP SDK：
- TypeScript: `@modelcontextprotocol/sdk`
- Python: `mcp`

---

## 下一步

- 深入 Hooks：[Hooks 自动化](08-hooks.md)
- 配置权限：[权限管理](11-permissions.md)
- 查看完整配置：[Advanced Features](../附录/14-advanced-features.md)
