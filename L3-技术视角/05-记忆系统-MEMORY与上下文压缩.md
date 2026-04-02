# 记忆系统设计：多级加载、Typed Memory 与上下文压缩

## 导言

Claude Code 的记忆系统不是简单的 KV 存储，也不是单一文件注入。它由三个紧密配合的子系统构成：**CLAUDE.md 多级加载体系**、**AutoMemory 持久化记忆**、**AutoCompact 上下文压缩**。三者共同解决了大模型应用中的核心矛盾——如何在有限的上下文窗口中，持久化最重要的信息，同时管理不断膨胀的对话历史。

---

## 一、CLAUDE.md 体系：5 级加载链

### 1.1 加载优先级（从低到高）

Claude Code 的 CLAUDE.md 文档加载顺序有明确注释：

```
加载顺序（后加载 = 高优先级 = 模型更关注）
┌─────────────────────────────────────────────────────┐
│ Level 1: Managed Memory (最低优先级)                  │
│   /etc/claude-code/CLAUDE.md                        │
│   → 全局策略，所有用户共享                          │
├─────────────────────────────────────────────────────┤
│ Level 2: User Memory                                │
│   ~/.claude/CLAUDE.md                               │
│   → 用户私有，对所有项目生效                        │
├─────────────────────────────────────────────────────┤
│ Level 3: Project Memory                             │
│   ./CLAUDE.md, ./.claude/CLAUDE.md,                 │
│   ./.claude/rules/*.md (从 CWD 向上遍历到根目录)    │
│   → 纳入版本控制的团队共享规则                       │
├─────────────────────────────────────────────────────┤
│ Level 4: Local Memory                               │
│   ./CLAUDE.local.md                                │
│   → 本地私有（不提交到 git）                        │
├─────────────────────────────────────────────────────┤
│ Level 5: AutoMem (最高优先级)                       │
│   <project>/memory/MEMORY.md                        │
│   → 自动记忆系统，由 Agent 自动维护                  │
└─────────────────────────────────────────────────────┘
```

### 1.2 核心加载函数

```typescript
export const getMemoryFiles = memoize(
  // 实现：getClaudeMds() 内部调用此函数
  // 返回所有 CLAUDE.md 文件的路径 + 内容
)

export const getClaudeMds = (
  memoryFiles: MemoryFile[],  // 从 getMemoryFiles() 获取
): string => {
  // 按优先级从低到高排列，最后一个覆盖前面的同名规则
}
```

### 1.3 `@include` 指令支持

CLAUDE.md 系统支持 `@include` 指令，可以在文件中引用其他文件：

```
语法：
  @path           → 相对路径（相对于当前文件）
  @./relative     → 同上
  @~/home/path    → 用户 home 目录
  @/absolute/path → 绝对路径

限制：
  - 仅在文本节点展开，不在代码块内
  - 最多嵌套 5 层（MAX_INCLUDE_DEPTH = 5）
  - 循环引用自动检测并跳过
  - 文件不存在时静默忽略
```

### 1.4 文件发现算法

Project Memory 的文件发现通过**从 CWD 向上遍历到根目录**实现：

```typescript
export async function getMemoryFilesForNestedDirectory(
  cwd: string,
): Promise<MemoryFile[]> {
  // 1. 从 CWD 开始向上遍历
  // 2. 每层检查：CLAUDE.md, .claude/CLAUDE.md, .claude/rules/*.md
  // 3. 越接近 CWD 的文件优先级越高
  // 4. 调用 getMemoryFiles() 获取实际内容
}
```

这意味着：**同一规则在项目根目录的 CLAUDE.md 和子目录的 CLAUDE.md 中冲突时，子目录的版本优先**，因为它后被加载。

---

## 二、AutoMemory：Agent 自动维护的持久记忆

### 2.1 目录结构

```
~/.claude/projects/<slug>/memory/
  MEMORY.md      ← 索引文件（最多 200 行，25KB）
  user_role.md   ← 用户角色记忆
  feedback_*.md   ← 用户反馈记忆
  project_*.md   ← 项目上下文记忆
  reference_*.md ← 参考信息记忆
```

### 2.2 四型分类体系

AutoMemory 要求所有记忆属于四个类型之一：

```typescript
// 记忆四型分类
type MemoryType = 'user' | 'feedback' | 'project' | 'reference'

// 各类型含义：
// - user: 用户角色、偏好、工作方式
// - feedback: 用户给出的修正、偏好、不喜欢重复的内容
// - project: 项目背景、架构决策、上下文
// - reference: 外部参考信息（文档、链接等）
```

