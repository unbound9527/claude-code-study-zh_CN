# 递归生成器之上：Claude Code 的 Agent 架构与最小 MVP

## 导言

当我们说一个工具是"Agent"时，这个词已经高度通胀——从简单的问答机器人到能够自我改进的 AGI 系统，都被冠以 Agent 之名。Claude Code 对此有一种务实且工程化的理解：**Agent 是带有工具调用能力的递归查询生成器**。这不是一个宏大的哲学定义，而是一个可执行的工程架构。

本文拆解 Claude Code 的核心架构：消息如何流入、Task 如何组织、query 生成器如何递归运转、以及整个系统如何通过 Budget 机制实现自我约束。

---

## 一、核心架构三分层

Claude Code 的运行时可以被划分为三层，每一层都有清晰的职责边界：

```
┌─────────────────────────────────────────────────────────┐
│                    QueryEngine (调度层)                   │
│  submitMessage() → processUserInput() → query()         │
│  负责：slash 命令解析、消息初始化、Budget 守卫           │
├─────────────────────────────────────────────────────────┤
│                      query() (循环层)                    │
│  AsyncGenerator — 递归生成器，驱动主对话循环              │
│  负责：API 调用、工具决策、AutoCompact、结果 yield       │
├─────────────────────────────────────────────────────────┤
│                   Task 系统 (任务层)                     │
│  TaskType = local_agent | remote_agent | ...           │
│  负责：长期运行的并发任务管理（tmux pane、后台任务等）    │
└─────────────────────────────────────────────────────────┘
```

### 1.1 调度层：QueryEngine

QueryEngine 是整个系统的入口点。它的类型签名如下：

```typescript
// 核心方法签名
async *submitMessage(
  prompt: string | ContentBlockParam[],
  options?: { uuid?: string; isMeta?: boolean },
): AsyncGenerator<SDKMessage, void, unknown>
```

**关键设计：AsyncGenerator 作为 API 契约**。整个 `submitMessage()` 是一个 `AsyncGenerator<SDKMessage>`，它向调用者 yield `SDKMessage` 类型的消息。SDKMessage 是一个 discriminated union，它是一个联合类型，表示一个消息可以有多种类型。

注：yield 关键字在 JavaScript 中用于生成器函数中，用于返回一个值，并暂停函数的执行，直到下一次调用 next() 方法。
```typescript
function* generator() {
  yield 1
  yield 2
  yield 3
}
const gen = generator()
console.log(gen.next()) // { value: 1, done: false }
console.log(gen.next()) // { value: 2, done: false }
console.log(gen.next()) // { value: 3, done: false }
console.log(gen.next()) // { value: undefined, done: true }
```

```typescript
type SDKMessage =
  | { type: 'assistant'; message: AssistantMessage; ... }
  | { type: 'user'; message: SDKUserMessageReplay; ... }
  | { type: 'system'; subtype: 'compact_boundary'; ... }
  | { type: 'result'; subtype: 'success' | 'error_max_turns' | ...; ... }
  | { type: 'stream_event'; event: StreamEvent; ... }
```

这个设计使得 Claude Code 可以作为**被嵌入的 SDK** 使用（`@anthropic-ai/claude-agent-sdk` 的核心），而不只是 CLI。CLI 和 SDK 共用同一套 query 逻辑，只是 yield 出来的消息被不同层消费。

**一次 submitMessage 调用的完整流程**：

```
submitMessage() 入口
    ├─ processUserInput()          ← 解析 slash 命令（/commit, /mcp, ...）
    │                                返回 shouldQuery / resultText / allowedTools
    ├─ fetchSystemPromptParts()       ← 构建 system prompt（工具描述、规则）
    ├─ 若 shouldQuery = false        ← slash 命令已本地处理，直接返回 result
    └─ query() generator             ← 启动主循环
         ├─ AutoCompact 检查          ← token 超阈值则压缩上下文
         ├─ callModel()               ← API 流式调用
         ├─ 收集 tool_use blocks     ← 模型决定调用的工具
         ├─ runTools() / StreamingToolExecutor  ← 执行工具
         └─ yield* query()            ← 递归，继续下一轮
```

