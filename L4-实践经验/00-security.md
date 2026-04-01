# Claude Code 安全最佳实践（权威来源整理）
本文整理 **Anthropic 官方、安全厂商（Trail of Bits、SlowMist）、企业实践** 中可信赖的 Claude Code 安全最佳实践，每条均标注来源与作者/机构，覆盖权限、沙箱、文件隔离、审计、Computer Use、企业部署全场景。

---

## 一、官方核心安全原则（Anthropic 官方）
### 1. 最小权限原则（默认安全）
- **来源**：Anthropic 官方文档《Claude Code Security》、《Sandboxing Engineering Blog》
- **作者**：Anthropic 安全与工程团队
- **要点**：
  - 默认：**读自动通过，写/执行必须确认**（`default` 模式）
  - 禁止全局开放 `Bash(*)`、`--dangerously-skip-permissions`
  - 工具分级：**只读（安全）→ 修改（中危）→ Bash（高危）**
  - 规则优先级：**`deny` > `ask` > `allow`**

### 2. 沙箱强制启用（OS 级隔离）
- **来源**：Anthropic 《Making Claude Code more secure with sandboxing》
- **作者**：Anthropic 工程团队（2025-10-20）
- **强制要求**：
  - **文件沙箱**：仅允许访问项目目录，禁止父目录/系统文件
  - **网络沙箱**：仅允许 `allowedDomains`，禁止本地端口/未知域名
  - 基于 Linux `bubblewrap` / macOS `seatbelt` 实现，覆盖所有子进程
- **配置（必加）**：
```json
{
  "permissions": {
    "sandbox": { "enabled": true, "readOnly": false }
  },
  "network": {
    "allowedDomains": ["github.com", "*.npmjs.org"],
    "allowLocalBinding": false
  }
}
```

### 3. 分层权限体系（官方架构）
- **来源**：Anthropic 《Claude Code Settings》
- **作者**：Anthropic 文档团队
- **三层优先级（下层覆盖上层）**：
  1. **User（全局）**：`~/.claude/settings.json`（个人默认）
  2. **Project（项目）**：`./.claude/settings.json`（团队共享，Git 提交）
  3. **Local（本地）**：`./.claude/settings.local.json`（个人覆盖，`.gitignore`）

---

## 二、文件与敏感数据保护（官方+安全厂商）
### 1. `.claudeignore` 双重隔离（官方+Trail of Bits）
- **来源**：Anthropic 官方、Trail of Bits `claude-code-config`
- **作者**：Anthropic、Trail of Bits（顶级安全公司）
- **强制清单（必配）**：
```
# 敏感文件
.env
.env.*
secrets.yaml
*.pem
*.key
config/*.json

# 敏感目录
secrets/
config/
node_modules/
dist/
.ssh/
```
- **额外防护**：`permissions.denyPaths` 同步配置

### 2. 禁止硬编码密钥（官方+安全内参）
- **来源**：Anthropic 安全指南、安全内参《Claude Code 容器安全》
- **作者**：Anthropic、安全内参团队
- **规则**：
  - 项目根目录创建 `CLAUDE.md`，明确安全红线
  - 禁止 AI 生成/读取/修改含密钥文件
  - 环境变量通过 `containerEnv` 传递，不硬编码

---

## 三、权限精细控制（官方+社区权威）
### 1. 工具白名单（官方+火山引擎）
- **来源**：Anthropic 官方、火山引擎《Claude Code 最佳实践》
- **作者**：Anthropic、火山引擎开发者社区
- **标准 allow 规则（直接复制）**：
```json
{
  "permissions": {
    "allow": [
      "Read", "Grep", "Glob", // 只读（安全）
      "Edit(./src/**)", "Edit(./tests/**)", // 仅允许修改代码
      "Bash(git*)", "Bash(npm run lint)", "Bash(npm run test)" // 仅安全命令
    ],
    "deny": [
      "Bash(rm -rf*)", "Bash(sudo*)", "Bash(chmod 777*)", // 高危命令
      "Edit(.env)", "Read(secrets/**)" // 敏感文件
    ]
  }
}
```

