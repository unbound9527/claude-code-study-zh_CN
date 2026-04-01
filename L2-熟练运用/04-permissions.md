# Claude Code 权限管理完全教学：安全边界、精细控制、实战配置
Claude Code 的权限管理是一套**分层、细粒度、可审计**的安全控制系统，核心是**最小权限原则**——只开放 AI 完成任务必需的能力，严格隔离敏感文件、高危命令与网络访问，兼顾开发效率与代码安全。本文从核心概念、权限模式、工具分级、三层配置、精细规则、沙箱与审计全流程讲解，彻底掌握 Claude Code 安全管控。

---

## 一、什么是 Claude Code 权限管理？核心定位
### 1. 官方定义
**权限管理 = 安全边界 + 操作控制 + 审计追踪**
- 本质：给 Claude Code 设定**“能做什么、不能做什么、哪些必须确认”**的规则
- 架构：**六级权限验证 + 四层决策管道**，所有操作（读/写/执行/网络）必经校验
- 目标：防止**误操作、敏感数据泄露、命令逃逸、注入攻击**，安全与效率平衡

### 2. 核心设计哲学
- **工具即权限边界**：每个工具（Read/Edit/Bash）是独立权限单元，而非开放完整 Shell
- **分层隔离**：用户/项目/本地三层配置，团队统一 + 个人定制
- **默认安全**：保守默认（写/执行需确认），权限主动授予而非被动开放
- **可审计**：全操作日志记录，可追溯、可复盘

### 3. 与 Skill/MCP/Hooks 区别
| 模块 | 定位 | 控制范围 | 触发方式 |
|:--- |:--- |:--- |:--- |
| **权限管理** | 安全底层 | 全工具/文件/命令/网络 | 全局拦截、强制校验 |
| Skill | 工作流 | 单技能工具白名单 | 手动/自动触发 |
| MCP | 外部连接 | 第三方服务权限 | 工具调用 |
| Hooks | 事件自动化 | 前置/后置拦截 | 事件驱动 |

---

## 二、支持版本与模型
### 1. 版本要求
- Claude Code **v1.0+** 全版本支持，**v2.1+** 强化精细规则（通配符、命令过滤）
- 所有订阅：免费版 / Pro / Max 均可用（企业版支持集中管控）

### 2. 模型兼容性
- **全模型支持**：Claude 3（Opus/Sonnet/Haiku）、Claude 2、国产模型（通义/GLM）
- 权限是**客户端本地控制**，与模型无关，切换模型安全规则不变

---

## 三、五大权限模式：从保守到全自动（核心）
Claude Code 内置 **5 种预设权限模式**，`Shift+Tab` 快速切换，适配全场景：

### 1. default（默认模式 · 新手推荐）
- 行为：**读文件自动通过**；**写文件、执行 Bash 必须手动确认**
- 适用：新手、陌生项目、高敏感代码库
- 特点：最安全，每步操作可控，无意外修改

### 2. acceptEdits（主力模式 · 日常首选）
- 行为：**读写文件自动通过**；**执行命令仍需确认**
- 适用：日常开发、信任 AI 修改代码、高频编辑
- 特点：效率 + 安全平衡，推荐团队默认

### 3. plan（规划模式 · 只读侦察）
- 行为：**仅允许读/搜索/分析**；**禁止任何写/执行**
- 适用：探索代码、架构分析、重构前评估、输出方案不执行
- 特点：**零修改风险**，绝对安全

### 4. auto（智能自动 · 长任务）
- 行为：内置**安全分类器**，自动判断操作风险；安全操作放行，高危拦截
- 适用：大型重构、批量修改、长时间无人值守任务
- 特点：比 `--dangerously-skip-permissions` 安全，比默认高效

### 5. dontAsk（免询问 · 信任环境）
- 行为：**所有操作自动通过**（读写/执行/网络）
- 适用：沙箱、测试环境、完全信任的私有项目
- 警告：**生产环境禁用**，风险极高