### 1.2 循环层：query() 生成器

query() 的核心是一个 `while(true)` 循环，通过 `yield* queryLoop()` 启动。关键在于：它不是简单的循环，而是**递归调用自身**来继续下一轮对话。

```typescript
// 简化的 query() 签名
export async function* query(
  params: QueryParams,
): AsyncGenerator<StreamEvent | RequestStartEvent | Message | ...> {
  const consumedCommandUuids: string[] = []
  const terminal = yield* queryLoop(params, consumedCommandUuids)
  return terminal
}
```

每一轮循环结束后，在特定条件下会创建新的 State 并**通过 `while(true)` 循环继续**，而非真正递归调用函数自身。这是一个**受控的递归模式**：

```typescript
// 递归模式核心：state 替换而非函数递归
const toolUseContextWithQueryTracking = {
  ...updatedToolUseContext,
  queryTracking,
}

const nextTurnCount = turnCount + 1

// 检查 maxTurns 上限
if (maxTurns && nextTurnCount > maxTurns) {
  yield createAttachmentMessage({ type: 'max_turns_reached', ... })
  return { reason: 'max_turns', turnCount: nextTurnCount }
}

queryCheckpoint('query_recursive_call')
const next: State = {
  messages: [...messagesForQuery, ...assistantMessages, ...toolResults],
  toolUseContext: toolUseContextWithQueryTracking,
  turnCount: nextTurnCount,
  transition: { reason: 'next_turn' },
}
state = next  // 直接替换 state 对象，while(true) 继续下一次迭代
```

**为什么这样设计？**

传统的函数递归 `result = await query(state)` 会有调用栈不断加深的问题。采用 `while(true) + state 对象替换` 的模式：
- 每次"递归"只是修改变量，不是真正的函数调用
- 无调用栈溢出风险
- `queryCheckpoint()` 可以记录每次"递归"的 checkpoint 用于性能分析

**每轮循环中发生了什么**：

```
API 流式调用 (deps.callModel)
    │
    ├─ 收集 assistantMessages（完整响应）
    ├─ 提取 tool_use blocks（模型决定调用的工具）
    ├─ needsFollowUp = tool_use_blocks.length > 0
    │
    ├─ 工具执行：runTools() 或 StreamingToolExecutor
    │   ├─ 权限检查 canUseTool()
    │   ├─ 工具结果转换为 UserMessage (tool_result)
    │   └─ normalizeMessagesForAPI() 标准化后放入 toolResults
    │
    └─ needsFollowUp = true → 循环继续（递归点）
      needsFollowUp = false → return { reason: 'completed' }
```

---

## 二、Task 系统：超越单轮对话

单靠 query 生成器只能处理单次 submitMessage 的多轮对话。Claude Code 的 Task 系统负责管理**超出单个 query 生命周期的长期任务**。

### 2.1 Task 类型体系

```typescript
export type TaskType =
  | 'local_bash'      // 本地 shell 命令
  | 'local_agent'     // 本地子 agent（tmux pane）
  | 'remote_agent'    // 远程 agent
  | 'in_process_teammate'  // 进程内 agent
  | 'local_workflow'  // 本地工作流
  | 'monitor_mcp'     // MCP 监控
  | 'dream'           // 深度思考模式
```

Task ID 采用前缀 + 8 字节随机 hex 的设计，前缀用于快速识别类型：

```typescript
const TASK_ID_PREFIXES: Record<string, string> = {
  local_bash: 'b',
  local_agent: 'a',
  remote_agent: 'r',
  in_process_teammate: 't',
  local_workflow: 'w',
  monitor_mcp: 'm',
  dream: 'd',
}
// 36^8 ≈ 2.8 万亿组合，防止暴力 symlink 攻击
const TASK_ID_ALPHABET = '0123456789abcdefghijklmnopqrstuvwxyz'
```

