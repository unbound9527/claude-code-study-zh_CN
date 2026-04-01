# Claude Code 中文教程项目

## 项目概述

这是一个 Claude Code 中文学习教程项目，包含 46 个 Markdown 文档，分为 6 个学习阶段和附录。

## 目录结构

```
docs/
├── L1-认识工具/       # 安装教程、AI 基础、插件生态、Git、CLI
├── L2-掌握用法/       # 清晰表达、工具使用、语音输入、Prompt 技巧
├── L3-熟练运用/       # 任务分解、上下文管理、迭代优化、工作流
├── L4-精通技巧/       # 记忆、子代理、Agent Teams、MCP、Hooks
├── L5-工程化学习/     # 源码解读、MCP 协议、Hooks 机制
├── L6-实践经验/       # 高效模式、避坑指南、用户案例
├── L7-环境配置/       # 代理设置、国产模型接入
└── 附录/              # 模板库、FAQ、官方文档、开源项目
```

## 文档标准结构

每章文档包含以下标准章节：
1. 本章目标
2. 概念讲解
3. 跟着做
4. 练一练
5. 小测验
6. 答案
7. 下一步
8. 本章小结（大章节最后一个文件有，帮助回顾）

## 项目规范

### 链接规范
- 同一章节内链接：使用相对路径，如 `02-first-chat.md`
- 跨章节链接：使用相对路径，如 `../L2-掌握用法/01-clear-request.md`
- 外部链接：使用完整 URL

### 标题层级
- 章节标题：H1（#）
- 主要章节：H2（##）
- 子章节：H3（###）
- 禁止跳过层级（如 H1 后直接 H4）

### 格式规范
- 分割线：使用 `---`
- 列表：使用 `-` 或 `1.` 保持一致
- 代码块：使用 ``` 并标注语言

## 可用命令

### 质量检查
- `/quality [文件路径或目录]` - 运行完整质量检查（链接、标题、格式）
- `/check-links` - 仅检查所有链接有效性

### 内容管理
- `/sync-readme` - 同步更新 README 目录索引
- `/check-crossref` - 验证交叉引用

### 信息获取
- `/fetch-info <URL>` - 从外部 URL 提取有用信息补充到项目
- `/check-updates` - 检查 Claude Code 最新动态

### 构建发布
- `/build` - 构建静态网站
- `/preview` - 本地预览

## 自动化工作流

### 定时任务
- 每周同步 README 目录
- 每 3 天检查 Claude Code 更新

### Hooks
- 文档修改前自动检查
- Git 提交后自动通知

## 技术栈

- Claude Code CLI
- MCP (Model Context Protocol) - filesystem, github
- Hooks - preToolUse, postCommit
- Agent Teams - quality-check-team

## 相关开源项目

### 多 Agent 框架

| 项目 | Stars | 说明 |
|------|-------|------|
| [MetaGPT](https://github.com/geekan/MetaGPT) | ⭐ 38k+ | 多 Agent 软件开发框架 |
| [CrewAI](https://github.com/crewAI/crewAI) | ⭐ 20k+ | 多 Agent 编排框架 |
| [AutoGen](https://microsoft.github.io/autogen/) | ⭐ 15k+ | 微软多 Agent 对话框架 |

### MCP 生态

| 项目 | Stars | 说明 |
|------|-------|------|
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | ⭐ 3.2k+ | 官方 MCP 服务器集合 |
| [mcp-go](https://github.com/ModelContextProtocol/mcp-go) | ⭐ 200+ | Go 语言 MCP 框架 |

### Agent 开发工具

| 项目 | Stars | 说明 |
|------|-------|------|
| [LangChain](https://github.com/langchain-ai/langchain) | ⭐ 65k+ | Agent 开发主流框架 |
| [LlamaIndex](https://github.com/run-llama/llama_index) | ⭐ 35k+ | 知识库 Agent 框架 |
| [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) | ⭐ 125k+ | 自主 Agent 先驱项目 |
