# 内部员工特权（ANT）：USER_TYPE === 'ant' 的门控体系

## 导言

Claude Code 的源码中散布着大量 `process.env.USER_TYPE === 'ant'` 的条件分支。这些分支不是 Bug，而是**构建时特性门控**——Claude Code 区分内部员工（ANT）和外部用户，为前者解锁额外工具、隐藏模型代号名、自动检测公开仓库等特权。

理解这套体系，不仅能看清 Claude Code 的内外用户分层设计，还能理解为什么某些"功能"在外部版本中完全找不到入口。本文从源码层面解析 ANT 门控的完整面貌。


## 一、USER_TYPE 的本质：编译期常量

### 1.1 什么是 USER_TYPE

`USER_TYPE` 是一个**构建时注入的环境变量**，通过 `--define process.env.USER_TYPE='ant'` 等方式在编译时确定。这意味着：

- 外部构建（external build）：`USER_TYPE` 永远不是 `'ant'`
- 内部构建（ant build）：`USER_TYPE === 'ant'`
- 所有 `USER_TYPE === 'ant'` 的检查在外部构建中被**常量折叠**（dead-code elimination）消除

源码注释明确说明了这一点：

```typescript
// src/utils/undercover.ts
// All code paths are gated on process.env.USER_TYPE === 'ant'. Since USER_TYPE is
// a build-time --define, the bundler constant-folds these checks and dead-code-
// eliminates the ant-only branches from external builds. In external builds every
// function in this file reduces to a trivial return.
```

这带来一个重要结论：**ANT 分支的代码在外部构建中不存在**，不会增加二进制体积，不会影响运行时性能。

### 1.2 如何模拟 ANT 环境

如果你想在外部构建中测试 ANT 相关的代码路径，可以通过编译时 define 注入：

```bash
# 模拟 ANT 构建（需要重新编译）
claude-code --define process.env.USER_TYPE=ant
```


## 二、ANT 专享工具

### 2.1 工具门控一览

以下工具在 `src/tools.ts` 中被 `USER_TYPE === 'ant'` 条件加载：

```typescript
// src/tools.ts
SkillTool,
EnterPlanModeTool,
...(process.env.USER_TYPE === 'ant' ? [ConfigTool]  []),      // 配置工具
...(process.env.USER_TYPE === 'ant' ? [TungstenTool]  []),    // 钨工具（会话管理）
...(SuggestBackgroundPRTool ? [SuggestBackgroundPRTool]  []),  // PR 建议（无 ANT 门控）
...(process.env.USER_TYPE === 'ant' && REPLTool ? [REPLTool]  []),  // REPL 工具
```

三个被完全门控的工具：

| 工具 | 源码位置 | ANT 专属原因 |
|------|---------|-------------|
| **ConfigTool** | `src/tools/ConfigTool/` | 允许 Claude 修改 settings.json、企业策略等敏感配置 |
| **TungstenTool** | `src/tools/TungstenTool/` | 内部工作流工具，与内部基础设施集成 |
| **REPLTool** | `src/tools/REPLTool/` | REPL 交互模式，用于内部调试和语音模式 |

### 2.2 SuggestBackgroundPRTool 的特殊性

值得注意的是，`SuggestBackgroundPRTool` 在源码中也被 `USER_TYPE === 'ant'` 包裹：

```typescript
// src/tools.ts
const SuggestBackgroundPRTool =
  process.env.USER_TYPE === 'ant'
    ? require('./tools/SuggestBackgroundPRTool/SuggestBackgroundPRTool.js')
        .SuggestBackgroundPRTool
     null
```

但它在工具注册时**没有再次检查 ANT**（`src/tools.ts`）：

```typescript
...(SuggestBackgroundPRTool ? [SuggestBackgroundPRTool]  []),
```

这意味着在 ANT 构建中，`SuggestBackgroundPRTool` 会被注册；在外部构建中，`SuggestBackgroundPRTool` 为 `null`，不会被注册。

### 2.3 工具在 restored build 中的状态

**TungstenTool 完全是一个 stub**：

```typescript
// src/tools/TungstenTool/TungstenTool.ts
export const TungstenTool = buildTool({
  name 'tungsten',
  userFacingName() {
    return 'Tungsten'
  },
  async description() {
    return 'Unavailable in restored development build.'
  },
  isEnabled() {
    return false
  },
  async call() {
    return { data { ok false } }
  },
})
```

