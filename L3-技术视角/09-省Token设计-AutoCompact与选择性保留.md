# 省 Token 设计：AutoCompact 与上下文压缩的选择性保留策略

## 导言

大模型应用的核心矛盾之一是：**上下文窗口是有限的，但对话历史是无限的**。Claude Code 的解决方案不是简单的"满了就截断"，而是一套精心设计的压缩策略体系。本文解析 Claude Code 如何在省 Token 和保信息之间取得平衡。


## 一、Token 消耗的来源

在 Claude Code 中，每次 API 调用的 Token 消耗来自多个部分：

```typescript
// API 请求的 Token 构成
totalTokens = systemPromptTokens     // 系统提示（工具描述、规则）
                    + userContextTokens     // 用户上下文（CLAUDE.md）
                    + messagesTokens        // 对话历史
                    + toolsResultTokens     // 工具执行结果
```

其中对话历史 `messagesTokens` 是增长最快的部分，因为每次工具调用都会产生新的 `tool_result` 消息。


## 二、AutoCompact 触发机制

### 2.1 阈值计算

Claude Code 不是等到上下文快满了才压缩，而是**提前触发**（autoCompact.ts）：

```typescript
// autoCompact.ts
export const AUTOCOMPACT_BUFFER_TOKENS = 13_000

export function getAutoCompactThreshold(model string) number {
  const effectiveContextWindow = getEffectiveContextWindowSize(model)
  // effectiveContextWindow = contextWindow - 20,000 (摘要输出预留)
  return effectiveContextWindow - AUTOCOMPACT_BUFFER_TOKENS
}
```

对于 200K context 的 Sonnet 4.6：
- `effectiveContextWindow = 200,000 - 20,000 = 180,000`
- `autoCompactThreshold = 180,000 - 13,000 = 167,000`

**在 167K tokens 时就触发压缩**，留出 13K tokens 的操作空间。

### 2.2 为什么要提前触发？

原因在于压缩本身也需要 token：
- 压缩摘要需要额外的 API 调用
- 摘要结果需要写入上下文
- 如果等到接近 200K 才压缩，压缩操作本身可能超出上下文限制


## 三、压缩保留策略：不是丢弃，而是替换

### 3.1 Claude Code 的压缩哲学

Claude Code 的压缩**不是简单的丢弃历史**，而是**选择性保留 + 摘要替换**：

```
压缩前（167K tokens）：
  [早期消息 1-50] + [中期消息 51-100] + [近期消息 101-150] + [当前消息 151]
       ↓
压缩后（目标 100K tokens）：
  [SystemCompactBoundary] + [摘要消息] + [保留的近期文件] + [Skills] + [当前消息 151]
```

### 3.2 保留的内容（buildPostCompactMessages）

```typescript
// compact.ts
export const POST_COMPACT_MAX_TOKENS_PER_FILE = 5_000
export const POST_COMPACT_SKILLS_TOKEN_BUDGET = 25_000
```

压缩后保留：

| 内容 | 保留策略 |
|------|---------|
| **最近 5 个文件** | 每个最多 5,000 tokens |
| **Skills** | 总预算 25,000 tokens |
| **Plan attachments** | 保持关联 |
| **工具/MCP/Agent 公告** | 保留（deferred announcements）|
| **SystemCompactBoundary** | 压缩分界线消息 |
| **摘要 UserMessage** | 替代被压缩的历史 |

### 3.3 为什么保留最近文件？

代码搜索是 Claude Code 的核心场景。如果压缩时把所有历史文件引用都删掉，模型就无法知道某个函数上次被修改是什么时候、修改了什么。**保留最近 5 个文件**使得模型可以引用具体的代码行，而不是只能说"根据之前的对话"。


## 四、两种压缩方向

### 4.1 prefix-preserving vs suffix-preserving

Claude Code 支持两种压缩方向（compact.ts约 line 330）：

```typescript
export function buildPostCompactMessages(result CompactionResult) Message[] {
  // 根据压缩方向决定保留哪一段
}
```

| 方向 | 含义 | 使用场景 |
|------|------|---------|
| `up_to` | 从指定消息向上压缩 | 保留 suffix（最近的对话）|
| `from` | 从指定消息向下压缩 | 保留 prefix（开头的上下文）|

### 4.2 缓存影响

不同的压缩方向对 prompt cache 有不同影响：
- **prefix-preserving**：保留开头，可以复用 cache（因为开头不变）
- **suffix-preserving**：保留结尾，适用于"结论在开头"的场景


## 五、SnipCompact：消息条目截断

### 5.1 HISTORY_SNIP 实验

除了 AutoCompact，还有一种更轻量的压缩——**SnipCompact**：

```typescript
// compact.ts
if (feature('HISTORY_SNIP')) {
  const snipResult = snipModule!.snipCompactIfNeeded(messagesForQuery)
  messagesForQuery = snipResult.messages
  snipTokensFreed = snipResult.tokensFreed
}
```

Snip 不是压缩整个对话，而是**截断过长的消息条目**。

### 5.2 与 AutoCompact 的关系

两者可以同时启用：
- **Snip**：移除过长的消息内容，保留消息结构
- **AutoCompact**：把早期消息替换为摘要