### 模式切换（3 种方式）
```bash
# 1. 会话内快捷键
Shift+Tab  # 循环切换

# 2. 命令行启动
claude --permission-mode acceptEdits  # 日常
claude --permission-mode plan         # 只读探索
claude --permission-mode auto         # 自动执行

# 3. settings.json 默认配置
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

---

## 四、工具权限分级：三类风险（必知）
所有工具按风险分为 **3 级**，权限严格递增：

### 1. 只读工具（✅ 安全，默认允许）
- 工具：`Read`、`Grep`、`Glob`、`Ls`、`Cat`
- 能力：仅读取、搜索、列出文件，**不修改任何内容**
- 权限：默认自动通过，无需确认

### 2. 修改工具（⚠️ 中危，需配置）
- 工具：`Edit`、`Write`、`NotebookEdit`
- 能力：修改/写入文件，范围可控
- 权限：`default` 需确认；`acceptEdits/auto` 自动通过

### 3. 执行工具（🔴 高危，严格管控）
- 工具：`Bash`（唯一高危）
- 能力：执行 Shell 命令，可绕过文件权限（如 `echo > file`）
- 权限：**默认必须确认**，支持**命令级精细过滤**（核心安全点）

---

## 五、三层配置体系：全局/项目/本地（核心架构）
权限按 **3 层优先级** 叠加，**本地 > 项目 > 全局**（下层覆盖上层）：

### 1. User（全局 · 个人所有项目）
- 路径：`~/.claude/settings.json`
- 作用：个人通用规则（如禁止访问 `.env`、限制 Bash）

### 2. Project（项目 · 团队共享）
- 路径：`./.claude/settings.json`
- 作用：**团队统一安全规范**（Git 提交，全员生效）
- 推荐：团队必配，统一权限基线

### 3. Local（本地 · 个人私有）
- 路径：`./.claude/settings.local.json`
- 作用：个人覆盖项目规则（`.gitignore`，不提交）
- 适用：个人临时权限、私有密钥

### 配置文件标准结构
```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [],    // 自动通过（无需确认）
    "ask": [],      // 必须确认（优先级最高）
    "deny": [],     // 绝对禁止（优先级最高）
    "denyPaths": [".env", "secrets/", "config/"], // 敏感路径
    "allowPaths": ["./src", "./utils"],          // 允许路径
    "sandbox": {
      "enabled": true,
      "readOnly": false
    }
  },
  "network": {
    "allowedDomains": ["github.com", "*.npmjs.org"], // 允许网络
    "allowLocalBinding": false
  }
}
```

---

## 六、精细权限规则：allow/ask/deny（最强控制）
三大规则 **优先级：deny > ask > allow**，支持**通配符、命令过滤、路径限定**：

### 1. deny（绝对禁止 · 红线）
- 作用：**永久阻止**，任何情况不执行
- 示例：
```json
"deny": [
  "Bash(rm -rf*)",       // 禁止删除
  "Bash(mv*)",           // 禁止移动
  "Edit(.env)",          // 禁止修改环境变量
  "Read(secrets/**)"    // 禁止读密钥目录
]
```

### 2. ask（必须确认 · 校验）
- 作用：每次操作**弹窗确认**，可选择“允许本次”或“永久允许”
- 示例：
```json
"ask": [
  "Bash(*)",             // 所有命令需确认
  "Edit(**/*.config.js)" // 配置文件修改确认
]
```

### 3. allow（自动通过 · 免确认）
- 作用：符合规则**直接执行**，无需弹窗
- 示例（**精细过滤，核心安全**）：
```json
"allow": [
  // 1. 工具级
  "Read", "Grep", "Glob",
  
  // 2. 命令级（Bash 过滤，最常用）
  "Bash(git*)",          // 仅允许 Git 命令
  "Bash(npm run test:*)",// 仅允许测试命令
  "Bash(eslint*)",       // 仅允许 Lint
  
  // 3. 路径级
  "Edit(./src/**)",      // 仅允许修改 src
  "Write(./dist/**)"     // 仅允许写 dist
]
```

### 4. 路径隔离（.claudeignore）
- 作用：类似 `.gitignore`，**彻底屏蔽敏感文件/目录**
- 路径：项目根目录 `.claudeignore`
- 示例：
```
# 敏感文件
.env
.env.*
secrets.yaml
config/*.json

# 敏感目录
secrets/
config/
node_modules/
dist/
```

---

## 七、沙箱 Sandbox：系统级隔离（企业级安全）
### 1. 什么是沙箱？
- **OS 级安全隔离**（Linux: bubblewrap / macOS: seatbelt），强制文件/网络边界
- 核心能力：
  - **文件沙箱**：仅允许访问项目目录，禁止父目录、系统文件
  - **网络沙箱**：仅允许 `allowedDomains`，禁止本地端口、未知域名
  - **进程隔离**：所有子进程受限，无法逃逸

### 2. 启用沙箱（推荐必开）
```json
// .claude/settings.json
{
  "permissions": {
    "sandbox": {
      "enabled": true,       // 启用沙箱
      "readOnly": false,      //  false=可写；true=只读
      "allowedPaths": ["./src", "./tests"] // 沙箱内允许路径
    }
  },
  "network": {
    "allowedDomains": ["github.com", "api.npmjs.org"],
    "allowLocalBinding": false // 禁止本地网络
  }
}
```

---

## 八、实战：企业/个人标准配置（直接复制）
### 1. 个人标准配置（~/.claude/settings.json）
```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "deny": [
      "Bash(rm -rf*)", "Bash(sudo*)", "Bash(chmod 777*)",
      "Edit(.env)", "Read(secrets/**)"
    ],
    "allow": [
      "Read", "Grep", "Glob", "Edit(./src/**)",
      "Bash(git*)", "Bash(npm run lint)", "Bash(npm run test)"
    ],
    "denyPaths": [".env", "secrets/", "config/"],
    "sandbox": { "enabled": true }
  },
  "network": {
    "allowedDomains": ["github.com", "*.npmjs.org"],
    "allowLocalBinding": false
  }
}
```

### 2. 团队项目配置（.claude/settings.json）
```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "deny": [
      "Bash(rm*)", "Bash(mv*)", "Bash(sudo*)",
      "Edit(**/*.env)", "Read(**/secrets*)"
    ],
    "allow": [
      "Read", "Grep", "Edit(./src/**)", "Edit(./tests/**)",
      "Bash(git*)", "Bash(npm run*)", "Bash(yarn*)"
    ],
    "sandbox": { "enabled": true, "readOnly": false }
  },
  "network": {
    "allowedDomains": ["github.com", "*.npmjs.org", "api.sentry.io"],
    "allowLocalBinding": false
  }
}
```

### 3. 高敏感项目（plan 模式 + 只读沙箱）
```json
{
  "permissions": {
    "defaultMode": "plan", // 强制只读
    "deny": ["Edit(**)", "Write(**)", "Bash(**)"],
    "allow": ["Read", "Grep", "Glob"],
    "sandbox": { "enabled": true, "readOnly": true }
  },
  "network": { "allowedDomains": [] } // 禁止所有网络
}
```

---

## 九、权限管理工具：查看/校验/审计
### 1. 查看权限（/permissions）
```bash
# 会话内查看所有权限规则
/permissions
# 显示：模式、allow/ask/deny、路径、沙箱状态、来源（全局/项目/本地）
```

### 2. 校验配置
```bash
# 校验 settings.json 语法
claude settings validate