### 2.3 MEMORY.md 索引限制

`MEMORY.md` 是索引文件，不是存储文件。系统对其有严格的截断限制：

```typescript
export const MAX_ENTRYPOINT_LINES = 200   // 最多 200 行
export const MAX_ENTRYPOINT_BYTES = 25_000 // 最多 25KB
```

截断时会附加警告：

```markdown
> WARNING: MEMORY.md is 247 lines (limit: 200). Only part of it was loaded.
> Keep index entries to one line under ~200 chars; move detail into topic files.
```

**设计意图**：强制用户保持索引简洁，把详细内容放到独立文件中。这避免了长索引文件吞噬上下文窗口的问题。

### 2.4 如何写入记忆

Agent 被引导用两步写入记忆：

```markdown
**Step 1** — write the memory to its own file (e.g., `user_role.md`)
**Step 2** — add a pointer to that file in `MEMORY.md`
```

格式要求：

```markdown
---
name: {{memory name}}
description: {{one-line description — used to decide relevance in future conversations}}
type: {{user, feedback, project, reference}}
---

{{memory content}}
```

---

## 三、AutoCompact：上下文压缩机制

### 3.1 触发时机

AutoCompact 不是"满了才压"，而是**预留缓冲区的主动触发**：

```typescript
export const AUTOCOMPACT_BUFFER_TOKENS = 13_000  // 缓冲区大小
export const WARNING_THRESHOLD_BUFFER_TOKENS = 20_000
export const MANUAL_COMPACT_BUFFER_TOKENS = 3_000

export function getAutoCompactThreshold(model: string): number {
  const effectiveContextWindow = getEffectiveContextWindowSize(model)
  return effectiveContextWindow - AUTOCOMPACT_BUFFER_TOKENS
}
```

以 Sonnet 4.6（200K context）为例：

```
effectiveContextWindow = 200,000 - 20,000 (摘要输出预留) = 180,000
autoCompactThreshold   = 180,000 - 13,000 = 167,000 tokens
```

**在 token 达到 167K 时就触发压缩**，而不是等到接近 200K 满。这 13,000 token 的缓冲区是为了给压缩操作本身留出空间。

### 3.2 压缩熔断机制

连续 3 次压缩失败后，AutoCompact 进入熔断状态：

```typescript
const MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES = 3

// 3 次连续失败 → 停止重试
// 这防止了 API 已满（413）时无限重试浪费 API 调用
```

### 3.3 压缩保留策略

Claude Code 的压缩是**选择性保留**，不是全量摘要：

```typescript
// 压缩后保留的内容：
// 1. 最近 5 个文件（每个最多 5,000 tokens）
const POST_COMPACT_MAX_TOKENS_PER_FILE = 5_000
// 2. Skills 相关内容（预算 25,000 tokens）
const POST_COMPACT_SKILLS_TOKEN_BUDGET = 25_000
// 3. Plan attachments（计划模式相关）
// 4. SystemCompactBoundaryMessage + 摘要 UserMessage
// 5. 压缩后的工具/MCP/Agent 公告（deferred announcements）
```

**设计意图**：压缩不是简单地把历史丢掉，而是"保留最重要的上下文，换一种更紧凑的形式"。最近的文件、Skills、计划状态是决策的核心依据，不能丢失。

### 3.4 两种压缩方向

Claude Code 支持两种压缩方向：

| 方向 | 含义 | 保留 |
|------|------|------|
| `up_to` | 从指定消息向上压缩 | 前缀（suffix-preserving cache）|
| `from` | 从指定消息向下压缩 | 后缀（prefix-preserving cache）|

这使得压缩可以针对**对话中间**的某个历史区间进行，而不是只能压缩最早的消息。

---

## 四、Context Collapse：实验性未来方案

### 4.1 当前状态：完整 stub

contextCollapse 中的所有函数目前都是 stub：

```typescript
export function isContextCollapseEnabled(): boolean {
  return false  // 实验性功能，默认关闭
}

export async function applyCollapsesIfNeeded<T>(
  messages: T,
): Promise<{ messages: T; changed: boolean }> {
  return { messages, changed: false }  // 不做任何事
}
```

### 4.2 设计思路

Context Collapse 是相对于 AutoCompact 的另一种思路：

- **AutoCompact**：压缩时生成摘要，丢失细粒度历史
- **Context Collapse**：保留所有历史，通过某种**折叠机制**让模型只看需要的一部分