### 2. 5 种权限模式规范（官方）
- **来源**：Anthropic 《Claude Code Permissions》
- **作者**：Anthropic 安全团队
- **推荐使用**：
  - **日常**：`acceptEdits`（读写自动，执行确认）
  - **探索**：`plan`（**只读零风险**，禁止任何修改）
  - **长任务**：`auto`（安全分类器，比 `dontAsk` 安全）
  - **禁止**：`dontAsk`（生产环境绝对禁用）

---

## 四、Computer Use（电脑操控）安全（官方+企业）
### 1. 最小权限启用（官方）
- **来源**：Anthropic 《Computer Use Security》
- **作者**：Anthropic 安全团队
- **强制配置**：
```json
{
  "permissions": {
    "computerUse": {
      "enabled": true,
      "allowScreenshot": true,
      "allowInput": true,
      "allowFileAccess": ["./src", "./dist"],
      "denyPaths": ["~/.ssh", "~/Documents", ".env"],
      "requireApproval": ["delete", "password", "systemSettings"]
    }
  }
}
```
- **红线**：**禁止输入密码、修改系统设置、删除关键文件**

### 2. 安全使用规范（企业实践）
- **来源**：CSDN《阻止 Claude Code 泄露源码》
- **作者**：七牛云安全团队（2026-04-01）
- **规则**：
  - 仅在 **开发/测试机** 使用，**生产环境禁用**
  - 首次使用 **全程监控**，熟悉后再后台运行
  - `Ctrl+C` 随时紧急停止

---

## 五、审计与合规（官方+安全厂商）
### 1. 全量日志审计（官方）
- **来源**：Anthropic 官方
- **作者**：Anthropic 工程团队
- **路径**：`~/.claude/logs/permissions-*.log`、`computer-use-*.log`
- **审计内容**：所有操作（时间、工具、命令、路径、权限结果）

### 2. 企业合规（Anthropic+安全厂商）
- **来源**：Awesome Claude Code Security（GitHub）
- **作者**：社区安全专家（维护 Trail of Bits、SlowMist 清单）
- **合规清单**：
  - 签署 **DPA（数据处理协议）**，明确数据不用于训练
  - CI/CD 集成权限检查，禁止 `--dangerously-skip-permissions`
  - 定期审查日志、更新安全规则

---

## 六、安全厂商黄金标准（Trail of Bits）
### 1. 生产级配置模板
- **来源**：Trail of Bits `claude-code-config`（GitHub 安全标杆）
- **作者**：Trail of Bits（全球顶级安全公司）
- **核心加固**：
  - 沙箱强制开启 + 文件/网络双重隔离
  - 命令过滤：禁止 `rm -rf`、`sudo`、`curl | bash`
  - 敏感路径黑名单 + `.claudeignore` 双重防护
  - 审计日志 +  hooks 自动告警

### 2. MCP 安全规范（SlowMist）
- **来源**：SlowMist MCP Security Checklist
- **作者**：慢雾安全（区块链/AI 安全头部厂商）
- **要点**：
  - MCP 连接需 **信任验证**
  - 输入验证、速率限制、RBAC 权限
  - 凭证加密、容器加固

---

## 七、最佳实践总结（权威汇总）
### 1. 个人开发者必做（3 步）
1. **启用沙箱**：`sandbox.enabled: true`（官方强制）
2. **配置 `.claudeignore` + `denyPaths`**（双重隔离）
3. **默认模式 `acceptEdits`**，精细 `allow/deny` 规则

### 2. 团队/企业必做（5 步）
1. **项目级 `settings.json`**（Git 提交，全员统一）
2. **禁止危险标志**：`--dangerously-skip-permissions`
3. **CI/CD 安全检查**：验证权限配置
4. **审计日志**：定期审查
5. **Computer Use 严格限制**：仅开发机使用

### 3. 权威来源清单（可追溯）
1. **Anthropic 官方**：https://www.anthropic.com/engineering/claude-code-sandboxing
2. **Anthropic 文档**：https://docs.anthropic.com/zh-CN/docs/claude-code/settings
3. **Trail of Bits**：https://github.com/trailofbits/claude-code-config
4. **SlowMist**：https://github.com/efij/awesome-claude-code-security
5. **七牛云**：https://blog.csdn.net/aidoudoulong/article/details/159726775

---

## 八、一句话安全底线
**所有项目必配：沙箱 + `.claudeignore` + 精细权限 + 审计日志**，遵循 **最小权限、默认安全、可审计** 三大原则（Anthropic 官方+Trail of Bits 联合推荐）。