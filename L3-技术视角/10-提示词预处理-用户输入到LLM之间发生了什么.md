# 提示词预处理：用户输入到 LLM 之间发生了什么

## 导言

当用户在 Claude Code 中输入一条指令时，从按下回车到 LLM 开始处理之间，经历了一个复杂的预处理管道。这个管道负责：slash 命令解析、权限检查、上下文预热、消息规范化、模型验证等工作。本文解析这个预处理管道的完整流程。


## 一、预处理管道总览

```
用户输入 "帮我优化这段代码"
        │
        ▼
┌──────────────────────────────────────────┐
│       processUserInput() (入口)            │
│  ┌────────────────────────────────────┐   │
│  │  1. 显示用户输入（setUserInputOnProcessing）│   │
│  │  2. 权限模式检查                      │   │
│  │  3. processUserInputBase()            │   │
│  └────────────────────────────────────┘   │
└────────────────────┬───────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│       processUserInputBase()              │
│  ┌────────────────────────────────────┐   │
│  │  1. Slash 命令解析                   │   │
│  │  2. Skills 发现                      │   │
│  │  3. 附件处理（IDE selection）         │   │
│  │  4. 消息构建                         │   │
│  │  5. 返回 shouldQuery / resultText   │   │
│  └────────────────────────────────────┘   │
└────────────────────┬───────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
  shouldQuery = false         shouldQuery = true
  (slash 命令已处理)          (需要 LLM 处理)
        │                         │
        ▼                         ▼
  返回 resultText            进入 query() 循环
```


## 二、processUserInput：入口函数

### 2.1 核心职责

```typescript
// processUserInput.ts
export async function processUserInput({
  input,
  mode,
  setToolJSX,
  context,
  messages,
  uuid,
  querySource,
  canUseTool,
  skipSlashCommands,
  ...
}) Promise<ProcessUserInputBaseResult> {
  // 1. 显示用户输入（REPL UI）
  if (mode === 'prompt' && inputString !== null && !isMeta) {
    setUserInputOnProcessing?.(inputString)
  }

  // 2. 获取当前权限模式
  const appState = context.getAppState()

  // 3. 调用核心处理
  const result = await processUserInputBase(
    input,
    mode,
    setToolJSX,
    context,
    messages,
    uuid,
    isAlreadyProcessing,
    querySource,
    canUseTool,
    appState.toolPermissionContext.mode,  // 传入权限模式
    skipSlashCommands,
    ...
  )

  // 4. 如果是 slash 命令（shouldQuery=false），执行 hooks
  if (!result.shouldQuery) {
    return result
  }

  // 5. 执行 UserPromptSubmit hooks
  // ...
}
```


## 三、processUserInputBase：核心处理

### 3.1 Slash 命令解析

当输入以 `/` 开头时，`processSlashCommand()` 负责解析：

```typescript
// processSlashCommand.tsx
export async function processSlashCommand({
  input,
  context,
  messages,
  ...
}) Promise<ProcessSlashCommandResult> {
  // 1. 提取 slash 命令名称
  // 2. 查找对应的命令处理器
  // 3. 执行命令（可能修改 messages）
  // 4. 返回 shouldQuery / resultText
}
```

### 3.2 消息构建

当需要 LLM 处理时，`createUserMessage()` 构建标准化的 UserMessage：

```typescript
// messages.ts
export function createUserMessage(params {
  content string | ContentBlockParam[]
  uuid string
  isMeta? boolean
  toolUseResult? string
  ...
}) UserMessage
```


## 四、sideQuery：验证与辅助查询

### 4.1 什么是 sideQuery

`src/utils/sideQuery.ts` 用于**不需要完整 query 循环的轻量 API 调用**：

```typescript
// sideQuery.ts
export async function sideQuery(params SideQueryOptions) Promise<Anthropic.Messages.Message> {
  // 用于：
  // 1. 模型验证（validateModel.ts）
  // 2. 工具描述生成
  // 3. 摘要生成（compact）
  // 4. 其他轻量调用
}
```

### 4.2 与主 query 的区别

| 特性 | sideQuery | query (主循环) |
|------|-----------|----------------|
| 用途 | 轻量验证/辅助 | 完整对话 |
| 工具 | 不带工具 | 带完整工具池 |
| 历史 | 可选 | 完整上下文 |
| 并发 | 可并行 | 串行递归 |


## 五、上下文构建

### 5.1 三个上下文组件

在 `QueryEngine.submitMessage()` 中，构建了三个上下文组件：