当 `feature('CONTEXT_COLLAPSE')` 启用时，它会**阻止 AutoCompact 触发**，作为两种互斥策略之一。

---

## 五、三系统的协同关系

```
                    ┌────────────────────────────────────────────┐
                    │              用户对话输入                    │
                    └──────────────────┬───────────────────────┘
                                       │
                    ┌──────────────────▼───────────────────────┐
                    │         CLAUDE.md 加载 (5 级)            │
                    │  注入优先级：managed < user < project     │
                    │            < local < automem              │
                    └──────────────────┬───────────────────────┘
                                       │
                    ┌──────────────────▼───────────────────────┐
                    │     getSystemPrompt() + getUserContext()  │
                    │         构建 system prompt                  │
                    └──────────────────┬───────────────────────┘
                                       │
              ┌─────────────────────────┼─────────────────────────┐
              │                         │                         │
              ▼                         ▼                         ▼
    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
    │   AutoMemory     │    │    AutoCompact   │    │  query() 循环   │
    │  (持续积累)      │    │   (按需触发)      │    │   (主对话)       │
    │                 │    │                  │    │                 │
    │ memory/*.md     │    │ threshold 达到   │    │ tool_use →     │
    │ + MEMORY.md     │    │ → compactConv()  │    │ results →      │
    └──────────────────┘    └──────────────────┘    │ needsFollowUp? │
                                                    └───────┬────────┘
                                                            │
                                          ┌─────────────────┘
                                          │ 无 needsFollowUp
                                          ▼
                                 ┌─────────────────────┐
                                 │   compact if needed │
                                 │  (167K threshold)   │
                                 └─────────────────────┘
```

---

## 六、记忆的加载时机与缓存

### 6.1 SystemContext vs UserContext

```typescript
// context.ts
export function getSystemContext(): string {
  // 每次对话只调用一次（缓存）
  // 内容：git status + cache breaker
}

export function getUserContext(): string {
  // 每次对话只调用一次（缓存）
  // 内容：CLAUDE.md 内容（通过 getClaudeMds()）
}
```

两者都是**缓存在整个对话生命周期内的**，这避免了每轮都重复扫描文件系统。

### 6.2 Nested Memory 机制

`loadedNestedMemoryPaths` 用于跟踪已加载的项目子目录内存文件，防止重复加载同一个记忆：

```typescript
loadedNestedMemoryPaths = new Set<string>()

// 当进入新的子目录时，自动加载该子目录下的 CLAUDE.md
// 并将路径加入 loadedNestedMemoryPaths
```

---

## 七、设计哲学总结

### 7.1 为什么多级而非单级？

单一 CLAUDE.md 的问题：**无法同时支持全局规则和个人偏好**。5 级体系允许：
- 公司管理员在 `/etc/claude-code/` 设置全公司规范
- 用户在 `~/.claude/` 设置个人偏好
- 项目在 `.claude/rules/` 设置团队规范
- 个人在 `CLAUDE.local.md` 设置临时本地规则
- Agent 在 `memory/MEMORY.md` 自动维护项目上下文

### 7.2 为什么压缩是选择性保留？

全量摘要的问题：**丢失的信息无法恢复**，且对于代码搜索等任务，历史文件的精确内容比摘要更有价值。选择性保留策略使得压缩后的上下文仍然包含具体的代码引用，而不是抽象的描述。

### 7.3 为什么不自动记住所有对话？

AutoMemory 坚持**两步写入**（文件 + 索引）而非自动记录所有对话。这是有意的设计选择：
- **可审计性**：用户可以查看 `memory/` 目录，了解 Agent 认为什么是重要的
- **可控性**：用户可以删除错误的记忆
- **可组合性**：Agent 在做决策时可以查看完整的记忆历史

---

## 总结

Claude Code 的记忆系统是一个**分层、多机制协作**的系统：

1. **CLAUDE.md 5 级体系**解决了"谁的规定优先"的问题，支持从全局到个人的渐进式定制
2. **AutoMemory** 通过强制索引 + 独立文件的结构，确保记忆可审计、可维护
3. **AutoCompact** 是上下文管理的最后防线，通过 13K 缓冲区和选择性保留策略，在满之前主动压缩而非被动失败
4. **ContextCollapse** 是未来方向，用折叠代替压缩，但目前仍是实验性 stub

理解这三个子系统的协作关系，才能理解为什么 Claude Code 可以在长对话中保持稳定的行为，而不是被无限增长的历史上下文压垮。
