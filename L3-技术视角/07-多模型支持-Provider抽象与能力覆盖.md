# 多模型支持：Provider 抽象、别名系统与能力覆盖

## 导言

Claude Code 不仅仅是一个 Anthropic API 的客户端——它同时支持四种 Provider：Anthropic 官方（firstParty）、AWS Bedrock、Google Vertex 和 Azure Foundry。如何让同一套代码在四种完全不同的后端上运行？答案在于 Provider 抽象层。本文从代码层面解析这个设计。

---

## 一、四种 Provider 的识别

### 1.1 Provider 决策函数

```typescript
export function getAPIProvider(): APIProvider {
  return isEnvTruthy(process.env.CLAUDE_CODE_USE_BEDROCK)
    ? 'bedrock'
    : isEnvTruthy(process.env.CLAUDE_CODE_USE_VERTEX)
      ? 'vertex'
      : isEnvTruthy(process.env.CLAUDE_CODE_USE_FOUNDRY)
        ? 'foundry'
        : 'firstParty'
}

export type APIProvider = 'firstParty' | 'bedrock' | 'vertex' | 'foundry'
```

通过环境变量切换 Provider，不需要代码改动。这使得 Claude Code 可以作为**统一的 CLI 工具**，对接不同的后端服务。

---

## 二、模型 ID 的 Provider 映射

### 2.1 为什么同一模型有四个不同的 ID

在 `configs.ts` 中，每个模型都有四个版本：

```typescript
export const ALL_MODEL_CONFIGS = {
  // ...
  'claude-opus-4-6': {
    firstParty: 'claude-opus-4-6',
    bedrock:   'us.anthropic.claude-opus-4-6-v1',
    vertex:    'claude-opus-4-6',
    foundry:   'claude-opus-4-6',
  },
  'claude-sonnet-4-6': {
    firstParty: 'claude-sonnet-4-6',
    bedrock:   'us.anthropic.claude-sonnet-4-6',
    vertex:    'claude-sonnet-4-6',
    foundry:   'claude-sonnet-4-6',
  },
}
```

这是因为：
- **Anthropic 官方**：使用短 ID（`claude-opus-4-6`）
- **Bedrock**：使用 ARN 格式前缀（`us.anthropic.claude-opus-4-6-v1`）
- **Vertex**：使用带日期后缀的格式（`claude-opus-4-6`）
- **Foundry**：使用不带前缀的基础名称（`claude-opus-4-6`）

### 2.2 模型字符串解析

当用户指定一个模型时，系统需要知道如何将其转换为当前 Provider 的正确 ID：

```typescript
export function getModelStrings(): ModelStrings {
  const provider = getAPIProvider()
  return applyModelOverrides(getBuiltinModelStrings(provider))
}
```

这意味着每次获取模型字符串时，都会根据当前 Provider 返回正确格式的 ID。

### 2.3 Bedrock 的特殊处理

Bedrock 还有一个额外复杂性：模型 ID 是 ARN（Amazon Resource Name），需要解析 region 和 account：

```typescript
// ARN format: arn:aws:bedrock:<region>:<account>:foundation-model/model-name
// 或
// arn:aws:bedrock:<region>:<account>:inference-profile/<profile-id>
```

---

## 三、模型别名系统

### 3.1 内置别名

Claude Code 支持一组模型别名，简化用户输入：

```typescript
switch (modelString) {
  case 'opusplan':
    return getDefaultSonnetModel() + (has1mTag ? '[1m]' : '')
  case 'sonnet':
    return getDefaultSonnetModel() + (has1mTag ? '[1m]' : '')
  case 'haiku':
    return getDefaultHaikuModel() + (has1mTag ? '[1m]' : '')
  case 'opus':
    return getDefaultOpusModel() + (has1mTag ? '[1m]' : '')
  case 'best':
    return getBestModel()
}
```

别名的好处：用户可以用 `opus` 而非 `claude-opus-4-6`，代码会自动解析为当前 Provider 的正确模型 ID。

### 3.2 自定义别名

用户可以设置 `ANTHROPIC_CUSTOM_MODEL_OPTION`，这是预验证的用户自定义模型：

```typescript
// validateModel.ts
if (normalizedModel === process.env.ANTHROPIC_CUSTOM_MODEL_OPTION) {
  return { valid: true }  // 跳过验证
}
```

---

## 四、环境变量覆盖机制

### 4.1 Tier 默认模型覆盖

Claude Code 允许通过环境变量覆盖每个 Tier 的默认模型：

