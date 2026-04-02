# KAIROS：持久助手与自动记忆整合

## 导言

KAIROS 是 Claude Code 中最复杂的隐藏功能——它让 Claude 从"一次性对话工具"变成"永不关机的 AI 助手"。这个名字来自希腊语 χρόνος（时间）的反义词，暗示它是一种"超时间"的持久存在。

KAIROS 不是单一功能，而是一套**持久化子系统**：跨会话运行、自动做梦整合记忆、主动模式（没人说话时自己找活干）、Cron 调度器。本文用通俗语言解释其核心概念。


## 一、KAIROS 是什么？

### 1.1 日常对话 vs KAIROS

**日常对话模式**：
```
用户: "帮我写一个函数"
Claude: [写函数]
用户: "测试一下"
Claude: [测试]
用户: [关掉终端]
→ 对话结束，Claude 消失
```

**KAIROS 模式**：
```
用户: "帮我写一个函数"
Claude: [写函数]
用户: [关掉终端]
→ Claude 继续在后台运行
→ 每天自动整理记忆
→ 晚上自动"做梦"整合知识
→ 用户回来时，Claude 记得所有上下文
```

### 1.2 激活条件

KAIROS 需要五层检查全部通过：

```
1. feature('KAIROS')           ← 编译时 feature flag
2. settings.assistant: true    ← settings.json 配置
3. 目录信任状态检查             ← 防止恶意仓库劫持
4. tengu_kairos (GrowthBook)   ← 远程开关
5. setKairosActive(true)        ← 全局状态
```

这意味着 KAIROS 不是默认开启的——Anthropic 通过多层开关控制其发布节奏。


## 二、跨会话持久运行

### 2.1 会话恢复

当用户重新打开终端时，KAIROS 需要恢复之前的会话状态：

```typescript
// src/utils/conversationRecovery.ts
// 识别 BriefTool 和 SendUserFileTool 的结果
// 判断 turn 是正常完成还是被中断
```

关键机制：**BriefTool**（KAIROS 通信频道）和 **SendUserFileTool** 被识别为"终端工具"，用于判断会话恢复点。

### 2.2 持久 Cron 任务

KAIROS 的后台任务定义在 `.claude/scheduled_tasks.json`：

```json
{
  "id": "morning-checkin",
  "recurring": true,
  "permanent": true,  // 不受 7 天过期限制
  "cron": "0 8 * * *"
}
```

三种特殊任务：
- `catch-up`：恢复中断的工作
- `morning-checkin`：每日早间签到
- `dream`：记忆整合


## 三、做梦机制（Dream）：自动记忆整合

### 3.1 为什么叫"做梦"？

KAIROS 每天晚上运行一个后台 agent，从分散的会话日志中提取知识，整理到结构化的记忆中。这个过程被称为"做梦"——因为它发生在用户不活跃时，且目的是整合和重构信息。

### 3.2 触发条件（三层门控）

```typescript
// src/services/autoDream/autoDream.ts
// 1. 时间门控：距上次整合超过 24 小时
// 2. 会话门控：至少 5 个新会话
// 3. 锁门控：没有其他进程正在整合
```

阈值通过 GrowthBook `tengu_onyx_plover` 动态配置——不需要更新代码就能调整触发频率。

### 3.3 四阶段整合流程

整合不是简单地把对话记录复制到记忆文件，而是一个精心设计的 pipeline：

| 阶段 | 动作 |
|------|------|
| **Orient** | 读取 MEMORY.md 索引，了解已有主题 |
| **Gather** | 从每日日志、已有记忆中搜集新信号 |
| **Consolidate** | 合并到主题文件，转换相对日期为绝对日期，删除过时事实 |
| **Prune** | 更新 MEMORY.md 索引，保持在 200 行限制内 |

### 3.4 整合锁

多实例环境下，需要防止多个 KAIROS 同时整合：

```typescript
// src/services/autoDream/consolidationLock.ts
// 使用 .consolidate-lock 文件
// 文件 mtime = lastConsolidatedAt
// 文件内容 = 持有者 PID
// 支持 PID 存活检查（1 小时超时）
```


## 四、主动模式（Proactive Mode）

### 4.1 什么是 Proactive Mode

没人说话时，Claude 自己找活干：

```markdown
# Proactive Mode

You are in proactive mode. Take initiative -- explore, act, and make progress
without waiting for instructions.

Start by briefly greeting the user.

You will receive periodic <tick> prompts. These are check-ins.
```

### 4.2 SleepTool 集成

当没活干时，Claude 调用 **SleepTool** 休息一段时间：

```typescript
// minSleepDurationMs / maxSleepDurationMs
// 控制 Sleep 持续时间范围
// 没活干就 Sleep，等下次 tick
```

### 4.3 状态管理

```typescript
// src/proactive/index.ts
active: boolean    // 是否激活
paused: boolean    // 用户按 Esc 暂停
contextBlocked: boolean  // API 错误时阻塞 tick，防止死循环
```


## 五、Cron 调度器

### 5.1 调度器架构

```typescript
// src/utils/cronScheduler.ts
CHECK_INTERVAL_MS = 1000  // 每秒 tick 一次
// 使用 chokidar 监视 .claude/scheduled_tasks.json
// 支持调度器锁，防止多实例重复触发
```

### 5.2 Jitter 防雷群机制

循环任务使用**确定性延迟**（基于 taskId 的 hash）避免多客户端同时触发：

```typescript
// src/utils/cronJitterConfig.ts
// 循环任务：interval 的 10%，上限 15 分钟
// 一次性任务：在 :00 和 :30 施加最多 90 秒提前量
```


## 六、与普通 /plan 模式的区别

| 维度 | /plan 模式 | KAIROS |
|------|-----------|--------|
| 生命周期 | 单次会话 | 持久运行 |
| 会话中断 | 丢失 | 自动恢复 |
| 记忆整理 | 手动 | 自动（做梦）|
| 主动行为 | 无 | 有（Proactive）|
| Cron 任务 | 无 | 有 |
| 开启条件 | 无需特殊配置 | 5 层检查 |


## 七、设计哲学

### 8.1 为什么需要五层激活检查？

KAIROS 涉及后台运行、自动执行命令等高风险能力。五层检查确保：
- Feature flag 控制编译
- Settings.json 控制用户意愿
- 目录信任检查防止恶意仓库
- GrowthBook 控制灰度发布
- 运行时状态是最后一道防线

### 8.2 为什么叫"做梦"？

"做梦"是一个巧妙的隐喻：
- 发生在休息时（用户不活跃）
- 目的是整合和重构（记忆整理）
- 过程对用户不可见（后台运行）
- 醒来后状态更好（有更新的记忆）


## 总结

KAIROS 是 Claude Code 的"超能力"——它让 AI 助手从"用完即走"变成"永不下线的搭档"。其核心机制：

1. **跨会话持久化**：通过会话恢复和持久 Cron 任务实现
2. **自动记忆整合**：每天"做梦"，将分散的日志整理为结构化记忆
3. **主动模式**：没活干时自己找活干，而不是干等着
4. **多层安全门控**：五层检查确保风险可控

理解 KAIROS，需要理解它不是一个功能，而是一套**持久化子系统**的协调机制。


