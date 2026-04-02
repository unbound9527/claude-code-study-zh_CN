# Claude Code 中文教程

> 从零开始，系统学习 Claude Code，成为 AI 协作高手。

[![GitHub stars](https://img.shields.io/github/stars/unbound9527/claude-code-zh_CN?style=flat&color=gold)](https://github.com/unbound9527/claude-code-zh_CN/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/unbound9527/claude-code-zh_CN?style=flat)](https://github.com/unbound9527/claude-code-zh_CN/network/members)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Compatible-purple)](https://code.claude.com)

---

## 目录

- [这是什么](#这是什么)
- [学习路径](#学习路径)
- [快速导航](#快速导航)
- [项目特色](#项目特色)
- [项目结构](#项目结构)
- [参考资料](#参考资料)

---

## 这是什么

一套完整的学习路径，帮你从"第一次用 AI"到"熟练使用 Claude Code"。

**特别适合**：对 AI 工具感兴趣的任何人。无论你是刚接触命令行的技术新手，还是想深度定制 Agent 的资深架构师。

---

## 学习路径

```
L1 掌握用法 → L2 熟练运用 → L3 技术视角 → L4 实践经验
```

| 阶段 | 内容 | 水平评价 |
|:---:|------|:----:|
| **L1 掌握用法** | 安装、清晰表达、工具使用、Git、Skills、MCP | 基本成功 |
| **L2 熟练运用** | 任务分解、CLAUDE.md、上下文管理、权限、Hooks、子代理、Teams、Computer Use | 登堂入室 |
| **L3 技术视角** | 源码解读、从零构建Agent、MCP协议原理、Hooks机制 | 炉火纯青 |
| **L4 实践经验** | 安全、Git规范、团队规范、省钱、工作流、TDD | 融会贯通 |

---

## 快速导航

### L1 掌握用法 — 入门基础

| 文档 | 说明 |
|:-----|:-----|
| [安装与配置](L1-掌握用法/01-install.md) | 安装、代理、国产模型接入 |
| [清晰表达需求](L1-掌握用法/02-prompt.md) | Prompt 公式 + 示例技巧 |
| [工具使用](L1-掌握用法/03-tools.md) | 核心工具详解 |
| [Git 版本控制](L1-掌握用法/04-git.md) | 分支、提交、协作 |
| [Skills 技能](L1-掌握用法/05-skills.md) | 自定义技能系统 |
| [MCP 扩展](L1-掌握用法/06-mcp.md) | 模型上下文协议 |

### L2 熟练运用 — 进阶能力

| 文档 | 说明 |
|:-----|:-----|
| [任务分解](L2-熟练运用/01-break-tasks.md) | Plan Mode、拆解方法 |
| [CLAUDE.md](L2-熟练运用/02-claude-md.md) | 项目永久记忆 |
| [上下文管理](L2-熟练运用/03-manage-context.md) | /clear、/compact、/context |
| [权限管理](L2-熟练运用/04-permissions.md) | 安全边界、精细控制 |
| [Hooks 钩子](L2-熟练运用/05-hooks.md) | 事件驱动自动化 |
| [子代理](L2-熟练运用/06-subagent.md) | 多代理协作 |
| [Agent Teams](L2-熟练运用/07-agent-teams.md) | 多AI团队协作 |
| [Computer Use](L2-熟练运用/08-compute-use.md) | 电脑操控能力 |
| [Link WeChat](L2-熟练运用/09-link-wechat.md) | 微信链接能力 |

### L3 技术视角 — 原理与实现

| 文档 | 说明 |
|:-----|:-----|
| [深入理解Claude-Code的Agent架构](L3-技术视角/01-深入理解Claude-Code的Agent架构.md) | 核心架构 |
| [内置Tools系统](L3-技术视角/02-内置Tools系统-注册加载与调用决策.md) | 工具机制 |
| [Subagent机制](L3-技术视角/03-Subagent机制-创建方式与递归防护.md) | 子代理原理 |
| [Harness工程分类](L3-技术视角/04-Harness工程分类-目录全解析.md) | 目录结构 |
| [记忆系统](L3-技术视角/05-记忆系统-MEMORY与上下文压缩.md) | 上下文管理 |
| [MCP扩展机制](L3-技术视角/06-MCP扩展机制与Computer-Use分析.md) | 扩展协议 |
| [多模型支持](L3-技术视角/07-多模型支持-Provider抽象与能力覆盖.md) | Provider抽象 |
| [权限系统](L3-技术视角/08-权限系统-三元决策链路.md) | 安全决策 |
| [省Token设计](L3-技术视角/09-省Token设计-AutoCompact与选择性保留.md) | 成本优化 |
| [提示词预处理](L3-技术视角/10-提示词预处理-用户输入到LLM之间发生了什么.md) | 输入处理 |
| [KAIROS通俗解释](L3-技术视角/11-KAIROS通俗解释-持久助手与自动记忆整合.md) | 持久记忆 |
| [内部员工特权](L3-技术视角/12-内部员工特权-ANT标识与特权功能.md) | 特权机制 |
| [AI-Agent面试指南](L3-技术视角/13-AI-Agent面试指南-Claude-Code源码知识点.md) | 面试要点 |

### L4 实践经验 — 最佳实践

| 文档 | 说明 |
|:-----|:-----|
| [安全最佳实践](L4-实践经验/00-security.md) | 沙箱、权限、审计 |
| [Git 最佳实践](L4-实践经验/01-Git.md) | 分支、提交、协作规范 |
| [团队规范](L4-实践经验/02-team-norms.md) | 团队协作模式 |
| [省钱技巧](L4-实践经验/03-save-money.md) | Token 优化、成本控制 |
| [工作流](L4-实践经验/04-workflow.md) | 自动化工作流 |
| [TDD](L4-实践经验/05-TDD.md) | 测试驱动开发 |

### 附录 — 参考资料

| 文档 | 说明 |
|:-----|:-----|
| [Prompt 模板库](附录/prompt模板库.md) | 模板集合 |
| [Agent Teams 官方](附录/agent-teams-official.md) | 官方文档 |
| [MCP 官方](附录/mcp-official.md) | 官方文档 |
| [检查点与回滚](附录/11-checkpoints.md) | 会话快照 |
| [Plugins 插件](附录/12-plugins.md) | 功能打包 |
| [CLI 参考](附录/13-advanced-cli.md) | 命令行详解 |
| [高级功能](附录/14-advanced-features.md) | 高级功能 |

---

## 项目特色

| 特色 | 说明 |
|:-----|:-----|
| **由浅入深** | 从安装到工程化实现，循序渐进 |
| **中文原创** | 专为中文用户设计，告别机翻 |
| **免费开源** | MIT 协议，欢迎贡献 |

---

## 项目结构

```
claude-code-zh-cn/
├── L1-掌握用法/      # 入门基础
├── L2-熟练运用/      # 进阶能力
├── L3-技术视角/      # 原理实现
├── L4-实践经验/      # 最佳实践
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
