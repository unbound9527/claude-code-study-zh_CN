# Claude Code Hooks 完全教学：事件驱动自动化、配置与实战
Hooks 是 Claude Code 中**事件驱动的自动化系统**，让你在 AI 执行关键操作（读写文件、运行命令、提交消息）的**前后自动触发脚本**，实现安全校验、代码格式化、自动提交、日志审计、权限管控等本地自动化，相当于给 Claude 装上“自动规则引擎”。本文从概念、配置、事件、实战、排错全流程讲解，让你彻底掌握 Hooks。

---

## 一、什么是 Hooks？核心价值
### 1. 定义
Hooks = **事件触发 + 条件匹配 + 自动执行脚本**
- **事件**：AI 生命周期节点（工具调用前/后、会话开始/结束、提交消息）
- **匹配器**：过滤特定操作（如只匹配 Write/Edit/Bash）
- **动作**：执行 Shell/Python/JS 脚本，可**允许/阻止/修改** AI 行为

### 2. 与 Skills/Subagent 区别
- **Hooks**：**被动自动触发**，无人工干预，用于安全、规范、自动化
- **Skills**：**主动手动调用**，自定义快捷命令
- **Subagent**：**独立 AI 分工**，处理复杂任务

### 3. 核心价值
✅ **安全锁**：阻止危险命令（rm -rf /、修改 .env）
✅ **规范强制**：代码自动格式化、Lint、禁止硬编码密钥
✅ **流程自动化**：改完代码自动 Git 提交、自动跑测试
✅ **审计追踪**：全操作日志记录，可回溯
✅ **上下文增强**：会话自动注入项目规则、关键信息

---

## 二、配置位置与优先级（三层）
Hooks 配置在 **JSON 设置文件**，支持三层，优先级从高到低：

### 1. 配置路径
```bash
# 1. 项目本地（最高，不进 Git，个人专属）
.claude/settings.local.json

# 2. 项目级（团队共享，提交 Git）
.claude/settings.json

# 3. 全局（所有项目生效）
~/.claude/settings.json
```

### 2. 交互式管理（/hooks 命令）
直接在 Claude Code 输入：
```bash
/hooks
```
- 可视化查看、启用、禁用、编辑所有 Hooks
- 无需手写 JSON，新手友好

---

## 三、核心配置结构（三要素）
### 标准 JSON 结构
```json
{
  "hooks": {
    "事件名": [
      {
        "matcher": "工具匹配正则",
        "blocking": true, // 是否阻塞主流程（默认true）
        "timeout": 10,    // 超时秒（默认60）
        "hooks": [
          {
            "type": "command", // 仅支持 command（shell/脚本）
            "command": "要执行的脚本/命令"
          }
        ]
      }
    ]
  }
}
```

### 三要素详解
#### 1. 事件（When：何时触发）
支持 8 大核心事件（覆盖全生命周期）：

| 事件 | 触发时机 | 典型用途 |
|:--- |:--- |:--- |
| **PreToolUse** | 工具执行**前** | 安全检查、阻止危险操作、参数校验 |
| **PostToolUse** | 工具执行**后** | 代码格式化、自动提交、更新文档 |
| **UserPromptSubmit** | 用户发送消息前 | 注入上下文、敏感词过滤 |
| **SessionStart** | 会话启动 | 加载项目规则、初始化环境 |
| **Stop** | 会话结束 | 清理、通知、生成总结 |
| **PreCompact** | 上下文压缩前 | 保存关键信息、防止丢失 |
| **Notification** | AI 发通知时 | 桌面提醒、企业微信推送 |
| **SubagentStop** | 子代理结束 | 子任务收尾 |

#### 2. 匹配器（Matcher：过滤操作）
- 正则表达式，匹配**工具名**
- 仅适用于 `PreToolUse` / `PostToolUse`
- 常用匹配：
  - `"Write|Edit|MultiEdit"`：所有文件修改
  - `"Bash"`：仅终端命令
  - `"Read"`：仅读文件
  - `"*"` / 空：匹配所有工具

#### 3. 动作（Command：做什么）
- **type 固定为 command**（仅支持 Shell/脚本）
- 可执行：
  - 单行 Shell 命令
  - Shell 脚本（.sh）
  - Python/Node 脚本
- **环境变量**（脚本内可用）：
  - `$CLAUDE_TOOL`：工具名（Write/Bash）
  - `$CLAUDE_TOOL_INPUT_FILE_PATH`：文件路径
  - `$CLAUDE_PROJECT_DIR`：项目根目录
  - `$CLAUDE_NOTIFICATION_TITLE`：通知标题

