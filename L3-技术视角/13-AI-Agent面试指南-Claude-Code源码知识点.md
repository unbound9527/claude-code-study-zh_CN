# AI Agent 面试想拿高薪？Claude Code 源码中的这些知识对你一定有帮助

## 导言

AI Agent 是当下最热门的方向，但面试中真正能拉开差距的，不是你会调 API，而是你理解**生产级 Agent 系统如何处理递归、并发、容错和资源管理**。

Claude Code 是目前最复杂的开源 Agent 系统之一。它的代码中藏着大厂面试官最想看到的知识点：如何设计递归查询循环、如何做工具的延迟加载和权限控制、如何实现多实例协调、如何在有限上下文窗口中做自适应压缩。本文提炼 12 篇分析中最有面试价值的 8 个主题，每个主题附上核心答案和代码依据。


## 一、递归查询循环：while(true) + 状态替换而非函数递归

### 面试高频问题

> "大模型 Agent 的主循环一般怎么写？如果用函数递归有什么问题？"

### 标准答案

Claude Code 的 query 循环是 `while(true)` 模式，不是函数递归：

```typescript
// src/query.ts约 line 200
async function* query(params QueryParams) AsyncGenerator<SDKMessage> {
  let state = initialState
  while (true) {
    // 1. 调用模型
    const response = await callModel(state.messages)

    // 2. 处理响应（可能是 tool_call 或 text）
    for (const content of response.content) {
      if (content.type === 'tool_use') {
        // 执行工具
        const result = await executeTool(content.name, content.input)
        // 将工具结果追加到消息
        state = appendToolResult(state, content.name, result)
      } else if (content.type === 'text') {
        yield { type 'text', content content.text }
      }
    }

    // 3. 检查终止条件
    if (shouldStop(response)) break
  }
}
```

**为什么不用函数递归？**
1. **栈溢出风险**：递归深度取决于对话轮次，1000 轮对话会撑爆调用栈
2. **状态管理复杂**：每层递归需要闭包捕获局部状态，退出时状态丢失
3. **尾递归优化**：JS 引擎不保证尾递归优化

**while(true) + 状态对象的优势**：
- 状态在堆上，不受调用栈限制
- 每次迭代是平级的，可以精确控制流程
- message 数组不断增长，天然适合对话历史

### 面试加分项

> "Claude Code 的 query 是一个 AsyncGenerator，每个 yield 都是一个 SDKMessage。这种设计的精妙之处在于：调用方可以实时展示中间步骤（tool_call 执行中、tool_result 返回），而不需要等整个 query 结束才能给用户反馈。"


## 二、工具系统：isDeferred 延迟加载与权限分级

### 面试高频问题

> "当 Agent 有几十个工具时，怎么避免每次 API 调用都把全部工具 schema 发给模型？"

### 标准答案

Claude Code 使用 **deferred 工具加载模式**：

```typescript
// src/tools/ToolSearchTool/prompt.ts
function isDeferredTool(tool Tool) boolean {
  // MCP 工具永远是 deferred
  if (tool.isMcp) return true
  // 其他工具由 shouldDefer() 方法决定
  return tool.shouldDefer?.() ?? false
}
```

**两层加载机制**：
1. **第一层**：注册所有工具，但只有 `isDeferred = false` 的工具被直接加载
2. **第二层**：当模型请求一个 deferred 工具时，通过 **ToolSearchTool** 二次发现

```typescript
// 工具池组装
function assembleToolPool(tools Tool[], query string) {
  const eagerTools = tools.filter(t => !t.isDeferred)
  const deferredTools = tools.filter(t => t.isDeferred)

  // 模型只能看到 eager 工具
  return {
    tools eagerTools,
    // 当模型需要 deferred 工具时，触发 ToolSearchTool
    toolSearchTrigger deferredTools
  }
}
```

### 面试加分项

> "isDeferred 的设计哲学是：按需加载而非全部预加载。这和操作系统和数据库的 page fault、lazy loading 一脉相承。Claude Code 的工具系统有 22+ 内置工具，如果每次都发全部 schema，光工具描述就可能消耗 10K+ tokens。"


## 三、权限系统：allow / deny / ask 三元拦截链

### 面试高频问题

> "Agent 执行危险操作（如删除文件、执行 shell 命令）时，怎么做安全控制？"

### 标准答案

Claude Code 实现了一套 **Tool 级别的权限拦截链**：

```typescript
// src/types/permissions.ts
export type PermissionBehavior = 'allow' | 'deny' | 'ask'

type PermissionDecision<T> =
  | { behavior 'allow'; updatedInput? T }
  | { behavior 'deny'; reason string }
  | { behavior 'ask' }  // 弹出对话框询问用户
```

**三元决策流程**：

