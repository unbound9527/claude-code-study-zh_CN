# Subagent 机制：资源隔离单元与递归调用图谱

## 导言

在一个复杂的软件工程任务中，Claude Code 经常需要"分而治之"——将一个大任务拆解给多个子 Agent 并行处理。这些子 Agent 如何被创建、如何与父 Agent 通信、又如何避免"Agent 调用 Agent 调用 Agent"的无限递归？本文解析 Claude Code 的 Subagent 机制，揭示其设计选择与潜在风险。

---

## 一、Subagent 的本质：隔离单元而非进程副本

Claude Code 的 Subagent/Teammate**不是进程的简单复制**，而是一个具有隔离状态的**计算单元**。

关键设计选择：
- 隔离的不只是"运行环境"，还包括**文件状态缓存**、**权限上下文**、**工具发现记录**
- 但隔离是有选择性的——通过 `SubagentContextOverrides` 可以显式共享 `setAppState`、`abortController` 等关键状态

---

## 二、两种 Subagent 创建方式

### 2.1 进程外 Subagent（tmux/iTerm2）

当用户运行 `/agent` 并选择创建 pane 时，Claude Code 在 tmux 或 iTerm2 中启动一个**新的 Claude Code 进程**：

```typescript
// 子进程创建：关键参数通过 mailbox 传递
const binaryPath = getTeammateCommand()
const teammateArgs = [
  `--agent-id ${quote([teammateId])}`,
  // ... 其他 CLI 参数
]
// spawn() 创建新的 Claude Code 进程
```

**父子通信通过 Mailbox 机制**：

```
父 Agent (主 Claude Code 进程)
    │
    │  writeToMailbox(sanitizedName, { from: TEAM_LEAD_NAME, prompt, ... })
    │
    ▼
子 Agent (tmux pane 中的 Claude Code 进程)
    │  读取自己的 inbox
    │  poller 定期检查新消息
    ▼
    处理指令，返回结果到 mailbox
```

Mailbox 本质上是文件系统上的一个临时目录（`~/.claude/teammates/<teamName>/<teammateName>/inbox/`），通过 JSON 文件传递消息。这避免了父子进程间的 IPC 复杂性，但也意味着**Mailbox 目录必须有读写权限**。

### 2.2 进程内 Subagent（In-Process Teammate）

当不需要 UI 隔离时，Claude Code 可以用 `spawnInProcessTeammate()` 在同一进程内启动子 Agent：

```typescript
export async function spawnInProcessTeammate(
  name: string,
  prompt: string,
  parentContext: ToolUseContext,
  config: InProcessSpawnConfig,
): Promise<void> {
  await startInProcessTeammate({ name, prompt, parentContext, ...config })
}
```

进程内 Subagent 通过直接调用 `query()` 并传入隔离的 context 来运行。优势是**零进程创建开销**，适合短期、轻量级的子任务。

---

## 三、`createSubagentContext()`：隔离了什么？

`createSubagentContext()` 函数是 Subagent 隔离的核心：

```typescript
export function createSubagentContext(
  parentContext: ToolUseContext,
  overrides?: SubagentContextOverrides,
): ToolUseContext {
  // 1. AbortController：共享 or 独立？
  const abortController = overrides?.shareAbortController
    ? parentContext.abortController           // 共享：父中断子也中断
    : createChildAbortController(parentContext.abortController)  // 独立

  // 2. 文件状态缓存：始终 clone（隔离的）
  readFileState: cloneFileStateCache(parentContext.readFileState),

  // 3. AppState 访问：
  //    - 共享则直接用 parentContext.getAppState
  //    - 独立则返回包装版本，设置 shouldAvoidPermissionPrompts = true
  getAppState: overrides?.shareAbortController
    ? parentContext.getAppState
    : () => {
        const state = parentContext.getAppState()
        return { ...state, toolPermissionContext: {
          ...state.toolPermissionContext,
          shouldAvoidPermissionPrompts: true,  // 子 Agent 不弹权限框
        }}
      },

  // 4. setAppState：独立 Subagent 的为 no-op（不接受外部状态变更）
  setAppState: overrides?.shareSetAppState
    ? parentContext.setAppState
    : () => {},

  // 5. ContentReplacementState：clone 而非 fresh
  //    原因：cache-sharing forks 需要看到 parent 的 tool_use_ids
  //          才能做出相同的 replacement decisions → cache hit
  contentReplacementState: cloneContentReplacementState(parentContext.contentReplacementState),

  // 6. 各种 Set：每个 Subagent 有独立的 skill 发现记录
  discoveredSkillNames: new Set<string>(),
  nestedMemoryAttachmentTriggers: new Set<string>(),
  loadedNestedMemoryPaths: new Set<string>(),
}
```

