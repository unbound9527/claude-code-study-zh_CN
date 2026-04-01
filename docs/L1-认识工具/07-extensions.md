# Claude Code 插件生态

## 本章目标

- 了解 Claude Code 的生态扩展方式
- 掌握 VS Code 集成和 JetBrains 集成
- 理解 MCP（Model Context Protocol）生态

---

## Claude Code 的生态架构

Claude Code 不仅仅是一个独立应用，它是一个**可扩展的 AI 平台**：

```
┌─────────────────────────────────────────────────────┐
│                   Claude Code                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ VS Code  │  │ JetBrains│  │   CLI    │        │
│  │  插件     │  │   插件    │  │  终端    │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│       │              │              │              │
│       └──────────────┼──────────────┘              │
│                      ▼                              │
│              ┌───────────────┐                      │
│              │  MCP Client   │                      │
│              └───────┬───────┘                      │
│                      │                              │
│                      ▼                              │
│         ┌────────────────────────┐                   │
│         │   MCP Servers 生态     │                   │
│         │  (文件/数据库/API/...) │                   │
│         └────────────────────────┘                   │
└─────────────────────────────────────────────────────┘
```

---

## VS Code 集成

### Claude for VS Code 插件

在 VS Code 中直接使用 Claude，无需离开编辑器。

#### 安装步骤

1. 打开 VS Code
2. 按 `Ctrl + P`（Windows/Linux）或 `Cmd + P`（Mac）打开快速打开
3. 输入：
   ```
   ext install anthropics.claude-code
   ```
4. 按 Enter 安装
5. 安装完成后，点击左侧边栏的 Claude 图标
6. 登录你的账号，开始使用

#### 功能特性

| 功能 | 说明 |
|------|------|
| **内联对话** | 选中代码，右键"询问 Claude" |
| **代码审查** | 让 AI 审查当前文件 |
| **代码补全** | AI 驱动的代码补全建议 |
| **代码生成** | 描述需求，AI 生成代码 |
| **Bug 修复** | 选中错误，AI 帮你分析修复 |

#### 使用技巧

```
快捷键：
- Ctrl + Shift + A：打开 Claude 对话
- Ctrl + Shift + L：选中代码后，询问 Claude
- Ctrl + Shift + I：生成代码
```

### 与独立版的区别

| 特性 | VS Code 插件版 | 独立应用版 |
|------|--------------|-----------|
| 文件访问 | ✅ 直接访问 | ✅ 需要打开文件夹 |
| Git 集成 | ✅ 更深入 | ✅ 支持 |
| 界面 | 在编辑器内 | 独立窗口 |
| 多文件处理 | ✅ 方便 | ✅ 支持 |

---

## JetBrains 集成

### Claude for JetBrains 插件

支持 IntelliJ IDEA、WebStorm、PyCharm、PhpStorm 等 JetBrains 全家桶。

#### 安装步骤

1. 打开 JetBrains IDE
2. 进入 **Settings/Preferences** → **Plugins**
3. 搜索 **"Claude"**
4. 找到 **"Claude"** 插件并安装
5. 重启 IDE
6. 在 **Settings** 中登录 Claude 账号

#### 主要功能

| 功能 | 说明 |
|------|------|
| **Chat 面板** | IDE 侧边栏直接对话 |
| **代码审查** | 选中代码，右键发送审查 |
| **自动补全** | AI 代码补全建议 |
| **提交检查** | Git 提交时自动审查变更 |

---

## Claude Desktop 应用

### 独立桌面客户端

除了浏览器网页版，Claude 还有独立的桌面应用：

| 平台 | 下载地址 | 安装包格式 |
|------|---------|----------|
| macOS | claude.ai/code | .dmg |
| Windows | claude.ai/code | .exe |
| Linux | claude.ai/code | .deb / .AppImage |

#### 桌面版特有功能

- **系统级快捷键**：全局呼出 Claude
- **通知集成**：任务完成后发送系统通知
- **深色模式**：更好的视觉体验
- **离线可用**：已加载的对话可离线查看

---

## MCP 生态（Model Context Protocol）

### 什么是 MCP？

MCP 是一种开放协议，让 Claude 可以连接各种外部数据源和服务。

```
Claude Code ←→ MCP Client ←→ MCP Servers
                                   ↓
                    ┌──────────────┴──────────────┐
                    ↓              ↓              ↓
               文件系统        数据库           API
```

