# Claude Code 工程分类：一张图看懂目录的职责归属

## 导言

Claude Code 的 `src/` 目录下有 XX 个子目录，加上根目录的 XX 个核心文件。对于想深入理解这个项目的开发者来说，最大的障碍不是某个复杂的算法，而是**不知道代码在哪里**。

本文用工程视角对 `src/` 下的所有目录和核心文件做分类解析，帮你建立完整的"代码地图"。

---

## 一、目录全景图

```
src/
│
├── 【运行时核心】
│   ├── query/              ← query() 生成器的依赖注入（生产/测试 deps）
│   ├── Task.ts              ← Task 类型系统（TaskType, TaskState, generateTaskId）
│   ├── QueryEngine.ts       ← submitMessage() 主调度器
│   ├── query.ts             ← while(true) 主循环（~1800 行）
│   ├── context.ts           ← getSystemContext() / getUserContext() 上下文提供者
│   └── tools.ts             ← 工具注册表（getAllBaseTools / getTools / assembleToolPool）
│
├── 【命令系统】
│   └── commands/            ← 所有 /slash 命令（/commit, /model, /mcp, /diff...）
│       ├── add-dir/
│       ├── agents/
│       ├── branch/
│       ├── chrome/
│       ├── compact/
│       ├── config/
│       ├── context/
│       ├── doctor/
│       ├── exit/
│       ├── hooks/
│       ├── ide/
│       ├── mcp/
│       ├── model/          ← /model 命令（ModelPicker UI）
│       ├── plan/
│       ├── resume/
│       ├── skills/
│       ├── terminalSetup/
│       └── ...
│
├── 【工具实现】
│   └── tools/              ← 工具子目录
│       ├── AgentTool/      ← AgentTool（Subagent 入口）
│       ├── BashTool/       ← bash 命令执行
│       ├── FileReadTool/   ← 读文件
│       ├── FileEditTool/   ← 编辑文件
│       ├── FileWriteTool/  ← 写文件
│       ├── GlobTool/       ← glob 搜索
│       ├── GrepTool/      ← grep 搜索
│       ├── WebFetchTool/  ← Web 请求
│       ├── WebSearchTool/  ← 网络搜索
│       ├── MCPTool/       ← MCP 工具包装器
│       ├── TaskOutputTool/← Task 输出读取
│       ├── TaskStopTool/  ← Task 停止
│       ├── SkillTool/     ← Skill 调用
│       ├── ToolSearchTool/← 延迟工具发现
│       ├── EnterPlanModeTool/
│       ├── ExitPlanModeTool/
│       ├── EnterWorktreeTool/
│       ├── ExitWorktreeTool/
│       ├── BriefTool/     ← KAIROS 通信频道
│       └── ...
│
├── 【CLI 基础设施】
│   ├── cli/                ← 传输层、print、remoteIO、结构化输出
│   │   ├── handlers/       ← 命令处理器（agents, auth, autoMode, mcp, plugins）
│   │   └── transports/     ← 传输层（WebSocket, SSE, HybridTransport）
│   ├── ink.ts              ← CLI ink (React) 主渲染入口
│   └── cli/print.ts        ← CLI 输出格式化
│
├── 【Bridge 系统】（与服务端的通信）
│   └── bridge/             ← bridgeApi, replBridge, peerSessions, trustedDevice...
│       ├── bridgeMain.ts    ← Bridge 主入口
│       ├── replBridge.ts   ← REPL Bridge 实现
│       ├── sessionRunner.ts← Session 运行器
│       ├── jwtUtils.ts    ← JWT 工具
│       └── ...
│
├── 【状态管理】
│   ├── state/              ← AppState 定义（整个应用的全局状态）
│   └── hooks/              ← React hooks（useCanTool, useMergedTools...）
│
├── 【MCP 扩展】
│   ├── services/mcp/       ← MCP 服务器连接管理
│   ├── tools/MCPTool/      ← MCP 工具包装为内置 Tool
│   └── tools/ListMcpResourcesTool/
│   └── tools/ReadMcpResourceTool/
│
├── 【记忆与上下文】
│   ├── memdir/             ← MEMORY.md 系统、CLAUDE.md 加载
│   │   ├── memdir.ts      ← loadMemoryPrompt() / buildMemoryPrompt()
│   │   └── paths.ts       ← 内存目录路径解析
│   ├── context.ts          ← 系统上下文（git status, cache breaker）
│   └── services/contextCollapse/  ← 实验性上下文折叠（当前 stub）
│
├── 【Compact 压缩】
│   └── services/compact/   ← AutoCompact / SnipCompact / ReactiveCompact
│       ├── compact.ts      ← compactConversation() 主逻辑
│       ├── autoCompact.ts  ← 触发阈值（contextWindow - 13000）
│       ├── snipCompact.ts  ← HISTORY_SNIP 实验
│       └── reactiveCompact.ts ← 响应式压缩
│
├── 【原生绑定】
│   └── native-ts/          ← 原生模块 TypeScript wrappers
│       ├── color-diff/     ← 颜色差分计算
│       ├── file-index/     ← 文件索引
│       └── yoga-layout/    ← Yoga 布局引擎
│
├── 【Agent 与 Assistant】
│   ├── assistant/          ← Session 发现、历史管理
│   ├── tasks/              ← RemoteAgentTask（React 组件）
│   └── proactive/          ← Proactive 特性（KAIROS 相关）
│
├── 【Buddy 助手 UI】
│   └── buddy/              ← Companion Sprite 动画、companion 通知
│       ├── companion.tsx
│       └── sprites.tsx
│
├── 【React UI 组件】
│   ├── components/          ← CLI 中的 React 组件
│   │   ├── PromptInput/   ← 命令行输入组件
│   │   ├── ModelPicker.tsx← 模型选择器
│   │   ├── Spinner/
│   │   ├── CustomSelect/
│   │   └── wizard/
│   ├── screens/            ← 屏幕级组件（REPL.tsx）
│   └── components/mcp/    ← MCP 相关 UI
│
├── 【后台服务】
│   ├── services/           ← Analytics、SettingsSync、OAuth、token 估算
│   │   ├── analytics/     ← GrowthBook、DataDog
│   │   ├── api/          ← Claude API 封装、错误处理
│   │   ├── settingsSync/ ← 设置同步
│   │   ├── contextCollapse/← 实验性压缩（stub）
│   │   └── oauth/        ← OAuth 服务
│   └── jobs/              ← 后台任务（classifier 等）
│
├── 【插件系统】
│   └── plugins/            ← 插件加载、管理
│       └── bundled/       ← 内置插件
│
├── 【Skills】
│   └── skills/             ← 内置 Skill 定义
│       └── bundled/
│
├── 【Entry Points】
│   └── entrypoints/        ← SDK 类型定义、agent SDK 入口
│
├── 【Coordinator 模式】
│   └── coordinator/        ← COORDINATOR_MODE 实验功能
│
├── 【服务器】
│   └── server/             ← 服务器相关（SSH、上游代理）
│       ├── ssh/
│       └── upstreamproxy/
│
├── 【其他支撑目录】
│   ├── constants/          ← 常量（xml.ts 标签、figures 等）
│   ├── types/             ← TypeScript 类型（message.ts、permissions.ts...）
│   ├── utils/             ← 工具函数集（tokens、env、debug...）
│   ├── schemas/           ← JSON Schema 定义
│   ├── bootstrap/          ← 启动状态（getMainLoopModelOverride、getFastModeState）
│   ├── hooks/             ← React hooks（权限、工具使用）
│   └── migrations/        ← 数据库迁移
│
└── 【实验/可选目录】
    ├── vim/               ← Vim 模式
    ├── voice/             ← 语音模式
    ├── moreright/         ← 内部功能
    └── coordinator/       ← 多 Agent 协调模式
```