**隔离层级分析**：

| 资源 | 隔离方式 | 原因 |
|------|---------|------|
| `readFileState` | Clone | 防止子 Agent 的文件变更影响父 Agent |
| `AbortController` | 可共享 | 父终止时子应同步终止 |
| `toolPermissionContext` | 强制加 `shouldAvoidPermissionPrompts: true` | 子 Agent 不应自行弹出权限框 |
| `contentReplacementState` | Clone | 保持 prompt cache 命中 |
| `discoveredSkillNames` | 独立 | 每个 Agent 有独立的技能发现历史 |

---

## 四、递归深度追踪与潜在风险

### 4.1 `queryTracking` 的深度字段

每次 query 调用递归深入时，`queryTracking.depth` 递增：

```typescript
const queryTracking = toolUseContext.queryTracking
  ? {
      chainId: toolUseContext.queryTracking.chainId,  // 同一轮对话链的 ID
      depth: toolUseContext.queryTracking.depth + 1,   // 深度 +1
    }
  : {
      chainId: deps.uuid(),
      depth: 0,
    }
```

这个字段被记录在每一个 analytics 事件中：

```typescript
logEvent('tengu_auto_compact_succeeded', {
  queryChainId: queryChainIdForAnalytics,
  queryDepth: queryTracking.depth,
  ...
})
```

### 4.2 关键发现：没有硬限制

**在代码中，没有任何地方对 `queryTracking.depth` 进行上限检查。**

这意味着理论上 `depth` 可以无限增长：

```
用户: "帮我写一个 Web 服务器"
  └─ depth=0 → AgentTool 启动 subagent
      └─ depth=1 → subagent 说"我需要数据库"
          └─ depth=2 → subagent 启动另一个 subagent
              └─ depth=3 → ...
                  └─ depth=N → (无限)
```

### 4.3 实际限制来自其他 Budget

虽然 `depth` 本身没有硬限制，但实际的递归深度受限于：

```typescript
if (maxTurns && nextTurnCount > maxTurns) {
  yield createAttachmentMessage({ type: 'max_turns_reached', ... })
  return { reason: 'max_turns' }
}
```

- **`maxTurns`**：限制总轮次，而非递归深度。所以深度可能很大但每轮轮次有限
- **`maxBudgetUsd`**：费用上限

**这是一个设计选择，而非疏忽。** Claude Code 的设计哲学倾向于"信任开发者"，假设 Subagent 是被有意识地创建，而非被无限递归地自动生成。大多数实际场景中，3-4 层递归就能解决大多数复杂任务。

---

## 五、递归场景：正确使用与风险示例

### 5.1 正确的嵌套场景

```typescript
// 场景：大型重构任务
// 第一层：AgentTool(架构师) — 规划模块划分
//   └─ 第二层：AgentTool(开发者A) — 实现模块1
//       └─ 第三层：AgentTool(测试) — 验证模块1
//   └─ 第二层：AgentTool(开发者B) — 实现模块2
//       └─ 第三层：AgentTool(测试) — 验证模块2
```

每个子 Agent 有独立的 `discoveredSkillNames`，防止跨 Agent 的技能发现污染。

### 5.2 风险场景

```typescript
// 场景：AgentTool 调用自己（直接递归）
// 这是可能的，但：
// 1. 每次调用都是新的 spawn
// 2. 新的 subagent 有自己的 context（隔离）
// 3. 但 shareAbortController = true 时，父终止会终止所有子
// 4. depth 无限增长，analytics 记录越来越深的调用链
```

---

## 六、父子通信机制：Mailbox 的设计与局限

### 6.1 Mailbox 目录结构

```
~/.claude/teammates/
  <teamName>/
    <teammateName>/
      inbox/        ← 接收父 Agent 消息
      outbox/       ← 子 Agent 回复
      status/       ← 状态文件（running/completed/failed）
      teamFile      ← 团队配置
```

### 6.2 Mailbox 的同步特性

Mailbox 是**基于文件系统的同步机制**：
- 子 Agent 的 poller 定期轮询 `inbox/` 目录
- 没有 OS 级别的信号机制（如 signal）
- 超时需要手动处理

这意味着：
- **延迟低**：文件写入即同步
- **不适合高频通信**：轮询开销大
- **没有优先级机制**：所有消息平等