这与 `ant-computer-use-mcp` shim 的处理方式一致——在 restored build 中，内部工具被有意 stub 化，不提供实际功能。


## 三、模型代号屏蔽（maskModelCodename）

### 3.1 为什么需要屏蔽模型代号

ANT 用户（Anthropic 内部员工）在公开仓库工作时，如果 Claude Code 告诉模型"你是 Opus 4.6"或"你在用 Capybara"，这些信息会泄漏到公开的 commit message、PR 描述中。

解决方案：**屏蔽模型代号**，让模型只知道"我是 Claude"，而不知道具体是哪个内部代号。

### 3.2 屏蔽算法

```typescript
// src/utils/model/model.ts
function maskModelCodename(baseName string) string {
  // Mask only the first dash-separated segment (the codename), preserve the rest
  // e.g. capybara-v2-fast → cap*****-v2-fast
  const [codename = '', ...rest] = baseName.split('-')
  const masked =
    codename.slice(0, 3) + '*'.repeat(Math.max(0, codename.length - 3))
  return [masked, ...rest].join('-')
}
```

示例：

| 原始代号 | 屏蔽后 |
|---------|--------|
| `capybara-v2-fast` | `cap*****-v2-fast` |
| `tengu-4-6` | `ten***-4-6` |
| `claude-opus-4-6` | `cla****-opus-4-6` |

只屏蔽第一段（模型代号），保留后续的版本号等信息。

### 3.3 renderModelName 中的屏蔽逻辑

```typescript
// src/utils/model/model.ts
export function renderModelName(model ModelName) string {
  const publicName = getPublicModelDisplayName(model)
  if (publicName) {
    return publicName
  }
  if (process.env.USER_TYPE === 'ant') {
    const resolved = parseUserSpecifiedModel(model)
    const antModel = resolveAntModel(model)
    if (antModel) {
      const baseName = antModel.model.replace(/\[1m\]$/i, '')
      const masked = maskModelCodename(baseName)
      const suffix = has1mContext(resolved) ? '[1m]'  ''
      return masked + suffix
    }
    if (resolved !== model) {
      // ... handle alias resolution
    }
  }
  return model
}
```

ANT 用户的模型名称会被 `maskModelCodename()` 处理后再显示，而非 ANT 用户直接看到真实代号（如 `claude-opus-4-6`）。


## 四、Undercover 模式：公开仓库自动隐身

### 4.1 什么是 Undercover

Undercover 是 Claude Code 为 Anthropic 员工提供的一套**安全机制**，在向公开仓库提交代码时自动激活：

- 隐藏模型代号
- 隐藏 Anthropic 内部信息
- 隐藏项目名称
- 强制使用人类风格的 commit message

### 4.2 激活条件

```typescript
// src/utils/undercover.ts
export function isUndercover() boolean {
  if (process.env.USER_TYPE === 'ant') {
    if (isEnvTruthy(process.env.CLAUDE_CODE_UNDERCOVER)) return true
    // Auto active unless we've positively confirmed we're in an allowlisted
    // internal repo. 'external', 'none', and null (check not yet run) all
    // resolve to ON. The check is primed in setup.ts; only 'internal' → OFF.
    return getRepoClassCached() !== 'internal'
  }
  return false
}
```

三种激活方式：

| 方式 | 条件 | 说明 |
|------|------|------|
| 强制开启 | `CLAUDE_CODE_UNDERCOVER=1` | 无论是否内部仓库都开启 |
| 自动模式 | `USER_TYPE === 'ant'` 且未在内部仓库 | 默认开启 |
| 关闭 | `USER_TYPE === 'ant'` 且在内部仓库 | `getRepoClassCached() === 'internal'` 时关闭 |

注意：**外部用户永远无法启用 Undercover**——`isUndercover()` 对非 ANT 用户直接返回 `false`。

### 4.3 Undercover 指令

当 Undercover 激活时，`getUndercoverInstructions()` 返回以下指令：

```typescript
// src/utils/undercover.ts
export function getUndercoverInstructions() string {
  if (process.env.USER_TYPE === 'ant') {
    return `## UNDERCOVER MODE — CRITICAL

You are operating UNDERCOVER in a PUBLIC/OPEN-SOURCE REPOSITORY.
Your commit messages, PR titles, and PR bodies MUST NOT contain ANY
Anthropic-internal information. Do not blow your cover.