```
canUseTool(tool, input)
    │
    ├──► PermissionMode 检查（总开关）
    │         │
    │         ├── 'acceptEdits' → 直接 allow
    │         ├── 'dontAsk'     → 直接 deny
    │         └── 'default'     → 继续判断
    │
    ├──► alwaysAllowRules（白名单）
    │         └── 特定工具自动 allow
    │
    ├──► Tool.checkPermissions()（工具自定义）
    │         └── BashTool 检查 sudo、rm -rf 等
    │
    └──► 三元结果：allow / deny / ask
              │
              ├── allow → 执行
              ├── deny  → 返回拒绝原因
              └── ask  → 弹出交互对话框
```

### 面试加分项

> "很多面试者会说'我用了白名单'，但 Claude Code 的设计更精细：同一个工具（如 BashTool），不同命令有不同权限级别。`ls` 可以自动 allow，`rm -rf /` 必须 ask，`sudo rm` 直接 deny。这不是简单的工具级别开关，而是**工具 + 操作**的二元组权限映射。"


## 四、上下文压缩：选择性保留而非全量摘要

### 面试高频问题

> "大模型上下文窗口是有限的，对话历史无限增长时怎么办？"

### 标准答案（常见错误）

很多面试者会说"满了就截断旧消息"——这是错误答案，因为：
1. 早期对话可能包含关键约束（"这个项目用 Python 3.8"）
2. 随意截断会破坏上下文完整性

### 标准答案（正确做法）

Claude Code 的 AutoCompact 是**选择性保留 + 摘要替换**：

```typescript
// autoCompact.ts
export const AUTOCOMPACT_BUFFER_TOKENS = 13_000

export function getAutoCompactThreshold(model string) number {
  const effectiveContextWindow = getEffectiveContextWindowSize(model)
  // effectiveContextWindow = contextWindow - 20,000 (摘要输出预留)
  return effectiveContextWindow - AUTOCOMPACT_BUFFER_TOKENS
}
```

对于 200K context 的 Sonnet 4.6：**在 167K tokens 时就触发压缩**，留出 13K 的操作空间。

**压缩后的保留策略**（`compact.ts`）：

| 内容 | 保留策略 |
|------|---------|
| 最近 5 个文件 | 每个最多 5,000 tokens |
| Skills | 总预算 25,000 tokens |
| Plan attachments | 保持关联 |
| 压缩分界线 | `SystemCompactBoundary` |
| 早期消息 | 替换为摘要 UserMessage |

### 面试加分项

> "为什么保留最近 5 个文件而不是最近 5 条消息？因为代码搜索是 Claude Code 的核心场景。如果压缩时把所有历史文件引用都删掉，模型就无法知道某个函数上次被修改是什么时候。**文件引用比消息引用更持久**，这是 Claude Code 的设计洞察。"


## 五、Subagent 隔离：进程外 vs 进程内的资源边界

### 面试高频问题

> "如果你的 Agent 需要同时执行多个子任务，怎么做资源隔离？"

### 标准答案

Claude Code 有两种 subagent 创建方式：

```typescript
// src/tools/shared/spawnMultiAgent.ts

// 方式 1：tmux pane（进程外隔离）
export function spawnTeammate(sessionId string, cwd string) {
  // 每个 subagent 是独立的 tmux pane
  // 完全隔离：独立进程、独立内存、文件系统隔离
}

// 方式 2：in-process teammate（进程内隔离）
export function spawnInProcessTeammate() {
  // 通过 AbortController + mailbox 隔离
  // 共享进程但隔离 Task 级别状态
}
```

**隔离什么？**

```typescript
// src/utils/forkedAgent.ts
function createSubagentContext(parentState AppState) SubagentContext {
  return {
    // 共享：工作目录、git 状态、项目配置
    workingDirectory parentState.workingDirectory,
    gitContext parentState.gitContext,
    // 隔离：消息历史、工具调用计数、token 预算
    messages [],  // subagent 有自己的消息历史
    toolUseCount 0,
    budgetTracking fresh()
  }
}
```

### 面试加分项

> "很多面试者会提到'用 Docker 隔离'——这是正确的思路但不够细粒度。Claude Code 的 tmux 隔离是**会话级别**的，每个 subagent 有自己的 TTY、自己的环境变量、甚至自己的 `$HOME`。但同时共享工作目录，这让父 Agent 可以'看到'子 Agent 的输出文件。这是刻意设计的：隔离的是运行环境，共享的是协作上下文。"


## 六、多模型抽象：Provider 模式与能力覆盖

### 面试高频问题

> "如果你需要同时支持 Anthropic、AWS Bedrock、Google Vertex 三个模型供应商，怎么设计？"

### 标准答案

Claude Code 用 Provider 抽象层解决：