### 6.3 与 in-process Teammate 的区别

| 维度 | tmux Subagent | in-process Subagent |
|------|--------------|-------------------|
| 进程 | 独立进程 | 共用同一进程 |
| 通信 | Mailbox (文件) | 直接函数调用 |
| 资源隔离 | 完全隔离（独立内存） | 共享内存 |
| 启动开销 | 高（进程启动） | 极低 |
| 调试 | 困难（跨进程） | 容易 |
| 使用场景 | 长期任务、UI 展示 | 短期任务、批处理 |

---

## 七、`AgentTool` 的特殊地位

`AgentTool`（tools/AgentTool/）是整个 Subagent 系统的入口。它本身是一个 Tool，但它启动的 subagent 有独立的 `agentId`：

```typescript
// AgentTool 有独立的 TaskType: 'local_agent' / 'remote_agent'
// 每个 subagent 的 query 携带自己的 agentId
// analytics 通过 agentId 归因到正确的调用树
```

---

## 八、架构图

```
                        ┌──────────────────────────────────┐
                        │         AgentTool (入口)           │
                        │  call() → spawnTeammate / spawnInProcessTeammate │
                        └──────────────┬───────────────────┘
                                       │
               ┌────────────────────────┴────────────────────────┐
               ▼                                                 ▼
    ┌─────────────────────────┐                    ┌─────────────────────────────┐
    │   spawnTeammate()       │                    │   spawnInProcessTeammate()   │
    │   (tmux / iTerm2)       │                    │   (同一进程内)              │
    │                         │                    │                             │
    │  1. getTeammateCommand() │                    │  1. createSubagentContext() │
    │  2. spawn() 新进程       │                    │  2. startInProcessTeammate()│
    │  3. writeToMailbox()    │                    │  3. runAgent()               │
    │  4. 等待结果            │                    │  4. 直接返回                 │
    └───────────┬─────────────┘                    └──────────────┬──────────────┘
                │ Mailbox (文件系统)                             │ 直接调用
                ▼                                                ▼
    ┌───────────────────────────────────────────────────────────────────────┐
    │                         子 Agent Context                                │
    │  ┌─────────────────────────────────────────────────────────────────┐  │
    │  │  createSubagentContext(parentContext, overrides?)                 │  │
    │  │    - readFileState: clone (隔离)                                 │  │
    │  │    - abortController: 独立 or 共享                                │  │
    │  │    - setAppState: no-op or 共享                                   │  │
    │  │    - discoveredSkillNames: 独立 (防止污染)                        │  │
    │  │    - contentReplacementState: clone (保持 cache 命中)             │  │
    │  └─────────────────────────────────────────────────────────────────┘  │
    │                               │                                        │
    │                               ▼                                        │
    │  ┌─────────────────────────────────────────────────────────────────┐  │
    │  │  runAgent() → query() generator (独立递归)                      │  │
    │  │    queryTracking: { chainId: 继承, depth: parent.depth + 1 }    │  │
    │  └─────────────────────────────────────────────────────────────────┘  │
    └───────────────────────────────────────────────────────────────────────┘
```

---

## 九、递归防护的现状与思考

Claude Code 目前没有对 Subagent 递归深度做硬限制。这是一个**"信任开发者"的设计哲学**的体现：

- **相信用户**不会写出故意无限递归的提示词
- **相信模型**不会在没有充分理由时创建 subagent
- **analytics 记录**了 depth，但仅用于问题诊断，而非强制约束

潜在的改进方向：
1. **深度阈值告警**：当 depth 超过某值时在 analytics 中标记为异常
2. **深度上限配置**：`maxSubagentDepth` 环境变量
3. **父子关系图可视化**：让用户看到完整的 Subagent 调用树

---

## 总结

Claude Code 的 Subagent 机制是一个**精心设计的资源隔离方案**：

1. **两种隔离模式**：进程外（tmux）和进程内，各有适用场景
2. **Mailbox 通信**：简单但有效的父子消息传递，基于文件系统，同步但不实时
3. **`createSubagentContext()`**：逐字段决定隔离策略，兼顾安全与效率
4. **`queryTracking.depth`**：记录递归深度但不做限制，体现"信任开发者"哲学
5. **Budget 约束来自外部**：maxTurns 和 maxBudgetUsd 是防止无限运行的真正屏障

理解了这个机制，才能正确地使用 AgentTool——既享受分而治之的并行能力，又避免失控的递归深渊。