### 2.2 Task 状态机

```
pending → running → completed
                   → failed
                   → killed
```

状态转移通过 `registerTask()` 和 `updateTaskState()` 管理。TaskHandle 包含 `taskId` 和可选的 `cleanup` 回调，用于在任务结束时释放 tmux pane 等资源。

**Task 和 query 的关系**：Task 是比 query 更大的容器单位。一个 Task（比如一个本地 agent tmux pane）可以包含多个 query 会话。当用户输入 `/model` 切换模型时，当前 Task 的 query 会被中断并重启。

---

## 三、消息追踪链：queryTracking

Claude Code 在整个递归调用链中追踪一条 `queryTracking` 对象：

```typescript
// 递归追踪对象
const queryTracking = toolUseContext.queryTracking
  ? {
      chainId: toolUseContext.queryTracking.chainId,  // 同一轮对话共享
      depth: toolUseContext.queryTracking.depth + 1,   // 递归深度递增
    }
  : {
      chainId: deps.uuid(),
      depth: 0,
    }
```

- `chainId`：标识同一个对话链（用于 analytics 关联同一轮递归的所有事件）
- `depth`：递归深度计数器

这个对象在每次工具调用和 analytics 事件中被记录，用于重建完整的 agent 调用树。

---

## 四、Budget 体系：系统的刹车

Claude Code 在多个层面设置了 Budget 预算限制，防止无限运行：

### 4.1 Turn Budget

```typescript
maxTurns?: number
```

在 query 循环中：

```typescript
if (maxTurns && nextTurnCount > maxTurns) {
  yield createAttachmentMessage({ type: 'max_turns_reached', maxTurns, ... })
  return { reason: 'max_turns', turnCount: nextTurnCount }
}
```

### 4.2 USD Budget

```typescript
maxBudgetUsd?: number
```

在 QueryEngine 中：

```typescript
if (maxBudgetUsd !== undefined && getTotalCost() >= maxBudgetUsd) {
  yield { type: 'result', subtype: 'error_max_budget_usd', ... }
  return
}
```

### 4.3 Token Budget（实验性，feature-gated）

```typescript
const budgetTracker = feature('TOKEN_BUDGET') ? createBudgetTracker() : null
```

以及 API 层面的 `taskBudget`：

```typescript
taskBudget?: { total: number }  // API 的 output_config.task_budget
```

---

## 五、最小 MVP 构成

综合以上分析，Claude Code 的最小 Agent MVP 可以这样概括：

```
最小单位：一次 query() 循环
├── 输入：messages[] + systemPrompt + userContext
├── 决策：callModel() → 模型决定是否调用工具
├── 执行：runTools() → 工具结果
├── 继续条件：tool_use_blocks.length > 0 → while(true) 继续
└── 停止条件：无 tool_use / maxTurns / maxBudget / prompt_too_long
```

**所有 Budget 机制都是对上述循环的外部约束**，而非修改循环本身的逻辑。这意味着即使 Budget 耗尽，系统也会优雅地停止，而不是崩溃。

---

## 六、设计哲学分析

### 6.1 为什么选择 AsyncGenerator 而非 Promise？

传统的 Agent 框架会这样设计：

```typescript
async function chat(prompt: string): Promise<Result> {
  // 所有逻辑封装在一个 async 函数中
}
```

Claude Code 选择了 `AsyncGenerator<SDKMessage>`：

```typescript
async *chat(prompt: string): AsyncGenerator<SDKMessage> {
  yield { type: 'assistant', ... }   // 流式输出
  yield { type: 'progress', ... }     // 进度通知
  yield { type: 'result', ... }       // 最终结果
}
```