NEVER include in commit messages or PR descriptions
- Internal model codenames (animal names like Capybara, Tengu, etc.)
- Unreleased model version numbers (e.g., opus-4-7, sonnet-4-8)
- Internal repo or project names (e.g., claude-cli-internal, anthropics/…)
- Internal tooling, Slack channels, or short links (e.g., go/cc, #claude-code-…)
- The phrase "Claude Code" or any mention that you are an AI
- Any hint of what model or version you are
- Co-Authored-By lines or any other attribution
...`
  }
  return ''
}
```

这些指令被注入到 system prompt 中，影响 Claude 的输出行为。


## 五、ConfigTool 的 ANT 专属配置项

### 5.1 ANT 独有的配置项

`ConfigTool/supportedSettings.ts` 中有一个 ANT 专属的配置项：

```typescript
// src/tools/ConfigTool/supportedSettings.ts
...(process.env.USER_TYPE === 'ant'
  ? {
      classifierPermissionsEnabled {
        // ANT 专属：分类器权限启用开关
      },
      // ... 可能还有其他 ANT 专属配置
    }
   []),
```

这意味着某些高级配置项只有 Anthropic 内部员工才能通过 ConfigTool 修改。


## 六、KAIROS 的 ANT 关联

### 6.1 --session-id 是 ANT 专属

`--session-id` 参数用于 KAIROS 的持久会话恢复，但这个功能被 KAIROS feature flag 和 USER_TYPE === 'ant' 双重门控。

### 6.2 Assistant 模块的延迟加载

```typescript
// src/main.tsx
const assistantModule = feature('KAIROS') ? require('./assistant/index.js')  null;
const kairosGate = feature('KAIROS') ? require('./assistant/gate.js')  null;
```

KAIROS 的 assistant 模块只在 KAIROS feature flag 启用时加载，与 USER_TYPE === 'ant' 是两个独立的门控维度。


## 七、ANT 与外部用户的功能差异总览

| 功能 | ANT 专属 | 说明 |
|------|---------|------|
| **ConfigTool** | 是 | 修改 settings.json 等敏感配置 |
| **TungstenTool** | 是 | 内部工作流集成（restored build 中 stub） |
| **REPLTool** | 是 | REPL 交互模式 |
| **SuggestBackgroundPRTool** | 是 | 后台 PR 建议 |
| **模型代号屏蔽** | 是 | `maskModelCodename()` 处理 |
| **Undercover 模式** | 是 | 公开仓库自动隐身 |
| **classifierPermissionsEnabled** | 是 | 分类器权限配置 |
| **--session-id** | 部分 | KAIROS + feature flag 双重门控 |


## 八、设计哲学

### 8.1 为什么用构建时常量而非运行时环境变量？

`USER_TYPE` 是 `--define` 编译时注入，而非 `process.env.USER_TYPE` 运行时读取。这样做的好处：

1. **零运行时开销**：外部构建中所有 ANT 分支被常量折叠，条件判断变成 no-op
2. **代码清晰**：开发者不需要在每个 ANT 分支内额外做运行时检查
3. **安全性**：外部构建的二进制中根本不包含 ANT 专属代码

### 8.2 为什么某些工具在 restored build 中被 stub？

这是有意为之——不是为了隐藏，而是这个版本**不包含完整的内部工具链实现**。TungstenTool 依赖的内部基础设施（如 tmux 集成、特殊的终端抽象）在 restored build 中不可用，所以被 stub 化。

### 8.3 Undercover 的安全假设

Undercover 假设**即使 Claude Code 运行在内部仓库，它也可能意外地向公开仓库提交代码**（例如 `/tmp` 下的临时目录不是 git checkout）。因此：
- 没有"强制关闭"选项（no force-off）
- 默认行为是"保持隐身"（safe default is ON）


## 总结

Claude Code 的 ANT 门控体系是一套**构建时特性开关**，通过 `USER_TYPE === 'ant'` 实现内外用户的功能分层：

1. **ANT 专属工具**：ConfigTool、TungstenTool、REPLTool、SuggestBackgroundPRTool——这些工具在外部构建中完全不存在
2. **模型代号屏蔽**：`maskModelCodename()` 将 `capybara-v2-fast` 变成 `cap*****-v2-fast`，防止内部代号泄漏
3. **Undercover 模式**：自动检测公开仓库，注入隐藏指令，防止 commit/PR 中出现 Anthropic 内部信息
4. **零运行时开销**：构建时常量折叠确保 ANT 代码在外部构建中完全消除