### MCP 能做什么？

| MCP 服务器 | 功能 | 使用场景 |
|-----------|------|---------|
| **文件系统** | 读写本地文件 | 项目文件操作 |
| **Git** | 执行 Git 操作 | 版本控制 |
| **数据库** | 查询/操作数据库 | 数据分析 |
| **Slack/Discord** | 发送消息 | 团队通知 |
| **GitHub** | PR/Issue 操作 | 代码管理 |
| **浏览器** | 控制浏览器 | Web 自动化 |

### 常用 MCP 服务器

#### 官方 MCP 服务器

| 服务器 | npm 包 | 说明 |
|-------|--------|------|
| Filesystem | `@modelcontextprotocol/server-filesystem` | 本地文件读写 |
| Git | `@modelcontextprotocol/server-git` | Git 操作 |
| Brave Search | `@modelcontextprotocol/server-brave-search` | 网页搜索 |

#### 安装示例

```bash
# 安装文件系统 MCP 服务器
npm install -g @modelcontextprotocol/server-filesystem

# 安装 Git MCP 服务器
npm install -g @modelcontextprotocol/server-git
```

### 如何配置 MCP

在 Claude Code 设置中添加 MCP 服务器：

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/directory"]
    },
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git"]
    }
  }
}
```

---

## Skills（技能系统）

Claude Code 支持自定义 Skills，扩展 AI 的能力。

### 什么是 Skills？

Skill = 预定义的 Prompt 模板 + 配套工具，让你一键完成复杂任务。

### 内置 Skills

Claude Code 自带一些常用 Skills：

| Skill | 用途 |
|-------|------|
| `/review` | 代码审查 |
| `/test` | 生成测试 |
| `/explain` | 解释代码 |
| `/refactor` | 重构代码 |

### 自定义 Skills

你可以在 `.claude/skills/` 目录下创建自己的 Skills：

```
项目/
├── .claude/
│   ├── skills/
│   │   ├── my-skill.md      # 自定义技能
│   │   └── readme.md        # Skills 说明
│   └── hooks/               # Hooks 配置
```

---

## 其他扩展方式

### Browser Extensions（浏览器扩展）

| 扩展 | 功能 |
|------|------|
| **Claude for Chrome** | Chrome 侧边栏直接对话 |
| **Claude Clipper** | 网页内容一键发送给 Claude |

### API 集成

开发者可以通过 API 将 Claude 集成到自己的应用：

```javascript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();
const message = await client.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 1024,
  messages: [{
    role: 'user',
    content: '帮我写一个 Hello World'
  }]
});
```

### Slack/Discord 集成

通过 Bot 在团队沟通工具中使用 Claude：

| 平台 | 集成方式 |
|------|---------|
| Slack | Slack App + Claude Bot |
| Discord | Discord Bot + Claude API |
| Teams | Teams App + Claude API |

---

## 生态选择指南

| 使用场景 | 推荐方案 |
|---------|---------|
| 日常编码辅助 | VS Code 插件 或 JetBrains 插件 |
| 深度开发工作 | 独立桌面应用 + MCP |
| 快速问答 | 网页版 |
| 团队协作 | Team 版本 + Slack 集成 |
| 自动化脚本 | CLI 版本 + MCP |

---

## 跟做

1. 在 VS Code 或你的 IDE 中安装 Claude 插件
2. 安装至少一个 MCP 服务器
3. 尝试用 `/review` 命令审查一段代码

---

## 本章小结

| 扩展方式 | 核心价值 |
|---------|---------|
| VS Code / JetBrains 插件 | 在编辑器内直接获得 AI 辅助 |
| 桌面应用 | 完整功能 + 系统级集成 |
| MCP 生态 | 连接外部世界（数据库/API/文件系统）|
| Skills | 自定义 AI 工作流程 |

**生态图示**：
```
Claude Code 是核心
   ├── IDE 插件（VS Code / JetBrains）→ 编码辅助
   ├── 桌面应用 → 完整功能
   ├── MCP Servers → 能力扩展
   └── Skills → 自定义工作流
```

**重点记住**：Claude Code 不是一个孤立的工具，而是一个**可扩展的 AI 平台**。通过插件和 MCP，你可以让它连接任何你需要的服务和数据源。

---

## 下一步

- [Git 版本控制](04-git.md)
- [L4-精通技巧：MCP 协议](../L4-精通技巧/07-mcp.md)