**优势**：
1. **流式感知**：调用方（CLI 或 SDK）可以实时渲染 assistant 的逐 token 输出
2. **中间件能力**：可以在 yield 链中注入中间逻辑（如 analytics、UI 更新）
3. **双向通信**：subagent 的 mailbox 机制允许父 agent 向子 agent 发送信号（通过 AbortController）

### 6.2 为什么 query 是 while(true) 而不是真正的递归函数？

真正的函数递归：
```typescript
async function query(state: State): Promise<Terminal> {
  const result = await callModel(state)
  if (result.needsFollowUp) {
    return query({ ...state, messages: [...result.messages] })
  }
  return result
}
```

Claude Code 的方案：
```typescript
while (true) {
  const result = await callModel(state)
  if (!result.needsFollowUp) return result
  state = { ...state, messages: [...result.messages] }
}
```

**关键差异**：真正的递归在每次调用时是**新的函数调用栈**，无法从外部中断正在执行的深度递归。而 `while(true)` 循环在每次迭代之间是**平等的**，可以通过 `state.transition` 对象注入控制信息，也可以通过 `break` / `return` 立即退出到调用方。

### 6.3 Task 系统和 query 系统的分离

这是一个**关注点分离**的经典案例：

- `query()` 只关心：**一轮对话的输入 → LLM 响应 → 工具执行 → 继续/停止**
- `Task` 系统只关心：**长期运行任务的注册、状态、清理**

两者通过 `toolUseContext.agentId` 关联——当一个 Task（比如 subagent）启动时，它的 `query()` 调用会携带自己的 `agentId`，使 analytics 可以将对话事件归因到正确的 Task。

---

## 七、架构图

```
                    ┌──────────────────────┐
                    │   CLI / SDK / REPL   │
                    │  (调用方)             │
                    └──────────┬───────────┘
                               │ yield* AsyncGenerator<SDKMessage>
                               ▼
              ┌─────────────────────────────────┐
              │       QueryEngine.submitMessage() │
              │  ├─ processUserInput()            │
              │  ├─ fetchSystemPromptParts()       │
              │  └─ yield* query()                 │
              └────────────────┬──────────────────┘
                               │
                               ▼
              ┌─────────────────────────────────┐
              │   query() — while(true) 循环      │
              │                                 │
              │  每次迭代 = 一轮 API 调用         │
              │  ├─ AutoCompact 检查              │
              │  ├─ callModel() (流式)           │
              │  ├─ 收集 tool_use blocks         │
              │  ├─ runTools() / StreamingExec   │
              │  └─ needsFollowUp? → 继续/stop   │
              └────────────────┬──────────────────┘
                               │
              ┌────────────────┴──────────────────┐
              │           Task 系统                 │
              │  ├─ TaskRegistry (registerTask)   │
              │  ├─ TaskState: pending→running   │
              │  │   → completed|failed|killed    │
              │  └─ 生命周期通过 tmux/进程管理     │
              └─────────────────────────────────┘
```

---

## 总结

Claude Code 的 Agent 架构揭示了一种**工程化而非概念化**的 Agent 设计思路：

1. **AsyncGenerator 作为 API 契约**：使 CLI 和 SDK 共用同一套逻辑，同时支持流式输出和中间件注入
2. **while(true) 循环 + state 替换**：在保留"递归语义"的同时，避免了函数调用栈的深度问题
3. **Budget 体系是外部约束**：Turn/USD/Token Budget 不修改循环逻辑，而是在循环之上加装"断路器"
4. **Task 系统处理长期任务**：Task 和 query 的分离使系统可以管理超出单次 API 调用生命周期的并发工作
5. **queryTracking 实现全链路追踪**：通过 chainId + depth 在 analytics 层面还原完整的 agent 调用树

理解了这个架构，就能理解 Claude Code 所有上层功能（工具系统、Subagent、Compact、MCP）都建立在这个稳固的递归查询生成器之上。
