# Claude Code 插件生态

## 生态架构

```
Claude Code
├── VS Code 插件 / JetBrains 插件
├── 桌面应用
└── MCP Servers → 扩展能力
```

## IDE 集成

### VS Code
- 安装：`ext install anthropics.claude-code`
- 快捷键：`Ctrl+Shift+A` 打开对话

### JetBrains
- Settings → Plugins → 搜索 Claude
- 支持 IntelliJ/PyCharm/WebStorm 等

## MCP 生态

MCP = Model Context Protocol，连接外部数据源：

| 服务器 | 功能 |
|--------|------|
| filesystem | 读写本地文件 |
| git | Git 操作 |
| brave-search | 网页搜索 |
| github | PR/Issue 操作 |

**安装**：
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**配置**：
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

## Skills

自定义 Prompt 模板，存放在 `.claude/skills/`。

## 选择指南

| 场景 | 推荐 |
|------|------|
| 日常编码 | VS Code/JetBrains 插件 |
| 深度开发 | 桌面应用 + MCP |
| 快速问答 | 网页版 |
| 自动化 | CLI + MCP |

---

**下一步**：[Git 版本控制](04-git.md)