```typescript
const TIERS = [
  {
    modelEnvVar: 'ANTHROPIC_DEFAULT_OPUS_MODEL',
    capabilitiesEnvVar: 'ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES',
  },
  {
    modelEnvVar: 'ANTHROPIC_DEFAULT_SONNET_MODEL',
    capabilitiesEnvVar: 'ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES',
  },
  {
    modelEnvVar: 'ANTHROPIC_DEFAULT_HAIKU_MODEL',
    capabilitiesEnvVar: 'ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES',
  },
]
```

### 4.2 能力覆盖（Capability Override）

这是理解"为什么有些功能只有 Anthropic 模型能用"的关键：

```typescript
export const get3PModelCapabilityOverride = memoize(
  (model: string, capability: ModelCapabilityOverride): boolean | undefined => {
    if (getAPIProvider() === 'firstParty') {
      return undefined  // First-party 用户不使用能力覆盖
    }
    // 检查当前模型是否匹配 ANTHROPIC_DEFAULT_OPUS_MODEL 等 env var
    // 如果匹配，返回该 env var 指定的能力标志
  }
)

type ModelCapabilityOverride =
  | 'effort'
  | 'max_effort'
  | 'thinking'
  | 'adaptive_thinking'
  | 'interleaved_thinking'
```

这个机制允许第三方 Provider 声明"我的 Sonnet 模型支持 extended thinking"，而不需要修改 Claude Code 的默认行为。

---

## 五、第三方 Provider 的能力限制

### 5.1 First-Party 独占功能

通过 `getAPIProvider() !== 'firstParty'` 检查，可以找到所有第三方不支持的功能：

```typescript
// 定价信息显示
export function getOpus46PricingSuffix(fastMode: boolean): string {
  if (getAPIProvider() !== 'firstParty') return ''  // 第三方不显示价格
}

// Opus 1M Merge
export function isOpus1mMergeEnabled(): boolean {
  if (getAPIProvider() !== 'firstParty') return false  // 第三方不支持 1M merge
}

// Computer Use 支持
export function modelSupportsComputerUse(model: string): boolean {
  if (getAPIProvider() !== 'firstParty') return false  // 只有 Anthropic 支持
}
```

### 5.2 独占功能清单

以下功能只在 firstParty 上可用：

| 功能 | 限制原因 | 源码位置 |
|------|---------|---------|
| Opus 1M Merge | 需要 Anthropic 特殊 API | 需查源码 |
| Computer Use | Anthropic 独占能力 | 需查源码 |
| 定价信息显示 | 第三方定价不在 Claude Code 控制内 | 需查源码 |
| Extended Thinking | 需要模型支持 extended_thinking 模式 | 需查源码 |
| Opus 4.0/4.1 → 4.6 Remap | 仅针对 firstParty 的 legacy 模型处理 | 需查源码 |

### 5.3 Provider 专属默认值

某些默认值在不同 Provider 上不同：

```typescript
export function getDefaultSonnetModel(): ModelName {
  if (process.env.ANTHROPIC_DEFAULT_SONNET_MODEL) {
    return process.env.ANTHROPIC_DEFAULT_SONNET_MODEL
  }
  // 第三方可能还没有最新的 Sonnet 4.6
  if (getAPIProvider() !== 'firstParty') {
    return getModelStrings().sonnet45  // 第三方默认用 4.5
  }
  return getModelStrings().sonnet46  // First-party 默认用 4.6
}
```

---

## 六、模型验证机制

### 6.1 为什么需要验证

当用户输入一个模型名时，系统需要验证：
1. 该模型在当前 Provider 上真实存在
2. 用户有权使用该模型（组织级别的 allowlist）
3. 该模型没有被 deprecation 策略禁用

```typescript
export async function validateModel(model: string): Promise<{ valid: boolean; error?: string }> {
  // 1. 检查 allowlist
  if (!isModelAllowed(normalizedModel)) {
    return { valid: false, error: `Model '${normalizedModel}' is not in the available models` }
  }

  // 2. 已知别名跳过验证
  if (isKnownAlias(normalizedModel)) return { valid: true }

  // 3. 真实 API 调用验证（sideQuery）
  try {
    await sideQuery({ model: normalizedModel, max_tokens: 1, maxRetries: 0, ... })
    return { valid: true }
  } catch (error) {
    return handleValidationError(error, normalizedModel)
  }
}
```

### 6.2 验证的代价

