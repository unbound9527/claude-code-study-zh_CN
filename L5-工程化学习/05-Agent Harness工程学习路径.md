# Agent 工程学习路径

本教程与 [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) 的 Agent 工程学习路径对应，帮你建立完整的知识体系。

---

## 核心理念

> **"Agent 是模型，代码是 Harness"**
>
> Agent 是神经网络，通过训练学会感知环境、推理目标、采取行动。代码是 Harness，为模型提供工具、知识、观察和行动接口。

**Bash is all you need. Real agents are all the universe needs.**

---

## 学习路径映射

### 第一阶段：循环与工具

| 本教程 | learn-claude-code | 内容 |
|--------|-------------------|------|
| L2 掌握用法 | s01 Agent Loop | while 循环 + stop_reason 判断 |
| L2 掌握用法 | s02 Tool Use | dispatch map 工具注册 |

**核心概念**：
```
用户 → messages[] → LLM → response
                         ↓
              stop_reason == "tool_use"?
             /                            \
          yes                             no
           ↓                               ↓
      执行工具                         返回文本
      追加结果
           ↓
      返回 messages[]
```

### 第二阶段：规划与知识

| 本教程 | learn-claude-code | 内容 |
|--------|-------------------|------|
| L3 熟练运用 | s03 TodoWrite | TodoManager + 主动提醒 |
| L4 精通技巧 | s04 Subagent | 独立 messages[] 隔离 |
| L4 精通技巧 | s05 build-skills | tool_result 按需注入 |
| L3 熟练运用 | s06 Context Compact | 三层压缩策略 |

**s03 核心理念**：没有计划的 agent 走哪算哪。先列步骤再动手，完成率翻倍。

**s04 核心理念**：大任务拆小，每个小任务干净的上下文。防止噪声泄露。

**s05 核心理念**：用到什么知识，临时加载什么知识。Agent 应该知道有什么可用知识，然后按需自己拉取。

### 第三阶段：持久化与团队

| 本教程 | learn-claude-code | 内容 |
|--------|-------------------|------|
| L4 精通技巧 | s07 Task System | 文件持久化任务图 |
| L4 精通技巧 | s08 Background Tasks | 守护线程 + 通知 |
| L4 精通技巧 | s03 agent-teams | 队友 + 异步邮箱 |
| L4 精通技巧 | — | s10 Team Protocols |
| L4 精通技巧 | — | s11 Autonomous Agents |
| L4 精通技巧 | — | s12 Worktree Isolation |

**s07 核心理念**：大目标要拆成小任务，排好序，记在磁盘上。目标持久化到单次对话之外。

**s08 核心理念**：慢操作丢后台，agent 继续想下一步。Agent 不必阻塞等待。

**s09 核心理念**：任务太大一个人干不完，要能分给队友。持久化队友 + 异步邮箱。

---

## 进阶学习资源

### learn-claude-code 课程结构

```
s01-s02: 循环与工具（基础）
s03-s06: 规划与知识（进阶）
s07-s12: 持久化与团队（高级）
```

### 本教程对应内容

| 课程 | 本教程章节 |
|------|-----------|
| s01 Agent Loop | L2 掌握用法 / 工具使用 |
| s02 Tool Use | L2 掌握用法 / 工具使用 |
| s03 TodoWrite | L3 熟练运用 / 任务分解 |
| s04 Subagent | L4 精通技巧 / 子代理 |
| s05 Skills | L4 精通技巧 / 构建技能 |
| s06 Context Compact | L3 熟练运用 / 上下文管理 |
| s07 Task System | L4 精通技巧 / 自动化工作流 |
| s08 Background Tasks | L4 精通技巧 / 自动化工作流 |
| s09 Agent Teams | L4 精通技巧 / Agent Teams |
| s10 Team Protocols | L4 精通技巧 / Agent Teams |
| s11 Autonomous Agents | L4 精通技巧 / Agent Teams |
| s12 Worktree Isolation | L4 精通技巧 / 自动化工作流 |

---

## 与其他教程的关系

| 教程 | 特点 | 适用场景 |
|------|------|----------|
| **本教程** | 使用为导向，Claude Code 操作 | 想用好 Claude Code |
| **learn-claude-code** | 原理为导向，Agent 工程 | 想造自己的 Agent |
| **claude-howto** | 实用为导向，大量模板 | 想快速上手模板 |

**建议路线**：
1. 先学本教程，掌握 Claude Code 使用
2. 再学 learn-claude-code，理解 Agent 工程原理
3. 两者结合，从"会用"到"会造"

---

## 下一步

- [源码解读：Agent 架构](01-源码解读：Agent架构.md)
- [从零构建：最小 Agent 系统](02-从零构建：最小Agent系统.md)
- [MCP 协议](03-MCP协议：扩展Agent能力.md)
- [Hooks 机制](04-Hooks机制：自定义Agent行为.md)
