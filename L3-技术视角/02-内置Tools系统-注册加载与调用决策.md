# 内置 Tools 系统：注册加载与调用决策

## 导言

大多数 AI 编程工具将"工具"理解为"可以调用的命令"，本质上是函数指针的集合。Claude Code 的设计走了不同的路：**每一个工具都是一个完整的对象**，自带描述 Schema、输入验证、权限检查、并发安全策略，以及一个决定自身是否"延迟加载"的能力标志。

这个设计的影响是深远的：工具不是被"调用"的，而是被"编排"的；工具不是简单映射到 bash 命令的，而是有自我约束能力的智能体。

---

## 一、Tool 接口：超过 40 个字段的契约

Tool 接口定义了工具的契约，粗略数一下有超过 40 个属性和方法，是整个系统中最大最复杂的接口之一。

### 1.1 核心接口结构

```typescript
// Tool 接口的核心字段
export type Tool<
  Input extends AnyObject = AnyObject,
  Output = unknown,
  P extends ToolProgressData = ToolProgressData,
> = {
  readonly name: string
  readonly inputSchema: Input   // Zod schema — 核心类型契约

  call(
    args: z.infer<Input>,
    context: ToolUseContext,
    canUseTool: CanUseToolFn,
    parentMessage: AssistantMessage,
    onProgress?: ToolCallProgress<P>,
  ): Promise<ToolResult<Output>>

  description(
    input: z.infer<Input>,
    options: {...}
  ): Promise<string>

  // === 能力标志 ===
  readonly shouldDefer?: boolean    // 是否需要 ToolSearch 二次发现
  readonly alwaysLoad?: boolean      // 永不延迟，turn 1 立即可用
  readonly isConcurrencySafe?: (input) => boolean  // 是否可并行执行
  isReadOnly?(input): boolean
  isDestructive?(input): boolean    // 是否为不可逆操作
  isMcp?: boolean                   // 是否为 MCP 工具
  interruptBehavior?(): 'cancel' | 'block'  // 运行中收到新消息怎么办

  // === 安全相关 ===
  checkPermissions(input, context): Promise<PermissionResult>
  validateInput?(input, context): Promise<ValidationResult>

  // === UI 展示相关 ===
  isSearchOrReadCommand?(input): { isSearch, isRead, isList }
  isOpenWorld?(input): boolean
  requiresUserInteraction?(): boolean

  // === 观察者注入 ===
  backfillObservableInput?(input: Record<string, unknown>): void
}
```

这个接口的设计哲学是：**工具是"有行为的对象"，不是"带名字的函数"**。

### 1.2 能力标志系统

每个 Tool 实例通过实现特定方法向系统声明自己的能力：

| 方法/属性 | 含义 | 系统如何使用 |
|-----------|------|------------|
| `isConcurrencySafe()` | 可与其他工具并行执行 | `StreamingToolExecutor` 决定是否并发执行 |
| `isDestructive()` | 不可逆操作（删除、覆写） | 权限提示时显示警告 |
| `interruptBehavior()` | 工具运行时收到新消息 | `'cancel'` 中断；`'block'` 等待 |
| `shouldDefer` | 需要 ToolSearch 二次发现 | API 调用时使用 `defer_loading: true` |
| `alwaysLoad` | 无条件立即加载 | 即使 ToolSearch 开启也出现在初始 prompt |
| `maxResultSizeChars` | 结果大小上限 | 超过则持久化到磁盘，API 收到路径引用 |

---

## 二、工具注册：静动态结合的注册表

### 2.1 静态注册表：`getAllBaseTools()`

所有内置工具通过 `getAllBaseTools()` 函数注册：

