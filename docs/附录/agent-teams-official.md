# Anthropic Agent Teams 官方架构说明

> 本章节翻译自 Anthropic 官方文档，介绍 Claude Code 中 Agent Teams 的架构和使用方法。

---

## 官方架构概览

**Agent Teams = 多个 Agent 协同工作**

在 Claude Code 中，Agent Teams 允许你同时启动多个并行的 Agent，每个 Agent 负责不同的任务，最后整合结果。

```
用户（你）
    │
    ├── Agent A ──→ 负责任务 A
    ├── Agent B ──→ 负责任务 B
    └── Agent C ──→ 负责任务 C
                    │
                    ▼
              整合结果
                    │
                    ▼
               最终答案
```

---

## 核心概念

### 1. Orchestrator（协调者）

协调整个团队的运作：
- 分解任务
- 分配工作
- 整合结果

### 2. Specialized Agents（专业 Agent）

每个 Agent 有特定职责：
- 领域专家
- 特定任务
- 独立工作空间

### 3. Communication Protocol（通信协议）

Agent 之间如何通信：
- 共享文件
- 消息传递
- 状态同步

---

## 官方推荐架构

### 架构一：星形架构（推荐新手）

```
           用户
            │
      [Orchestrator]
       /    │    \
      A     B     C
    专家  专家   专家
```

所有 Agent 都通过协调者通信，简单易管理。

### 架构二：网状架构

```
    A ◀───▶ B
    │  用户  │
    ▼        ▼
    C ──────▶ D
```

Agent 之间可以直接通信，适合复杂协作。

---

## Anthropic 官方指南

### 最佳实践

1. **明确角色和职责**
   每个 Agent 应该有一个清晰的职责范围。

2. **保持 Agent 数量适中**
   建议 3-5 个 Agent，避免过于复杂。

3. **清晰的通信机制**
   使用文件或消息队列进行通信。

4. **错误处理**
   每个 Agent 应该能够独立处理错误。

### 适用场景

| 场景 | 推荐架构 |
|------|----------|
| 简单任务分解 | 星形 |
| 复杂多领域分析 | 网状 |
| 并行独立任务 | 星形 |
| 需要协商的创意任务 | 网状 |

---

## 与 Claude Code 的集成

Claude Code 支持通过内置 Agent 工具创建 Agent Teams：

```javascript
// 创建多个专业 Agent
const agentA = await Agent.create({
  name: 'frontend-dev',
  instructions: '你是一个前端专家，负责 UI 组件...',
  tools: ['Read', 'Write', 'Edit']
});

const agentB = await Agent.create({
  name: 'backend-dev',
  instructions: '你是一个后端专家，负责 API...',
  tools: ['Read', 'Write', 'Bash']
});

// 并行执行
const results = await Promise.all([
  agentA.run(),
  agentB.run()
]);
```

---

## 官方文档参考

更多详细信息请查看：
- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [Agent 架构说明](https://code.claude.com/docs/en/agent-teams)
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)

---

## 相关开源项目

### 多 Agent 框架

| 项目 | Stars | 说明 |
|------|-------|------|
| [MetaGPT](https://github.com/geekan/MetaGPT) | ⭐ 38k+ | 多 Agent 软件开发框架，模拟软件公司角色 |
| [CrewAI](https://github.com/crewAI/crewAI) | ⭐ 20k+ | 多 Agent 编排框架，支持角色定义 |
| [AutoGen](https://microsoft.github.io/autogen/) | ⭐ 15k+ | 微软多 Agent 对话框架 |
| [ChatDev](https://github.com/open-compass/ChatDev) | ⭐ 12k+ | 虚拟软件公司多 Agent 协作 |
| [AgentVerse](https://github.com/OpenGVLab/AgentVerse) | ⭐ 5k+ | 多 Agent 模拟框架 |

### Agent 开发工具

| 项目 | Stars | 说明 |
|------|-------|------|
| [LangChain Agents](https://github.com/langchain-ai/langchain) | ⭐ 65k+ | Agent 开发主流框架 |
| [LlamaIndex](https://github.com/run-llama/llama_index) | ⭐ 35k+ | 知识库 Agent 框架 |
| [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) | ⭐ 125k+ | 自主 Agent 先驱项目 |
| [GPT-Engineer](https://github.com/ignorept/gpt-engineer) | ⭐ 25k+ | AI 代码生成 Agent |

### Claude Code 生态

| 项目 | Stars | 说明 |
|------|-------|------|
| [claude-code](https://github.com/anthropics/claude-code) | ⭐ 8k+ | Claude Code CLI 源码 |
| [modelcontextprotocol](https://github.com/modelcontextprotocol) | ⭐ 3.2k+ | MCP 协议及服务器 |

---

## 跟做

1. 阅读官方文档
2. 尝试用 Claude Code 创建简单的 Agent Teams
3. 观察 Agent 之间的协作

---

## 下一步

- [Hooks 原理与官方指南](../L4-精通技巧/14-hooks-advanced.md)