---

## 二、核心文件职责速查表

### 运行入口

| 文件 | 职责 |
|------|------|
| `src/dev-entry.ts` | `npm run dev` 入口，Bun 启动脚本 |
| `src/ink.ts` | CLI ink 主渲染入口（React in terminal）|
| `src/query.ts` | `while(true)` 主循环，最核心 |
| `src/QueryEngine.ts` | `submitMessage()` 调度器，yield SDKMessage |

### 数据模型

| 文件 | 职责 |
|------|------|
| `src/Task.ts` | Task 类型系统（TaskType, TaskState, generateTaskId）|
| `src/types/message.ts` | 所有消息类型（Assistant/User/System/Progress...）|
| `src/types/permissions.ts` | PermissionMode、PermissionResult |
| `src/types/ids.ts` | AgentId、SessionId 等 ID 类型 |

### 工具系统

| 文件 | 职责 |
|------|------|
| `src/Tool.ts` | Tool 接口定义 |
| `src/tools.ts` | 工具注册表（getAllBaseTools / getTools / assembleToolPool）|
| `src/services/tools/toolExecution.ts` | `runToolUse()` 权限检查和执行 |
| `src/services/tools/toolOrchestration.ts` | `runTools()` 编排逻辑 |
| `src/services/tools/StreamingToolExecutor.ts` | 并发工具流式执行 |

