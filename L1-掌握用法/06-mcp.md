# Claude Code MCP 完全教学：模型上下文协议，让 AI 连接真实世界
## 一、什么是 MCP？
**MCP（Model Context Protocol，模型上下文协议）** 是 Anthropic 推出的**开放、标准化的 AI 外部连接协议**，是 Claude Code 突破“纯文本对话”、安全接入现实世界的核心桥梁。

### 1. 通俗理解
- **AI 的通用 USB 接口**：像 USB 统一连接鼠标、硬盘、打印机一样，MCP 让 Claude Code **即插即用**连接数百种外部工具与数据。
- **AI 的第二大脑**：打破训练数据截止限制，实时获取最新信息、操作真实系统。
- **安全可控的能力扩展**：不是无限制“越狱”，而是**权限最小、可审计、标准化**的外部能力接入。

### 2. 核心角色（Client/Server）
- **MCP Client（客户端）**：Claude Code（发起请求、调用工具）
- **MCP Server（服务端）**：外部服务（GitHub、数据库、Figma、文件系统等，暴露能力）

### 3. MCP 提供三大核心能力
- **Tools（工具）**：AI 可执行的操作（创建 PR、查询 SQL、修改文件）
- **Resources（资源）**：可读取的数据（文件、API 响应、数据库表）
- **Prompts（提示模板）**：标准化工作流（代码审查、文档生成）

### 4. 与 Skill/Hooks 对比
| 特性 | MCP | Skill | Hooks |
|:--- |:--- |:--- |:--- |
| **定位** | **外部连接层**（跨系统） | **内部工作流**（单项目） | **自动化规则**（事件触发） |
| **范围** | 本地/远程服务（GitHub/DB） | 项目/全局提示与流程 | 本地脚本与安全控制 |
| **触发** | AI 自动感知、自然语言调用 | 手动 `/命令` / 自动匹配 | 事件（PreTool/PostTool） |
| **权限** | 细粒度（读/写/执行） | 工具白名单 | 允许/阻止/修改 |

---

## 二、支持版本、模型与权限
### 1. 版本要求
- Claude Code **v1.9+**（2025 年后稳定版）
- **所有订阅**：免费版 / Pro / Max 均支持（无功能限制）

### 2. 模型兼容性（关键）
- **全模型支持**：Claude 3（Opus/Sonnet/Haiku）、Claude 2、**国产模型（通义/GLM/DeepSeek）**
- MCP 是**客户端本地协议**，与后端模型无关，**切换国产模型完全可用**

### 3. 作用域（三层）
- **User（全局）**：`~/.claude/mcp.json`（所有项目生效）
- **Project（项目）**：`.claude/mcp.json`（团队共享，Git 提交）
- **Local（本地）**：`.claude/mcp.local.json`（个人私有，gitignore）

---

## 三、核心架构：两种传输模式
### 1. Stdio（本地进程，默认）
- **原理**：启动本地 Node/Python 进程，通过标准输入输出通信
- **优点**：简单、低延迟、无需网络、安全
- **缺点**：仅限本地、每个会话独立
- **适用**：文件系统、本地 Git、本地脚本

### 2. HTTP（远程服务）
- **原理**：连接远程 HTTP/SSE 服务
- **优点**：跨设备、可共享、支持 OAuth/API Key 认证
- **缺点**：需网络、配置稍复杂
- **适用**：GitHub、Figma、Sentry、云数据库

---

## 四、安装与管理：3 种方式（新手首选）
### 方式 1：CLI 命令（最快，推荐）
#### 1. 基础命令
```bash
# 查看 MCP 状态（必用）
/mcp              # 交互界面列出所有服务
claude mcp list   # 命令行查看

# 添加 MCP 服务（核心）
# 语法：claude mcp add [选项] <名称> <命令/URL>
claude mcp add <名称> -- npx -y @modelcontextprotocol/server-xxx  # Stdio
claude mcp add --transport http <名称> https://xxx.com/mcp         # HTTP

# 管理
claude mcp remove <名称>  # 卸载
claude mcp enable <名称>   # 启用
claude mcp disable <名称>  # 禁用
```

#### 2. 常用选项
- `--transport stdio|http`：传输模式
- `--env KEY=VALUE`：环境变量（API Key、Token）
- `--header "Key: Value"`：HTTP 请求头
- `--scope user|project|local`：作用域

### 方式 2：手动配置文件（进阶）
#### 1. 配置路径
```bash
# 全局
~/.claude/mcp.json

# 项目
.claude/mcp.json
```

#### 2. 标准 JSON 结构
```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxx"
      }
    },
    "postgres": {
      "type": "http",
      "url": "https://mcp.db.com/postgres",
      "headers": {
        "Authorization": "Bearer xxx"
      }
    }
  }
}
```

#### 3. 生效
**必须重启 Claude Code** 才能加载新配置

### 方式 3：交互向导（新手）
```bash
claude mcp  # 启动配置向导，按提示操作
```

---

## 五、实战：8 个必备 MCP（直接复制）
### 1. 文件系统（本地读写）
```bash
claude mcp add fs -- npx -y @modelcontextprotocol/server-filesystem ~/dev
```
- 功能：读取、写入、搜索、监控项目文件
- 用法：`帮我读取 src/index.js`、`创建 utils/date.js`