# 测试权限（模拟操作）
claude permissions test "Edit src/app.js"
claude permissions test "Bash(git status)"
```

### 3. 审计日志
- 路径：`~/.claude/logs/permissions-*.log`
- 内容：所有操作记录（时间、工具、命令、路径、权限结果）
- 作用：安全复盘、违规追踪

---

## 十、常见问题与避坑
### 1. 问题：AI 无法修改文件（权限被拒）
- 检查：`defaultMode` 是否为 `plan`（只读）
- 检查：`denyPaths` 是否包含目标路径
- 检查：`deny` 规则是否禁止 `Edit`

### 2. 问题：Bash 命令总弹窗确认
- 解决：`allow` 加入精细规则（如 `Bash(git*)`）
- 避免：`allow: ["Bash"]`（开放所有命令，高危）

### 3. 问题：敏感文件被读取
- 必配：`.claudeignore` + `denyPaths` 双重屏蔽
- 检查：沙箱是否启用

### 4. 问题：权限配置不生效
- 重启 Claude Code（完全退出重进）
- 校验 JSON 语法（逗号、引号）
- 确认优先级：`local > project > user`

---

## 十一、最佳实践（安全 + 效率）
1. **最小权限**：仅开放必需工具/命令/路径，**禁止通配 `Bash(*)`**
2. **双层防护**：`.claudeignore` + `denyPaths` 双重隔离敏感文件
3. **团队统一**：项目 `settings.json` 提交 Git，全员基线一致
4. **沙箱必开**：所有项目启用系统级沙箱，杜绝逃逸
5. **模式适配**：日常 `acceptEdits`、探索 `plan`、长任务 `auto`
6. **定期审计**：每周审查权限日志，清理冗余规则

---

## 十二、总结
### 核心要点
- **5 种权限模式**：default/acceptEdits/plan/auto/dontAsk，适配全场景
- **3 层配置**：user/project/local，团队统一 + 个人定制
- **3 类规则**：deny（禁止）> ask（确认）> allow（自动），精细控制
- **工具分级**：只读安全、修改中危、Bash 高危，严格管控
- **沙箱隔离**：OS 级文件/网络隔离，企业级安全
- **核心价值**：**安全可控、效率平衡、可审计、防泄露、防误操作**

### 一句话建议
**所有项目必配 3 件套**：`acceptEdits` 默认模式 + 精细 `allow/deny` 规则 + 启用沙箱，既保证开发效率，又彻底杜绝安全风险。