### 上下文与记忆

| 文件 | 职责 |
|------|------|
| `src/context.ts` | `getSystemContext()` / `getUserContext()` 上下文提供者 |
| `src/memdir/memdir.ts` | MEMORY.md 内存目录系统 |
| `src/memdir/paths.ts` | 内存目录路径解析（`~/.claude/projects/...`）|
| `src/utils/claudemd.ts` | CLAUDE.md 加载器（5 级优先级）|

### API 层

| 文件 | 职责 |
|------|------|
| `src/services/api/claude.ts` | API 调用封装 |
| `src/services/api/errors.ts` | API 错误分类（NotFoundError、RateLimitError...）|
| `src/bridge/bridgeMain.ts` | Bridge 主入口 |
| `src/bridge/replBridge.ts` | REPL Bridge 实现 |
| `src/cli/transports/WebSocketTransport.ts` | WebSocket 传输层 |

### 权限

| 文件 | 职责 |
|------|------|
| `src/hooks/useCanTool.tsx` | `canUseTool()` 权限决策（allow/deny/ask）|
| `src/types/permissions.ts` | PermissionMode 四种模式 |
| `src/utils/permissions/` | 详细权限规则实现 |

### Compact

| 文件 | 职责 |
|------|------|
| `src/services/compact/compact.ts` | `compactConversation()` 全量压缩 |
| `src/services/compact/autoCompact.ts` | 自动压缩触发器（阈值：context - 13000）|
| `src/services/compact/snipCompact.ts` | HISTORY_SNIP 实验（消息截断）|
| `src/services/compact/reactiveCompact.ts` | 响应式压缩（413 时触发）|

---

## 三、非运行代码辨别

### Shims（本地依赖垫片）

```
shims/
├── color-diff-napi/   ← 颜色差分模块（重定向到 src/native-ts/color-diff/）
├── modifiers-napi/    ← 修饰符模块
├── url-handler-napi/  ← URL 处理模块
├── ant-computer-use-mcp/   ← Computer Use MCP（stub，全部返回空实现）
├── ant-computer-use-input/  ← Computer Use 输入
├── ant-computer-use-swift/  ← Swift 版本
└── ant-claude-for-chrome-mcp/ ← Chrome 扩展 MCP
```

### 配置文件

```
package.json        ← 依赖声明
tsconfig.json      ← TypeScript 配置
bun.lock           ← Bun 锁文件
.gitignore         ← Git 忽略
```

---

## 四、设计分层总结

Claude Code 在架构上可以分为五层：

```
┌─────────────────────────────────────────┐
│  Layer 5: 用户界面层                     │
│  components/, screens/REPL.tsx, buddy/  │
├─────────────────────────────────────────┤
│  Layer 4: 命令与交互层                    │
│  commands/ (87 commands), cli/          │
├─────────────────────────────────────────┤
│  Layer 3: 工具与 MCP 扩展层              │
│  tools/ (53 tools), services/mcp/       │
├─────────────────────────────────────────┤
│  Layer 2: Agent 核心层                   │
│  query.ts, QueryEngine.ts, Task.ts,     │
│  context.ts, tools.ts                    │
├─────────────────────────────────────────┤
│  Layer 1: 基础设施层                      │
│  state/, hooks/, utils/, services/,     │
│  bridge/, types/, constants/            │
└─────────────────────────────────────────┘
```

**理解层次的方法**：
- 如果你想理解**一次对话的完整生命周期**：从 Layer 2 入手（query.ts）
- 如果你想理解**某个具体能力**（如文件编辑、Web 搜索）：从 Layer 3 对应目录入手
- 如果你想理解**如何扩展 Claude Code**（添加命令、工具）：从 Layer 3/4 对应目录入手
- 如果你想理解**底层的通用能力**（权限、状态管理、传输）：从 Layer 1 入手

---

## 总结

Claude Code 遵循了**强关注点分离**的原则：

1. **命令系统（commands/）** 是用户入口，与核心逻辑完全分离
2. **工具系统（tools/）** 是能力扩展口，通过统一的 Tool 接口注册
3. **Query 引擎（query.ts）** 是调度核心，不包含任何 UI 逻辑
4. **Context/Memory 系统** 独立于 query 引擎，通过 `userContext` / `systemContext` 注入
5. **Compact 压缩** 完全是 query 引擎的附加策略，不修改循环本身

这个结构使得 Claude Code 可以**CLI、SDK、REPL 三种模式共用同一套核心**，而只需要替换最上层的 UI/transport 层。