```typescript
export function getAllBaseTools(): Tools {
  return [
    AgentTool,
    TaskOutputTool,
    BashTool,
    ...(hasEmbeddedSearchTools() ? [] : [GlobTool, GrepTool]),  // 条件排除
    ExitPlanModeV2Tool,
    FileReadTool,
    FileEditTool,
    FileWriteTool,
    NotebookEditTool,
    WebFetchTool,
    TodoWriteTool,
    WebSearchTool,
    TaskStopTool,
    AskUserQuestionTool,
    SkillTool,
    EnterPlanModeTool,
    // ANT 特权工具、实验性工具通过 feature flag 条件加载
    ...(isTodoV2Enabled() ? [TaskCreateTool, TaskGetTool, TaskUpdateTool, TaskListTool] : []),
    ...(SleepTool ? [SleepTool] : []),
    ...cronTools,
    ListMcpResourcesTool,
    ReadMcpResourceTool,
    ...(isToolSearchEnabledOptimistic() ? [ToolSearchTool] : []),
  ]
}
```

**总计约 22 个内置工具**，但通过 `process.env` 和 feature flag 可以动态增减。

### 2.2 动态组装：`getTools()` 和 `assembleToolPool()`

`getAllBaseTools()` 只是"所有可能工具的注册表"。实际使用时还需要两道过滤：

```typescript
export function getTools(permissionContext: ToolPermissionContext): Tools {
  // 第一道：SIMPLE 模式只保留 3 个基础工具
  if (isEnvTruthy(process.env.CLAUDE_CODE_SIMPLE)) {
    return filterToolsByDenyRules([BashTool, FileReadTool, FileEditTool], permissionContext)
  }

  // 第二道：REPL 模式下隐藏原始工具（通过 VM 间接暴露）
  if (isReplModeEnabled()) {
    allowedTools = allowedTools.filter(tool => !REPL_ONLY_TOOLS.has(tool.name))
  }

  // 第三道：检查每个工具的 isEnabled()（运行时能力检查）
  const isEnabled = allowedTools.map(_ => _.isEnabled())
  return allowedTools.filter((_, i) => isEnabled[i])
}

export function assembleToolPool(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools,  // MCP 工具由 MCP 服务器动态提供
): Tools {
  const builtInTools = getTools(permissionContext)
  const allowedMcpTools = filterToolsByDenyRules(mcpTools, permissionContext)

  // 排序 + 去重：built-in 优先
  return uniqBy(
    [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
    'name',
  )
}
```

**去重策略**：当内置工具和 MCP 工具同名时，内置工具优先（`uniqBy` 保留第一个出现的，即 built-in）。

### 2.3 工具分层

```
assembleToolPool
    │
    ├── 内置工具（Built-in Tools）
    │   ├── 核心文件操作：BashTool, FileReadTool, FileEditTool, FileWriteTool
    │   ├── 搜索：GlobTool, GrepTool
    │   ├── Web：WebFetchTool, WebSearchTool
    │   ├── Agent 管理：AgentTool, TaskOutputTool, TaskStopTool
    │   ├── 交互：AskUserQuestionTool
    │   ├── 计划：EnterPlanModeTool, ExitPlanModeV2Tool
    │   └── 其他：TodoWriteTool, NotebookEditTool, LSPTool ...
    │
    └── MCP 工具（动态加载）
        ├── mcp-tools from MCP servers
        └── 内置 MCP：ListMcpResourcesTool, ReadMcpResourceTool
```

---

## 三、Deferred 机制：工具的按需加载

### 3.1 为什么要延迟加载？

Claude Code 支持用户动态连接任意数量的 MCP 服务器。每个 MCP 服务器可能暴露数十个工具。如果每次 API 调用都将所有工具 schema 发送给 LLM：
1. **Prompt 膨胀**：大量 token 浪费在工具描述上
2. **L LM 困惑**：过多的工具选择增加决策难度
3. **上下文窗口压力**：200K 上下文很快被工具 schema 占满

**解决思路**：只在模型真正需要某个工具时才加载它的 schema。

### 3.2 `shouldDefer` 机制

`shouldDefer` 是 Tool 接口的一个可选标志：

```typescript
/**
 * When true, this tool is deferred (sent with defer_loading: true) and requires
 * ToolSearch to be used before it can be called.
 */
readonly shouldDefer?: boolean
```

