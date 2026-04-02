# MCP 扩展机制：Claude Code 的插件总线与 Computer Use 的 Anthropic 独占策略

## 导言

MCP（Model Context Protocol）是 Claude Code 的插件系统，允许用户接入任何符合 MCP 规范的外部工具和服务。但 Claude Code 的 MCP 实现远比"连接外部服务"更复杂——它有自己的 MCP shim（模拟包）、Computer Use 的独占策略、以及完整的 MCP 工具包装机制。本文解析 MCP 系统的全貌，并分析为什么 Computer Use 只支持 Anthropic 模型。

---

## 一、MCP 在 Claude Code 中的架构位置

### 1.1 MCP 是第三类工具

Claude Code 的工具池由三部分构成：

```
assembleToolPool()
    │
    ├── 内置工具（Built-in Tools）
    │   └── ~22 个核心工具（Bash, FileRead, FileEdit...）
    │
    ├── MCP 工具（通过 MCP 服务器动态接入）
    │   ├── 来自用户配置的 MCP servers
    │   └── 来自 @anthropic-ai/sdk 的内置 MCP
    │
    └── 工具搜索（ToolSearchTool）
        └── 负责延迟加载的二次发现
```

### 1.2 MCP 相关的源码分布

```
已删除
```

---

## 二、MCPTool：如何将 MCP Schema 转换为内置 Tool

### 2.1 适配器模式

`src/tools/MCPTool/MCPTool.ts` 使用了**适配器模式**，将 MCP 的外部工具适配为 Claude Code 内置的 Tool 接口：

```typescript
export const MCPTool = buildTool({
  isMcp: true,  // 标记为 MCP 工具
  name: 'mcp',  // 默认名，实际在 mcpClient.ts 中被覆盖

  // MCP 工具接受任意输入（因为各 MCP 工具 schema 不同）
  inputSchema: z.object({}).passthrough(),

  // 权限：passthrough 意味着由 MCP 服务器自己决定
  async checkPermissions(): Promise<PermissionResult> {
    return { behavior: 'passthrough', message: 'MCPTool requires permission.' }
  },

  // 结果渲染由 MCPTool/UI.ts 处理
  renderToolUseMessage,
  renderToolUseProgressMessage,
  renderToolResultMessage,
})
```

### 2.2 MCP 工具的命名规范

当 MCP 服务器暴露一个名为 `filesystem_readFile` 的工具时，Claude Code 会将其规范化为：

```typescript
// mcp__serverName__filesystem_readFile
// 或在 NO_PREFIX 模式下：filesystem_readFile
```

这个规范化在 `src/services/mcp/normalization.ts` 中处理：

```typescript
// 规范化函数
normalizeToolName(serverName: string, toolName: string): string {
  // 返回 mcp__${serverName}__${toolName} 格式
}
```

### 2.3 MCP 工具始终延迟加载

在 `isDeferredTool()` 函数中：

```typescript
// MCP 工具始终是延迟工具
if (tool.isMcp === true) return true
```

这是因为 MCP 工具来自外部服务器，Claude Code 不知道它们的 schema，也无法在 ToolSearch 之前将其安全地包含在初始 prompt 中。

---

## 三、MCP 服务器连接管理

### 3.1 支持的传输协议

MCP 服务器可以通过多种传输协议连接：

```typescript
const TransportSchema = z.enum([
  'stdio',   // 标准输入/输出（本地进程）
  'sse',     // Server-Sent Events
  'sse-ide', // IDE 的 SSE
  'http',    // HTTP
  'ws',      // WebSocket
  'sdk',     // SDK 控制传输（Claude Code 内部使用）
])
```

### 3.2 MCP 配置结构

MCP 服务器通过 JSON 配置文件或 CLI 参数添加：

```typescript
const McpStdioServerConfigSchema = z.object({
  command: z.string(),     // 启动命令，如 'npx'
  args: z.array(z.string()), // 参数，如 ['-y', '@anthropic/mcp-server-filesystem']
  env: z.record(z.string()), // 环境变量
})
```

### 3.3 OAuth 支持

MCP 系统支持 OAuth 认证（auth.ts），这允许用户连接需要认证的外部服务：

```typescript
const McpOAuthConfigSchema = z.object({
  clientId: z.string().optional(),
  callbackPort: z.number().optional(),
  authServerMetadataUrl: z.string().url().optional(),
  xaa: z.boolean().optional(),  // Cross-App Access
})
```

---

## 四、内置 MCP Shim：ant-computer-use 系列

### 4.1 什么是 Shim

有一组特殊的 shim 包（`shims/ant-computer-use-*`）：

```
shims/
├── ant-computer-use-mcp/     ← Computer Use MCP 主实现（stub）
├── ant-computer-use-input/   ← Computer Use 输入处理
└── ant-computer-use-swift/   ← Swift 版本
```

### 4.2 ant-computer-use-mcp 的 Stub 实现

`shims/ant-computer-use-mcp/index.ts` 是一个**完整的空实现**：

```typescript
// ant-computer-use-mcp/index.ts
export function buildComputerUseTools() {
  return []  // 返回空数组
}

export function createComputerUseMcpServer() {
  return {
    async connect() {},       // 空连接
    setRequestHandler() {},   // 空请求处理
    async close() {},         // 空关闭
  }
}

export function bindSessionContext() {
  return async () => ({
    is_error: true,  // 总是返回错误
    content: [{
      type: 'text',
      text: 'Computer use is unavailable in the restored development build.',
    }],
  })
}
```

