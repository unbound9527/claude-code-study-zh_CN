# MCP：连接外部世界

## MCP 是什么

**MCP = Model Context Protocol**，给 Claude Code 装插件，连接外部工具和数据源。

```
无 MCP：Claude Code → 只能对话

有 MCP：Claude Code → 文件系统 / GitHub / 数据库 / 网页 / API
```

## 能做什么

| MCP | 功能 |
|-----|------|
| filesystem | 读取/搜索文件 |
| github | 查看仓库、创建 Issue、提交代码 |
| 数据库 | 查询数据、生成报表 |
| 网页 | 获取内容、抓取数据 |

## 安装

```bash
claude mcp install filesystem
claude mcp install github
claude mcp list
```

或配置 `~/.claude/settings.json`：
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    }
  }
}
```

## 使用

```
"用文件系统 MCP 读取 /Users/me/desktop/notes.txt"
"用 GitHub MCP 查看我的仓库"
```

## 安全注意

| ✅ 安全 | ❌ 风险 |
|--------|--------|
| 只用可信 MCP | 安装来源不明的 |
| 限制文件访问范围 | 给完全权限 |

---

**下一步**：[Hooks：自动化钩子](08-hooks.md)