当 `shouldDefer: true` 时，API 调用时使用 `defer_loading: true` 标记该工具，**不发送完整 schema**。模型必须先调用 `ToolSearchTool` 来"发现"这个工具，之后才能使用。

### 3.3 `isDeferredTool()` 决策函数

真正决定一个工具是否应该延迟的是 `isDeferredTool()` 函数：

```typescript
export function isDeferredTool(tool: Tool): boolean {
  // 1. alwaysLoad 显式退出延迟
  if (tool.alwaysLoad === true) return false

  // 2. MCP 工具始终延迟（MCP 是外部扩展，系统不了解其 schema）
  if (tool.isMcp === true) return true

  // 3. ToolSearchTool 本身永不延迟
  if (tool.name === TOOL_SEARCH_TOOL_NAME) return false

  // 4. AgentTool 在 FORK_SUBAGENT 实验中不延迟
  if (feature('FORK_SUBAGENT') && tool.name === AGENT_TOOL_NAME) {
    if (isForkSubagentEnabled()) return false
  }

  // 5. Brief/SendUserFile 在 KAIROS 模式中不延迟
  if (tool.name === BRIEF_TOOL_NAME) return false

  // 6. 默认行为
  return tool.shouldDefer === true
}
```

这个函数的精妙之处：**工具本身声明的 `shouldDefer` 只是一个默认值**，实际决策由 `isDeferredTool()` 根据运行时 feature flag 和工具类型综合决定。

### 3.4 延迟工具的发现流程

当模型尝试调用一个未被加载的延迟工具时：

```typescript
export function buildSchemaNotSentHint(tool, messages, tools): string | null {
  if (!isToolSearchEnabledOptimistic()) return null
  if (!isToolSearchToolAvailable(tools)) return null
  if (!isDeferredTool(tool)) return null
  const discovered = extractDiscoveredToolNames(messages)
  if (discovered.has(tool.name)) return null

  // 返回提示：告诉模型需要先调用 ToolSearchTool 来加载这个工具
  return (
    `This tool's schema was not sent to the API. ` +
    `Load the tool first: call ${TOOL_SEARCH_TOOL_NAME} with query "select:${tool.name}", then retry.`
  )
}
```

这是一个**自我修正机制**：当模型错误地尝试调用延迟工具时，系统返回一个提示，指导模型先用 `ToolSearchTool` 加载工具，然后再重试。整个过程对用户透明。

### 3.5 工具搜索的三种模式

```typescript
// toolSearch.ts
export type ToolSearchMode = 'tst' | 'tst-auto' | 'standard'

// 'tst'（tool search tool）：所有延迟工具都必须先发现
// 'tst-auto'：当 MCP 工具描述总 token 超过阈值时才启用延迟
// 'standard'：不使用延迟，所有工具 schema 都直接发送
```

---

## 四、权限系统：工具级别的决策链

### 4.1 三元权限决策

当模型尝试调用一个工具时，`canUseTool()` 函数（hooks/useCanTool.tsx）返回一个三元决策：

```typescript
type PermissionDecision =
  | { behavior: 'allow' }      // 直接执行
  | { behavior: 'deny'; reason: string }  // 直接拒绝
  | { behavior: 'ask' }        // 弹出权限对话框询问用户
```

### 4.2 Tool 级别的权限钩子

每个工具可以定义自己的 `checkPermissions()` 方法：

```typescript
checkPermissions(
  input: z.infer<Input>,
  context: ToolUseContext,
): Promise<PermissionResult>
```

这允许工具实现**自有的权限判断逻辑**，例如 `BashTool` 会检查命令是否涉及 `sudo`、`rm -rf` 等高危操作。

### 4.3 PermissionMode 影响权限决策

```typescript
type PermissionMode =
  | 'manual'      // 所有操作需要用户确认
  | 'auto'        // 根据 alwaysAllowRules 自动决定
  | 'autoApprove' // 跳过确认，直接执行
  | 'bypass'      // 完全绕过权限检查（慎用）