## 六、Microcompact：工具结果的微压缩

### 6.1 工具结果的大小问题

一个 `grep` 工具的结果可能有几万 tokens。如果不加控制，几次工具调用就能填满上下文。

### 6.2 Microcompact 的处理

Claude Code 有**工具结果的预算机制**：

```typescript
// toolResultStorage.ts
export function applyToolResultBudget(
  messages Message[],
  contentReplacementState ContentReplacementState | undefined,
  persistReplacements ((records ContentReplacementRecord[]) => void) | undefined,
  nonBudgetedTools Set<string>,
) Message[]
```

当工具结果超过阈值时：
1. 结果被保存到磁盘
2. API 收到的是文件路径引用
3. 需要时再从磁盘读取


## 七、ContextCollapse：替代方案

### 7.1 当前状态：完整 stub

`src/services/contextCollapse/index.ts` 是一个**实验性功能的 stub**：

```typescript
// contextCollapse/index.ts
export function isContextCollapseEnabled() boolean {
  return false  // 默认关闭
}

export async function applyCollapsesIfNeeded<T>(messages T) Promise<{ messages T; changed boolean }> {
  return { messages, changed false }  // 不做任何事
}
```

### 7.2 ContextCollapse 的设计思路

与 AutoCompact 不同，ContextCollapse 的思路是：
- **AutoCompact**：把历史变成摘要（有损）
- **ContextCollapse**：保留所有历史，但通过某种方式"折叠"不需要的部分

当 `feature('CONTEXT_COLLAPSE')` 启用时，它会**阻止 AutoCompact**（作为两种互斥策略）。


## 八、预算机制

### 8.1 Turn Budget

限制单次 submitMessage 的最大轮次：

```typescript
// QueryEngine.ts
maxTurns? number

// query.ts
if (maxTurns && nextTurnCount > maxTurns) {
  yield createAttachmentMessage({ type 'max_turns_reached', ... })
  return { reason 'max_turns' }
}
```

### 8.2 USD Budget

限制单次会话的最大花费：

```typescript
// QueryEngine.ts
maxBudgetUsd? number

// QueryEngine.ts
if (maxBudgetUsd !== undefined && getTotalCost() >= maxBudgetUsd) {
  yield { type 'result', subtype 'error_max_budget_usd', ... }
}
```

### 8.3 Token Budget（实验性）

```typescript
// query.ts
const budgetTracker = feature('TOKEN_BUDGET') ? createBudgetTracker()  null
```

以及 API 层面的 `taskBudget`：

```typescript
// query.ts
taskBudget? { total number }
```


## 九、架构图

```
                    用户输入
                        │
                        ▼
           ┌──────────────────────────┐
           │    tokenCount 估算         │
           │  (autoCompact 检查前)      │
           └────────────┬─────────────┘
                        │
           ┌────────────▼─────────────┐
           │   是否达到 autoCompact     │
           │   threshold (167K)?      │
           └────────────┬─────────────┘
                是      │      否
                ▼       │       │
    ┌──────────────────┤       │
    │  compactConversation()    │
    │  ┌──────────────────────┐ │
    │  │ 1. fork agent 生成摘要│ │
    │  │ 2. 保留：最近5文件    │ │
    │  │ 3. 保留：Skills 25K  │ │
    │  │ 4. 替换：早期→摘要    │ │
    │  │ 5. 熔断：3次失败停止   │ │
    │  └──────────────────────┘ │
    └───────┬──────────────────┘
            │
            ▼
    ┌──────────────────┐
    │  buildPostCompact │ → 重新构建消息列表
    │  Messages()       │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │   API 调用        │
    │  (更少的 tokens)  │
    └──────────────────┘
```


## 十、设计哲学

### 10.1 为什么不一开始就少发消息？

Claude Code 需要保持对话的完整性。如果一开始就只发最近 N 条消息，模型就无法做跨对话的全局推理。

### 10.2 为什么预算是"软"的？

Claude Code 没有硬性的"超过 N tokens 就拒绝"的限制，而是**提前触发压缩**。这是因为：
- 压缩是有成本的（额外的 API 调用）
- 但不压缩会更快地耗尽上下文
- **提前压缩**是一个平衡的选择

### 10.3 摘要 vs 折叠

Claude Code 选择**摘要**而非**折叠**，这是一个有意的权衡：
- 摘要会丢失细粒度信息
- 但摘要本身是 LLM 友好的（本来就是文本）
- 折叠保留了所有信息，但需要模型自己决定看哪部分


## 总结

Claude Code 的省 Token 设计是**多层次、多策略**的体系：

1. **AutoCompact**：在 167K tokens 时提前触发，留足操作空间
2. **选择性保留**：保留最近文件、Skills、计划状态，替换早期消息为摘要
3. **SnipCompact**：截断过长消息，而非整段删除
4. **Microcompact**：工具结果的按需持久化
5. **ContextCollapse**：未来的替代方案，用折叠代替摘要
6. **Budget 体系**：Turn/USD/Token 三个维度的外部约束

理解这个设计，就能理解为什么 Claude Code 可以在数千轮对话中稳定运行，而不是被无限增长的历史压垮。


