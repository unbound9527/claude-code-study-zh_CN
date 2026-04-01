# Claude Code 中文教程

> 从零开始，系统学习 Claude Code，成为 AI 协作高手。

[![GitHub stars](https://img.shields.io/github/stars/unbound9527/claude-code-zh_CN?style=flat&color=gold)](https://github.com/unbound9527/claude-code-zh_CN/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/unbound9527/claude-code-zh_CN?style=flat)](https://github.com/unbound9527/claude-code-zh_CN/network/members)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Compatible-purple)](https://code.claude.com)

---

## 目录

- [这是什么？](#这是什么)
- [学习路径](#学习路径)
- [快速导航](#快速导航)
- [项目特色](#项目特色)
- [项目结构](#项目结构)
- [参考资料](#参考资料)

---

## 这是什么？

一套完整的学习路径，帮你从"第一次用 AI"到"熟练使用 Claude Code"。

**特别适合**：对 AI 工具感兴趣的任何人。无论你是刚接触命令行的技术新手，还是想深度定制 Agent 的资深架构师。

---

## 学习路径

```
L0 环境配置 → L1 认识工具 → L2 掌握用法 → L3 熟练运用 → L4 精通技巧 → L5 工程化学习 → L6 实践经验
```

| 阶段 | 内容 | 水平评价 |
|:---:|------|:---:|
| **L0 环境配置** | 安装教程、代理设置、国产模型接入 | 初窥门径 |
| **L1 掌握用法** | 有效沟通、清晰表达、工具使用 | 渐入佳境 |
| **L2 掌握用法** | Git 版本控制 | 登堂入室 |
| **L3 熟练运用** | 任务分解、上下文管理、迭代优化、Claude.md | 基本成功 |
| **L4 精通技巧** | 记忆、子代理、Agent Teams、MCP、Hooks、安全、权限 | 融会贯通 |
| **L5 工程化学习** | 源码解读、从零构建 Agent、MCP 协议、Hooks 机制 | 炉火纯青 |
| **L6 实践经验** | 高效使用模式、常见陷阱与避坑、进阶用户案例 | 盖了帽了 |

---

## 快速导航

### L0 环境配置 — 安装与配置必备

| 文档 | 说明 |
|:-----|:-----|
| [安装教程](L0-环境配置/00-install.md) | 从零搭建环境 |
| [代理设置](L0-环境配置/01-proxy-settings.md) | 网络配置 |
| [国产模型接入](L0-环境配置/02-domestic-models.md) | 接入国内模型 |

### L1 掌握用法 — 核心技能

| 文档 | 说明 |
|:-----|:-----|
| [与 AI 有效沟通](L1-掌握用法/01-与AI有效沟通.md) | AI 基础 + 提问 + 理解回答 |
| [清晰表达需求](L1-掌握用法/02-清晰表达需求.md) | Prompt 公式 + 示例技巧 |
| [工具使用](L1-掌握用法/03-工具使用.md) | 核心工具详解 |

### L2 掌握用法 — Git 版本控制

| 文档 | 说明 |
|:-----|:-----|
| [Git 版本控制](L2-掌握用法/04-git.md) | 版本管理 |

### L3 熟练运用 — 进阶使用能力

| 文档 | 说明 |
|:-----|:-----|
| [分解复杂任务](L3-熟练运用/01-break-tasks.md) | 任务拆解 |
| [管理对话上下文](L3-熟练运用/02-manage-context.md) | 上下文管理 |
| [迭代优化答案](L3-熟练运用/03-iterate.md) | 迭代纠错 |
| [Claude.md 项目配置](L3-熟练运用/06-claude-md.md) | 项目级记忆 |

### L4 精通技巧 — 高级功能与定制

| 文档 | 说明 |
|:-----|:-----|
| [记忆功能](L4-精通技巧/01-memory.md) | 持久记忆 |
| [子代理](L4-精通技巧/02-subagent.md) | 多代理协作 |
| [Agent Teams](L4-精通技巧/03-agent-teams.md) | 团队协作 |
| [自动化工作流](L4-精通技巧/04-automate.md) | 自动化 |
| [构建自己的技能](L4-精通技巧/05-build-skills.md) | 自定义 Skill |
| [MCP：连接外部世界](L4-精通技巧/07-mcp.md) | MCP 扩展 |
| [Hooks：自动化钩子](L4-精通技巧/08-hooks.md) | Hooks 用法 |
| [安全最佳实践](L4-精通技巧/09-security.md) | 安全防护 |
| [模型选择与切换](L4-精通技巧/10-model-switch.md) | 模型配置 |
| [权限管理](L4-精通技巧/11-permissions.md) | 权限控制 |
| [Agent 架构与 ReAct 循环](L4-精通技巧/13-agent-architecture.md) | 原理深入 |

### L5 工程化学习 — 原理与实现

| 文档 | 说明 |
|:-----|:-----|
| [源码解读：Agent 架构](L5-工程化学习/01-源码解读：Agent架构.md) | 架构分析 |
| [从零构建：最小 Agent 系统](L5-工程化学习/02-从零构建：最小Agent系统.md) | 动手实现 |
| [MCP 协议：扩展 Agent 能力](L5-工程化学习/03-MCP协议：扩展Agent能力.md) | 协议原理 |
| [Hooks 机制：自定义 Agent 行为](L5-工程化学习/04-Hooks机制：自定义Agent行为.md) | Hooks 原理 |
| [Agent 工程学习路径](L5-工程化学习/05-Agent工程学习路径.md) | 学习路径对齐 |

### L6 实践经验 — 最佳实践汇总

### 附录 — 参考资料

| 文档 | 说明 |
|:-----|:-----|
| [Prompt 模板库](附录/prompt模板库.md) | 模板集合 |
| [常见问题](附录/常见问题.md) | FAQ |
| [官方 Agent Teams 架构](附录/agent-teams-official.md) | 官方文档 |
| [官方 MCP 指南](附录/mcp-official.md) | 官方文档 |
| [检查点与回滚](附录/11-checkpoints.md) | 会话快照 |
| [Plugins 插件系统](附录/12-plugins.md) | 功能打包 |
| [CLI 参考](附录/13-advanced-cli.md) | 命令行详解 |
| [Advanced Features](附录/14-advanced-features.md) | 高级功能 |

---

## 学习路径对齐

本教程与 [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) 的 Agent 工程学习路径对应：

| 本教程 | learn-claude-code | 核心理念 |
|--------|-------------------|----------|
| L2 掌握用法 | s01-s02 | Agent Loop + Tool Use |
| L3 熟练运用 | s03-s04 | 规划(TodoWrite) + 子代理隔离 |
| L4 精通技巧 | s05-s06 | Skills 按需加载 + 上下文压缩 |
| L5 工程化学习 | s07-s12 | 任务持久化 + 后台任务 + Teams + 自组织 |

---

## 项目特色

| 特色 | 说明 |
|:-----|:-----|
| **由浅入深** | 从安装到生态到工程化实现，循序渐进 |
| **中文原创** | 专为中文用户设计，告别机翻 |
| **免费开源** | MIT 协议，欢迎贡献 |

---

## 项目结构

```
claude-code-zh-cn/
├── L0-环境配置/      # 安装配置
├── L1-认识工具/      # 入门
├── L2-掌握用法/      # 基础
├── L3-熟练运用/      # 进阶
├── L4-精通技巧/      # 高级
├── L5-工程化学习/    # 工程化
├── L6-实践经验/      # 实践
├── 附录/             # 模板库、FAQ、官方文档
├── .claude/          # 自定义 Skill
├── CLAUDE.md         # 项目规范
└── README.md
```

---

## 参考资料

**开源项目**

| 项目 | 说明 |
|------|------|
| [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) | Agent 工程原理教学，12 课程递进式学习（s01-s12） |
| [claude-howto](https://github.com/luongnguyen/claude-howto) | Claude Code 实用指南，14k+ Stars |

**官方文档**

- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [Agent Teams 文档](https://code.claude.com/docs/en/agent-teams)
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)

---

## License

MIT License