```typescript
// QueryEngine.ts
const {
  defaultSystemPrompt,   // 工具描述、规则
  userContext,          // CLAUDE.md 内容
  systemContext,        // git status、cache breaker
} = await fetchSystemPromptParts({
  tools,
  mainLoopModel,
  additionalWorkingDirectories ...,
  mcpClients,
  customSystemPrompt ...
})
```

### 5.2 SystemPrompt 组装

```typescript
// QueryEngine.ts
const systemPrompt = asSystemPrompt([
  ...(customPrompt !== undefined ? [customPrompt]  defaultSystemPrompt),
  ...(memoryMechanicsPrompt ? [memoryMechanicsPrompt]  []),
  ...(appendSystemPrompt ? [appendSystemPrompt]  []),
])
```

最终发送给 LLM 的 prompt 是多层组装的：
- `defaultSystemPrompt`：Claude Code 内置的系统提示
- `memoryMechanicsPrompt`：Memory 系统使用说明
- `appendSystemPrompt`：用户附加的额外提示


## 六、模型字符串规范化

### 6.1 `[1m]` / `[2m]` 后缀处理

Claude Code 支持在模型名后加 `[1m]` 或 `[2m]` 来指定 1M 或 200K context：

```typescript
// model.ts
export function normalizeModelStringForAPI(model string) string {
  return model.replace(/\[(1|2)m\]/gi, '')
}
```

发送给 API 时，这些后缀被**移除**（因为 API 本身通过参数控制 context 大小）。

### 6.2 别名解析

```typescript
// model.ts
switch (modelString) {
  case 'opusplan'
    return getDefaultSonnetModel() + '[1m]'  // plan 模式用 Opus
  case 'sonnet'
    return getDefaultSonnetModel()
  case 'haiku'
    return getDefaultHaikuModel()
  case 'best'
    return getBestModel()
}
```


## 七、Token 估算与截断

### 7.1 预检查

在调用 API 前，会估算 token 数量：

```typescript
// query.ts
if (!compactionResult && querySource !== 'compact' && ...) {
  const { isAtBlockingLimit } = calculateTokenWarningState(
    tokenCountWithEstimation(messagesForQuery) - snipTokensFreed,
    toolUseContext.options.mainLoopModel,
  )
  if (isAtBlockingLimit) {
    yield createAssistantAPIErrorMessage({
      content PROMPT_TOO_LONG_ERROR_MESSAGE,
      error 'invalid_request',
    })
    return { reason 'blocking_limit' }
  }
}
```

### 7.2 消息数量截断

当用户粘贴大量内容时，系统会自动截断：

```typescript
// attachments.ts
export function getAttachmentMessages(params {
  pastedContents? Record<number, PastedContent>
  ...
}) AttachmentMessage[]
```


## 八、架构图

```
用户输入
    │
    ▼
processUserInput(input)
    │
    ├──► processSlashCommand()  ──► shouldQuery = false
    │                                    │
    │                             返回 resultText
    │                             (命令已在本地处理)
    │
    ▼
processUserInputBase()
    │
    ├──► createUserMessage()  ──► UserMessage
    ├──► getAttachmentMessages() ──► AttachmentMessages
    └──► fetchSystemPromptParts() ──► SystemPrompt
                                              │
                                              ▼
                                         asSystemPrompt()
                                              │
                                              ▼
                                    systemPrompt + userContext + systemContext
                                              │
                                              ▼
                                    query() 主循环
```


## 九、设计哲学

### 9.1 为什么 slash 命令要在预处理阶段处理？

Slash 命令（如 `/commit`、`/model`）是**元命令**，不是发送给 LLM 的内容。它们的存在让用户可以在对话中执行操作而不污染对话历史。处理它们的正确位置是**预处理阶段**，而不是让 LLM 看到并忽略它们。

### 9.2 为什么需要 sideQuery？

不是所有 LLM 调用都需要完整的 query 循环：
- 模型验证只需要一个 `max_tokens=1` 的调用
- 摘要生成需要 fork 一个独立 agent
- 这些场景使用 `sideQuery` 更高效


## 总结

Claude Code 的提示词预处理是一个**多层组装的管道**：

1. **processUserInput**：入口，处理 slash 命令和 UI 展示
2. **processUserInputBase**：核心，构建消息和上下文
3. **fetchSystemPromptParts**：组装 system prompt 的三个组件
4. **sideQuery**：轻量辅助查询，不走完整循环
5. **normalizeModelStringForAPI**：规范化模型名，移除 context 后缀

理解这个管道，就能理解为什么 Claude Code 可以：
- 在处理 slash 命令时保持对话完整
- 在验证模型时不影响主对话
- 在构建 prompt 时灵活组合多个来源