### 4.3 Computer Use 的 Anthropic 独占策略

Computer Use（让模型控制计算机执行操作的能力）在 Claude Code 中被有意限制在 Anthropic 模型。这不是技术限制，而是**商业策略**。

**独占策略的原因**（推断）：

1. **API 差异化**：Computer Use 是 Anthropic API 的核心竞争力之一，通过独占保持 API 差异化
2. **风险控制**：Computer Use 涉及实际操作计算机（点击、输入），需要 Anthropic 对模型行为有完全控制
3. **生态锁定**：使用 Anthropic 官方 MCP 服务器的用户会自然地继续使用 Anthropic 的模型

---

## 五、为什么 MCP 工具始终延迟加载

### 5.1 技术原因

MCP 工具的 schema 由外部服务器动态提供，Claude Code 在启动时无法预知：
- 某个 MCP 服务器暴露了哪些工具
- 每个工具的具体参数 schema
- 工具的描述和能力

### 5.2 架构原因

MCP 工具和其他工具在 `assembleToolPool()` 中被统一管理：

```typescript
return uniqBy(
  [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
  'name',
)
```

这意味着 MCP 工具和内置工具**有同等的地位**，但行为不同：
- 内置工具可以选择 `alwaysLoad` 立即加载
- MCP 工具只能延迟加载（因为外部服务器不可信）

---

## 六、MCP 与内置工具的合并策略

### 6.1 去重规则

当内置工具和 MCP 工具同名时，内置工具优先：

```typescript
uniqBy(
  [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
  'name',  // 以 name 去重，保留第一个（built-in）
)
```

### 6.2 排序稳定性

所有工具按名字排序后再合并，这是为了**prompt cache 稳定性**：

```typescript
// 排序使工具顺序固定，不随 MCP 服务器连接顺序变化
// 如果工具顺序变化，所有下游的 cache key 都会失效
const byName = (a: Tool, b: Tool) => a.name.localeCompare(b.name)
return uniqBy([...builtInTools].sort(byName).concat(...), 'name')
```

---

## 七、架构图

```
                   ┌──────────────────────────────────────┐
                   │         MCP Servers (用户配置)           │
                   │  ┌────────────────────────────────┐  │
                   │  │ stdio: npx @anthropic/mcp-xxx  │  │
                   │  │ SSE: https://api.example.com/sse│  │
                   │  │ HTTP: https://api.example.com   │  │
                   │  └────────────────────────────────┘  │
                   └──────────────────┬───────────────────┘
                                      │ MCP 协议
                                      ▼
              ┌──────────────────────────────────────────────┐
              │          src/services/mcp/client.ts            │
              │  1. MCP 握手 + 能力协商                        │
              │  2. 工具 schema 获取                           │
              │  3. 工具调用（通过 transport）                  │
              │  4. OAuth 认证（如果需要）                     │
              └──────────────────────┬───────────────────────┘
                                     │
              ┌──────────────────────▼───────────────────────┐
              │         src/tools/MCPTool/MCPTool.ts          │
              │  Tool 接口适配                                  │
              │  - isMcp: true                               │
              │  - inputSchema: z.object({}).passthrough()   │
              │  - 权限: passthrough                          │
              │  - 延迟加载: isDeferredTool() = true           │
              └──────────────────────┬───────────────────────┘
                                     │
              ┌──────────────────────▼───────────────────────┐
              │        assembleToolPool() (tools.ts)         │
              │  built-in Tools + MCP Tools（去重，排序稳定）   │
              └──────────────────────┬───────────────────────┘
                                     │
              ┌──────────────────────▼───────────────────────┐
              │     isDeferredTool() (ToolSearchTool/prompt.ts)│
              │  MCP 工具始终返回 true（必须延迟加载）         │
              └──────────────────────┬───────────────────────┘
                                     │
              ┌──────────────────────▼───────────────────────┐
              │         ToolSearchTool (二次发现)              │
              │  模型必须先 discover 工具，才能调用             │
              └───────────────────────────────────────────────┘
```

---

## 八、Computer Use 独占的商业逻辑

Computer Use 在 `shims/ant-computer-use-mcp/` 中被完全 Stub 化。这揭示了 Claude Code 的扩展策略：

1. **核心能力独占**：Anthropic 的核心差异化功能（Computer Use、Bedrock、Vertex）通过独占保持竞争优势
2. **Shim 作为开发期占位符**：外部依赖被 shim 替代，以便代码能编译运行
3. **MCP 作为真正插件系统**：用户可以通过 MCP 接入任何第三方服务，但核心 AI 能力由 Anthropic 独占控制

---

## 总结

Claude Code 的 MCP 系统是一个**完整的插件架构**：

1. **适配器模式**：`MCPTool` 将任意 MCP 工具适配为统一的 Tool 接口
2. **传输协议多样性**：支持 stdio/SSE/HTTP/WebSocket 多种连接方式
3. **始终延迟加载**：MCP 工具必须通过 ToolSearch 二次发现，而非立即可用
4. **Computer Use 独占**：通过 shim stub 揭示了 Anthropic 的商业策略——核心 AI 能力不对第三方模型开放
5. **排序稳定性**：工具池排序确保 prompt cache 的稳定性

理解这个架构，就能理解 Claude Code 如何在保持核心能力独占的同时，通过 MCP 提供最大化的扩展性。