### 2. GitHub（代码协作）
```bash
# 安装
claude mcp add github -- npx -y @modelcontextprotocol/server-github
# 认证（交互）
/mcp → github → Authenticate
```
- 用法：
  - `创建 PR，合并 feature/login 到 main`
  - `审查 PR #123`
  - `列出分配给我的 Issue`

### 3. PostgreSQL（数据库）
```bash
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
# 环境变量（可选）
export DATABASE_URL=postgresql://user:pass@localhost:5432/db
```
- 用法：`查询 users 表前 10 条`、`创建 orders 表`

### 4. Figma（设计转代码）
```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```
- 用法：`从 Figma 文件 https://figma.com/file/xxx 生成 React 代码`

### 5. Sentry（错误监控）
```bash
claude mcp add sentry -- npx -y @modelcontextprotocol/server-sentry
```
- 用法：`分析 Sentry 最近 24 小时的生产错误`

### 6. Context7（最新文档）
- 解决：AI 训练数据过时（如 Next.js 15、React 19 新特性）
```bash
# 远程（推荐）
claude mcp add --transport http context7 https://mcp.context7.com/mcp
```
- 用法：`用 Context7 查 Next.js 15 Server Actions 用法`

### 7. Chrome DevTools（网页自动化）
```bash
claude mcp add chrome -- npx -y chrome-devtools-mcp@latest
```
- 用法：`抓取 https://example.com 页面数据`、`测试页面性能`

### 8. JetBrains IDE（联动开发工具）
```bash
claude mcp add jetbrains -- npx -y @jetbrains/mcp-proxy
```
- 用法：`在 WebStorm 中打开当前文件`、`同步代码到 IDE`

---

## 六、使用 MCP：3 种方式（全自动）
### 1. 自然语言调用（最省心）
**无需命令**，直接描述需求，Claude **自动感知 MCP 能力**：
```bash
# 文件系统
帮我读取 src/app.js，分析路由逻辑

# GitHub
创建一个 PR，标题：Fix login bug，合并到 dev 分支

# 数据库
查询 users 表中最近注册的 5 个用户

# Figma
把这个 Figma 设计稿转成 React + Tailwind 代码
```

### 2. URI 精准引用（高级）
```bash
# 格式：@<mcp名称>:<类型>://<ID>
分析 @github:issue://123 并给出修复方案
对比 @postgres:schema://users 与文档
查看 @figma:file://xxx 的组件结构
```

### 3. 斜杠命令（快捷）
```bash
# 列出所有 MCP
/mcp

# 直接调用工具
/github create-pr --title "Fix" --base main --head feature
/postgres query "SELECT * FROM users LIMIT 5"
```

---

## 七、运行表现（你会看到什么）
### 1. 连接状态
- 输入 `/mcp` → 显示 **Connected**（已连接）、**Disconnected**（未连接）
- Claude Code 底部显示 **MCP: 8 个服务已连接**

### 2. 执行过程
- AI 识别需求 → **自动调用 MCP 工具** → 显示 `[MCP] github.createPR`
- 实时输出执行日志：`[MCP] 创建 PR #456 成功`
- 结果自动注入上下文，AI 基于真实数据继续对话

### 3. 权限控制
- 超出权限操作（如写只读数据库）→ **明确阻止**：`[MCP] 权限不足：只读`
- 所有操作可审计：`.claude/logs/mcp-*.log`

---

## 八、最佳实践与避坑
### 1. 最佳实践
- **最小权限**：仅开放必要目录/表/权限（如不开放 `/` 根目录）
- **作用域隔离**：团队共享用 `project`，个人用 `local`
- **安全认证**：远程服务用 API Key/OAuth，**不硬编码密钥**
- **版本管理**：项目 `mcp.json` 提交 Git，统一环境
- **定期清理**：卸载不常用 MCP，减少上下文干扰

### 2. 常见问题
#### 问题 1：MCP 不显示/连接失败
- 重启 Claude Code（必须完全退出）
- 检查 JSON 语法（逗号、引号）
- 验证命令/URL 正确、依赖已安装（如 `npx`）
- 查看日志：`~/.claude/logs/mcp-*.log`

#### 问题 2：AI 不自动调用 MCP
- 描述更精准（如 `用 GitHub 创建 PR` 而非 `提交代码`）
- 用 URI 强制指定：`@github create-pr ...`
- 检查 MCP 已启用（`/mcp` 查看）

#### 问题 3：国产模型下失效
- **不可能失效**：MCP 是客户端协议，与模型无关
- 检查配置路径与重启

---

## 九、总结
### 核心要点
- **MCP = AI 通用连接协议**，让 Claude 接入真实世界（文件/Git/DB/设计）
- **两种模式**：Stdio（本地）、HTTP（远程）
- **全模型支持**：Claude 3 / 国产模型均可用
- **三大能力**：Tools（执行）、Resources（读取）、Prompts（流程）
- **核心价值**：**实时数据、跨系统操作、零胶水代码、安全可控**

### 一句话建议
**必装 5 个 MCP**：`fs`（文件）、`github`（代码）、`postgres`（数据库）、`context7`（最新文档）、`figma`（设计），让 Claude Code 从“聊天助手”变成**全栈开发中枢**。