---

## 四、支持模型与版本
### 1. 版本要求
- Claude Code **v2.0+**（2025 年后版本）
- **所有订阅**：免费版 / Pro / Max 均支持（无限制）

### 2. 模型兼容性
- **完全兼容所有模型**：Claude 3 Opus/Sonnet/Haiku、Claude 2、**国产模型（通义/GLM/DeepSeek）**
- Hooks 是**本地客户端功能**，与后端模型无关，**切换国产模型完全可用**

---

## 五、实战：5 个高频 Hooks（直接复制）
### 实战 1：代码修改后自动格式化（最常用）
**场景**：AI 写完/改完代码，自动 Prettier/Black 格式化
```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null; exit 0"
          }
        ]
      }
    ]
  }
}
```

### 实战 2：阻止修改敏感文件（安全锁）
**场景**：禁止 AI 改 .env、package-lock.json、dist 目录
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "blocking": true,
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json,sys;d=json.load(sys.stdin);p=d.get('tool_input',{}).get('file_path','');sys.exit(2 if any(x in p for x in ['.env','package-lock.json','dist/']) else 0)\""
          }
        ]
      }
    ]
  }
}
```
- 退出码 `2` = **阻止操作**，`0` = 允许

### 实战 3：Bash 命令安全审计（防删库）
**场景**：记录所有 Bash 命令，阻止 `rm -rf /` 等高危
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' | grep -qE 'rm -rf /|sudo|mkfs'; exit $?"
          },
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' >> .claude/bash-log.txt"
          }
        ]
      }
    ]
  }
}
```

### 实战 4：会话启动自动注入项目规则
**场景**：每次打开 Claude，自动提醒项目规范
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo '项目规则：1. 用 Bun 不用 npm；2. 提交前跑 bun test；3. 禁止 console.log'"
          }
        ]
      }
    ]
  }
}
```

### 实战 5：AI 完成任务桌面通知
**场景**：Mac/Linux 弹通知，不用盯终端
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' '任务完成'"
          }
        ]
      }
    ]
  }
}
```

---

## 六、Hooks 运行表现（你会看到什么）
### 1. 执行流程（PreToolUse）
1. 你让 AI 改文件 → AI 触发 `PreToolUse`
2. Hooks 脚本运行 → **检查/阻止/允许**
3. 允许 → AI 执行 Write/Edit
4. 执行后 → `PostToolUse` 触发 → 格式化/提交

### 2. 终端表现
- 触发 Hooks 时，终端显示：`[Hook] Executing: 命令`
- 阻止操作：`[Hook] Blocked: 敏感文件修改`
- 脚本输出：直接进入 Claude 上下文（可被 AI 读取）

### 3. 阻塞与非阻塞
- **blocking: true**（默认）：脚本跑完 AI 才继续
- **blocking: false**：后台异步执行，不阻塞

---

## 七、最佳实践与避坑
### 1. 最佳实践
- **项目级共享**：通用 Hooks 放 `.claude/settings.json`（Git 共享）
- **本地个性化**：私人 Hooks 放 `.claude/settings.local.json`（gitignore）
- **最小权限**：Hooks 只做必要事，避免复杂逻辑
- **脚本化**：复杂逻辑写独立脚本（`.claude/hooks/`）
- **超时控制**：关键 Hooks 设 `timeout: 5-10` 秒

### 2. 常见问题
#### 问题 1：Hooks 不执行
- 检查 JSON 语法（逗号、引号）
- 脚本加执行权限：`chmod +x .claude/hooks/*.sh`
- 重启 Claude Code 会话
- 用 `/hooks` 查看是否启用

#### 问题 2：误阻止正常操作
- 优化 `matcher` 正则，缩小范围
- 降低脚本严格度，避免误判

#### 问题 3：国产模型下 Hooks 失效
- **不可能失效**：Hooks 是客户端功能，与模型无关
- 检查配置路径是否正确

---

## 八、总结
### 核心要点
- **Hooks = 本地自动化引擎**，事件驱动、自动触发
- **三层配置**：本地 > 项目 > 全局
- **8 大事件**：覆盖 AI 全生命周期
- **全模型兼容**：Claude 3 / 国产模型均支持
- **核心用途**：安全、规范、自动化、审计

### 一句话建议
**所有项目必配 3 个 Hooks**：敏感文件保护、代码自动格式化、Bash 安全审计，瞬间提升 Claude Code 安全性与规范性。