```

---

## 五、并发执行：StreamingToolExecutor

当多个工具被并行调用时，`StreamingToolExecutor`（services/tools/StreamingToolExecutor.ts）负责协调：

```typescript
// 关键决策：哪些工具可以并行？
const canRunInParallel = tool.isConcurrencySafe?.(input) ?? false

// StreamingToolExecutor 在工具流式输出完成时
// 立即将结果返回给 query 循环，
// 不等待其他并行工具完成
```

`isConcurrencySafe` 的设计使得 Claude Code 可以在单轮对话中同时执行多个不相关的工具（如同时读取多个文件），大幅减少等待时间。

---

## 六、工具的自我约束能力

Tool 接口中最独特的设计是**工具的自我约束**：

1. **`maxResultSizeChars`**：工具自己声明结果大小上限，超过则持久化到磁盘，避免大结果撑爆上下文
2. **`interruptBehavior()`**：工具自己声明被中断时的行为（`cancel` vs `block`）
3. **`isReadOnly()` / `isDestructive()`**：工具自己声明是否为只读或破坏性操作
4. **`validateInput()`**：工具自己验证输入的合法性

这意味着**工具不只是被系统调用的被动对象，而是主动声明自身能力和约束的主动参与者**。系统在此基础上做决策，而不是拥有所有决策权。

---

## 七、架构图

```
                    ┌──────────────────────────────────┐
                    │       assembleToolPool()          │
                    │  1. getAllBaseTools() 获取注册表  │
                    │  2. getTools() 过滤（SIMPLE/REPL）│
                    │  3. filterToolsByDenyRules()     │
                    │  4. 合并 MCP 工具（去重）          │
                    └──────────────┬───────────────────┘
                                   │
              ┌────────────────────┴────────────────────┐
              ▼                                         ▼
   ┌─────────────────────┐                  ┌──────────────────────────┐
   │  Built-in Tools     │                  │   MCP Tools (动态)          │
   │  ~22 个静态注册     │                  │   来自 MCP servers          │
   │  isEnabled() 检查   │                  │   isMcp = true             │
   └─────────────────────┘                  └──────────────────────────────┘
                                   │
                    ┌──────────────┴──────────────────┐
                    │     API 调用 (callModel)           │
                    │  ToolSchemas sent to LLM          │
                    │  defer_loading: true for deferred │
                    └──────────────┬───────────────────┘
                                   │
              ┌─────────────────────┴──────────────────────┐
              ▼                                             ▼
   ┌─────────────────────┐                    ┌────────────────────────┐
   │  模型输出 tool_use   │                    │  LLM 尝试调用延迟工具   │
   │  ┌───────────────┐  │                    │  → buildSchemaNotSentHint │
   │  │ name: BashTool │  │                    │  → 提示用 ToolSearch   │
   │  │ input: {...}  │  │                    └────────────────────────┘
   │  └───────────────┘  │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────────────────┐
   │   canUseTool() 权限检查           │
   │   allow / deny / ask             │
   └──────────┬────────────────────────┘
              │
              ▼
   ┌─────────────────────────────────┐
   │   StreamingToolExecutor          │
   │   并发执行 isConcurrencySafe     │
   │   串行执行 非并发安全工具          │
   └─────────────────────────────────┘
```

---

## 总结

Claude Code 的工具系统远不止"命令注册表"。它的核心设计特点：

1. **Tool 是对象而非函数**：每个工具主动声明自己的 schema、约束、能力标志和安全策略
2. **Deferred 加载是全局策略**：通过 `isDeferredTool()` 集中决策，而非每个工具自行决定
3. **MCP 是原生集成**：MCP 工具和内置工具在同一个池中管理，共享排序、去重、权限检查
4. **权限是分层决策**：系统级别 `PermissionMode` + 工具级别 `checkPermissions()` + 运行时 `canUseTool()`
5. **并发执行是主动声明**：工具通过 `isConcurrencySafe` 自己告诉系统是否安全并行

这套设计的代价是 Tool 接口极度复杂——但它换来的灵活性是：系统可以在运行时动态决定工具池的组成、权限级别和执行策略，而不需要修改任何工具本身的代码。
