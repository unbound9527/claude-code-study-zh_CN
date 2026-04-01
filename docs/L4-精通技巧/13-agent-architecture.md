# Agent 架构与 ReAct/Tool-Use 循环

## 什么是 Agent

能自主决策和执行任务的 AI 系统。

```
Agent = 大脑(LLM) + 手脚(Tools) + 记忆(Memory)
```

## ReAct 循环

**ReAct = Reasoning + Acting（思考 + 行动）**

```
1. Thought：LLM 分析，决定下一步
2. Action：执行工具（Read/Write/Bash...）
3. Observation：获取结果
4. 循环直到完成
```

## Tool-Use 循环

```
用户问题 → LLM 判断是否需要工具
  ↓需要 → 选择工具 → 执行 → 获取结果 → 回到 LLM
  ↓不需要 → 直接回答
```

## Claude Code 架构

```
┌─────────────────────────────────┐
│         Claude Code             │
│  ┌─────────┐    ┌─────────┐    │
│  │ LLM     │◄──►│ Memory  │    │
│  │(Claude) │    │(上下文) │    │
│  └────┬────┘    └─────────┘    │
│       │                          │
│       ▼                          │
│  ┌─────────────────────────┐    │
│  │   Tool Executor         │    │
│  │ Read│Write│Bash│Agent │    │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

---

**下一步**：[官方 Agent Teams 架构](../附录/agent-teams-official.md)