```typescript
// providers.ts
export function getAPIProvider() APIProvider {
  return isEnvTruthy(process.env.CLAUDE_CODE_USE_BEDROCK)
    ? 'bedrock'
     isEnvTruthy(process.env.CLAUDE_CODE_USE_VERTEX)
      ? 'vertex'
       isEnvTruthy(process.env.CLAUDE_CODE_USE_FOUNDRY)
        ? 'foundry'
         'firstParty'
}
```

**模型 ID 的 Provider 映射**：

```typescript
// configs.ts
export const ALL_MODEL_CONFIGS = {
  'claude-opus-4-6' {
    firstParty 'claude-opus-4-6',
    bedrock   'us.anthropic.claude-opus-4-6-v1',  // ARN 格式
    vertex    'claude-opus-4-6',
    foundry   'claude-opus-4-6',
  },
}
```

同一个模型在四个 Provider 上有**不同的 ID 格式**，通过中心化的 `getModelStrings()` 统一解析。

### 面试加分项

> "Provider 抽象的精髓不是'怎么支持多个供应商'，而是'怎么让调用方感受不到差异'。Claude Code 的 model 层做了两件事：1) 把 Provider 的差异在配置层消解；2) 把能力差异在运行时用 `get3PModelCapabilityOverride()` 覆盖。很多面试者只会写 if-else，但 Claude Code 用配置驱动：新增一个 Provider，只需要改配置文件，不需要改业务代码。"


## 七、Proactive Agent：SleepTool + Tick 机制的节电模式

### 面试高频问题

> "如果你的 Agent 需要长时间运行但没活干，怎么避免空转浪费资源？"

### 标准答案

Claude Code 的 Proactive Mode 使用 **SleepTool 休息机制**：

```typescript
// src/proactive/index.ts
export interface ProactiveState {
  active boolean      // 是否激活
  paused boolean      // 用户按 Esc 暂停
  contextBlocked boolean  // API 错误时阻塞 tick
}
```

```typescript
// 当没有待处理任务时
async function idleLoop() {
  while (true) {
    const hasWork = await checkForWork()
    if (!hasWork) {
      // 休息一段时间
      await SleepTool.call({
        duration randomBetween(minSleepDurationMs, maxSleepDurationMs)
      })
    }
    // 被唤醒后继续检查
    await waitForNextTick()
  }
}
```

### 面试加分项

> "这不是简单的 sleep，这是**事件驱动的唤醒机制**。Agent 在休息时完全不消耗 API，只有在收到 tick（定时器触发）或外部事件（用户输入）时才唤醒。这和 iOS/Android 的 background task 机制一脉相承：**最大化利用空闲时间做记忆整合**，最小化活跃时的资源消耗。"


## 八、Cron 调度器：Jitter 防雷群机制

### 面试高频问题

> "多实例部署的 Agent 系统，如何避免多个实例同时执行定时任务？"

### 标准答案

Claude Code 的 Cron 调度器使用 **确定性延迟（Jitter）防雷群**：

```typescript
// src/utils/cronJitterConfig.ts
// 循环任务：interval 的 10%，上限 15 分钟
// 一次性任务：在  和  施加最多 90 秒提前量
```

**防雷群原理**：

```
假设有 1000 个客户端同时订阅 "每天 8 执行"
没有 jitter：1000 个请求同时到达 → 服务器雪崩
有 jitter（interval 10% = 86.4s）：
  客户端 1 8 + 0s    → 8
  客户端 2 8 + 10s   → 8
  ...
  客户端 100 8 + 1000s → 8
```

Jitter 的计算是**基于 taskId 的确定性 hash**，同一个 taskId 每次计算出相同延迟，确保"exactly-once"语义。

### 面试加分项

> "这是分布式系统'防雷群'（thundering herd）问题的经典解法。Redis 的 `WAIT` 命令、Cassandra 的 `Gossip` 协议都用类似机制。但 Claude Code 做得更精细：**一次性任务和循环任务用不同的 jitter 策略**。循环任务用'延迟'，一次性任务用'提前'——这是因为循环任务不怕重复，一次性任务必须精确。"


## 总结：面试中的三个层次

### 层次一：知道是什么（能回答）

上述 8 个问题，能用自己的话复述 Claude Code 的做法，这是**及格线**。

### 层次二：知道为什么（能分析）

每个设计背后都有权衡：
- 为什么用 while(true) 而非递归？因为栈深度不可控
- 为什么 167K 就压缩？因为压缩本身也要 tokens
- 为什么工具用 deferred 而非全部加载？因为 schema 有成本

**能解释"为什么这样设计"，是面试官判断你是否有实战经验的关键**。

### 层次三：知道替代方案的代价（能批判）

> "Claude Code 的递归防护没有硬限制，只记录 depth 并在日志中警告。你认为这是设计缺陷还是设计选择？"

这类问题没有标准答案，但**能提出合理的 tradeoff 分析**，说明你具备架构思维，是面试官最想要的候选人。