每次用户设置模型时，如果不在缓存中，都会触发一次 API 调用。这是**必要的代价**：
- 网络延迟（约 100-500ms）
- API 费用（即使 `max_tokens=1` 也会收费）

这意味着模型选择不是免费的，用户应该尽量使用别名而非完整模型名。

---

## 七、模型选择优先级

```typescript
export function getUserSpecifiedModelSetting(): ModelSetting | undefined {
  let specifiedModel: ModelSetting | undefined

  // 1. 运行时覆盖（/model 命令）
  const modelOverride = getMainLoopModelOverride()
  if (modelOverride !== undefined) {
    specifiedModel = modelOverride
  } else {
    // 2. --model 命令行标志
    // 3. ANTHROPIC_MODEL 环境变量
    // 4. 用户设置（settings.json）
    specifiedModel = process.env.ANTHROPIC_MODEL || settings.model || undefined
  }

  // 5. 如果指定了但不在 allowlist 中，无视
  if (specifiedModel && !isModelAllowed(specifiedModel)) {
    return undefined
  }

  return specifiedModel
}
```

---

## 八、架构图

```
                 ┌────────────────────────────────────────────┐
                 │     用户指定模型（/model、--model、env）       │
                 └──────────────────┬─────────────────────────┘
                                    │
                 ┌──────────────────▼─────────────────────────┐
                 │     getUserSpecifiedModelSetting()            │
                 │     1. 运行时 override                    │
                 │     2. --model 标志                        │
                 │     3. ANTHROPIC_MODEL env                │
                 │     4. settings.json                        │
                 └──────────────────┬─────────────────────────┘
                                    │
                 ┌──────────────────▼─────────────────────────┐
                 │     isModelAllowed() allowlist 检查          │
                 │     validateModel() API 验证（缓存）         │
                 └──────────────────┬─────────────────────────┘
                                    │
                 ┌──────────────────▼─────────────────────────┐
                 │     parseUserSpecifiedModel()                 │
                 │     1. 解析别名（opus → Sonnet 4.6）        │
                 │     2. 解析 [1m] 后缀                       │
                 │     3. 解析 opusplan 等特殊别名             │
                 └──────────────────┬─────────────────────────┘
                                    │
                 ┌──────────────────▼─────────────────────────┐
                 │     getModelStrings()                       │
                 │     根据 getAPIProvider() 返回正确格式的 ID   │
                 │     ┌─────────────────────────────────────┐ │
                 │     │ firstParty: 'claude-opus-4-6'      │ │
                 │     │ bedrock: 'us.anthropic.claude-...' │ │
                 │     │ vertex: 'claude-opus-4-6'           │ │
                 │     │ foundry: 'claude-opus-4-6'          │ │
                 │     └─────────────────────────────────────┘ │
                 └──────────────────┬─────────────────────────┘
                                    │
                 ┌──────────────────▼─────────────────────────┐
                 │     callModel() API 调用                    │
                 │     只传入当前 Provider 的正确 ID           │
                 └─────────────────────────────────────────────┘
```

---

## 九、设计哲学

### 9.1 为什么模型选择如此复杂？

因为 Claude Code 是一个**多后端统一客户端**：
- 用户可能从 Anthropic 官方切换到 Bedrock
- 同一套代码需要在两种环境下工作
- 模型 ID 在不同 Provider 上格式完全不同

这需要一个中心化的模型字符串解析层，而非散落在各处的 if-else。

### 9.2 为什么验证是必要的？

模型名不是简单的字符串——它背后是一个需要认证、需要授权、需要支付的真实 API 端点。允许用户输入任意字符串而不验证，会导致：
- 用户用了无权使用的模型 → API 返回 403
- 用户拼错了模型名 → API 返回 404
- 模型已被 deprecation → API 返回 404

---

## 总结

Claude Code 的多模型支持建立在三层抽象之上：

1. **Provider 抽象**：`getAPIProvider()` 通过环境变量决定使用哪个后端
2. **模型 ID 映射**：`ALL_MODEL_CONFIGS` 为每个模型保存四个 Provider 的 ID
3. **能力覆盖**：`get3PModelCapabilityOverride()` 允许第三方声明自己的能力标志

**独占功能的机制**不是代码硬编码的 if-else，而是通过 `getAPIProvider()` 的检查实现——这使得未来的 Provider 接入更容易。

理解这个架构，就能理解为什么 `CLAUDE_CODE_USE_BEDROCK=1` 可以让整个 CLI 无缝切换到 AWS Bedrock，而不需要修改任何业务代码。
