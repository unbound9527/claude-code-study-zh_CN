# Hooks：自动化钩子

Hooks 让你在 Claude Code 的关键事件发生时自动执行脚本，实现自动化工作流。

---

## 核心概念

### 工作原理

```
无 Hooks：你让 AI 做 A → AI 直接做 A

有 Hooks：你让 AI 做 A → [触发 Hook] → 执行检查/修改/记录 → 执行 A
```

### 四种 Hook 类型

| 类型 | 说明 | 用途 |
|------|------|------|
| **Command Hooks** | 通过 stdin/stdout 执行 shell 脚本 | 默认类型 |
| **HTTP Hooks** | 发送请求到远程 webhook | 远程通知、集成 |
| **Prompt Hooks** | 用 LLM 评估提示内容 | 智能检查 |
| **Agent Hooks** | 用子代理进行多步验证 | 复杂推理 |

---

## 25 个 Hook 事件

### 工具 Hooks

| 事件 | 触发时机 | 常见用途 |
|------|----------|----------|
| `PreToolUse` | 工具执行前 | 确认、拦截、参数修改 |
| `PostToolUse` | 工具执行后 | 日志记录、后处理 |
| `PostToolUseFailure` | 工具执行失败 | 错误处理、告警 |
| `PermissionRequest` | 请求权限时 | 权限审查 |

### 会话 Hooks

| 事件 | 触发时机 | 常见用途 |
|------|----------|----------|
| `SessionStart` | 会话开始 | 初始化、加载上下文 |
| `SessionEnd` | 会话结束 | 清理、保存状态 |
| `Stop` | 会话停止 | 资源释放 |
| `StopFailure` | 会话停止失败 | 错误处理 |
| `SubagentStart` | 子代理启动 | 任务分配记录 |
| `SubagentStop` | 子代理结束 | 结果收集 |

### 任务 Hooks

| 事件 | 触发时机 | 常见用途 |
|------|----------|----------|
| `UserPromptSubmit` | 用户提交提示 | 输入验证、过滤 |
| `TaskCompleted` | 任务完成 | 完成通知、记录 |
| `TaskCreated` | 任务创建 | 任务跟踪 |
| `TeammateIdle` | 团队成员空闲 | 任务再分配 |

### 生命周期 Hooks

| 事件 | 触发时机 | 常见用途 |
|------|----------|----------|
| `ConfigChange` | 配置变更 | 配置验证 |
| `CwdChanged` | 工作目录变更 | 环境检查 |
| `FileChanged` | 文件变更 | 监听、同步 |
| `PreCompact` | 上下文压缩前 | 状态保存 |
| `PostCompact` | 上下文压缩后 | 状态恢复 |
| `WorktreeCreate` | 创建 worktree | 分支初始化 |
| `WorktreeRemove` | 删除 worktree | 清理 |
| `Notification` | 发送通知 | 通知处理 |
| `InstructionsLoaded` | 指令加载 | 指令验证 |
| `Elicitation` | 请求更多信息 | 输入处理 |
| `ElicitationResult` | 用户响应请求 | 响应处理 |

---

## 快速开始

### 1. 创建 Hook 目录

```bash
mkdir -p ~/.claude/hooks
```

### 2. 创建 Hook 脚本

```bash
#!/bin/bash
# ~/.claude/hooks/pre-tool-use.sh

# 从 stdin 读取 JSON 输入
input=$(cat)

# 提取工具名称
tool=$(echo "$input" | jq -r '.tool_name')

echo "将要执行工具: $tool"
echo "$input"
```

### 3. 配置 Hook

编辑 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": {
      "matcher": ".*",
      "hooks": ["~/.claude/hooks/pre-tool-use.sh"]
    }
  }
}
```

### 4. 脚本返回格式

```bash
# 允许执行（默认）
exit 0

# 阻止执行
exit 1

# 修改输入（stdout 输出新内容）
echo "modified_input"
```

---

## 实战示例

### 示例 1：代码格式化（PreToolUse）

每次写入文件前自动格式化：

```bash
#!/bin/bash
# pre-write-format.sh

input=$(cat)
tool=$(echo "$input" | jq -r '.tool_name')

# 只处理 Write 工具
if [ "$tool" = "Write" ]; then
  file=$(echo "$input" | jq -r '.parameters.file_path')
  # 格式化文件
  if [[ "$file" == *.js ]]; then
    npx prettier --write "$file" 2>/dev/null
  fi
fi

echo "$input"
```

### 示例 2：Git 提交前检查（PreCommit）

```bash
#!/bin/bash
# pre-commit.sh

set -e

echo "🔍 运行提交前检查..."

# ESLint 检查
npx eslint src/ --quiet || {
  echo "❌ ESLint 检查失败"
  exit 1
}

# 运行测试
npm test || {
  echo "❌ 测试失败"
  exit 1
}

# 检查敏感信息
if git diff --staged | grep -i "password\|api_key\|secret" > /dev/null; then
  echo "❌ 检测到敏感信息，请检查后再提交"
  exit 1
fi

echo "✅ 所有检查通过"
exit 0
```

### 示例 3：日志记录（PostToolUse）

```bash
#!/bin/bash
# post-tool-use.sh

input=$(cat)
result=$(jq -r '.result' <<< "$input")
tool=$(jq -r '.tool_name' <<< "$input")

# 记录到日志文件
timestamp=$(date '+%Y-%m-%d %H:%M:%S')
echo "[$timestamp] $tool: $result" >> ~/.claude/hooks/activity.log
```

### 示例 4：安全扫描（PostToolUse）

```bash
#!/bin/bash
# security-scan.sh

input=$(cat)
tool=$(echo "$input" | jq -r '.tool_name')

# 只扫描 Write 操作
if [ "$tool" = "Write" ]; then
  content=$(echo "$input" | jq -r '.parameters.content')
  file=$(echo "$input" | jq -r '.parameters.file_path')

  # 检测硬编码密钥
  if echo "$content" | grep -E "api[_-]?key|token|secret|password" > /dev/null; then
    echo "⚠️ 警告：文件 $file 可能包含敏感信息"
  fi
fi
```

### 示例 5：HTTP Webhook 通知

```json
{
  "hooks": {
    "TaskCompleted": {
      "matcher": ".*",
      "hooks": [{
        "type": "http",
        "url": "https://your-webhook.com/notify",
        "method": "POST",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "event": "task_completed",
          "timestamp": "${timestamp}",
          "task": "${task_description}"
        }
      }]
    }
  }
}
```

---

## 配置位置

| 位置 | 作用域 | 用途 |
|------|--------|------|
| `~/.claude/settings.json` | 用户级 | 全局 Hook |
| `.claude/settings.json` | 项目级 | 项目共享 Hook |
| `.claude/settings.local.json` | 本地 | 本地专用 Hook |

### 项目级配置示例

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": ["./.claude/hooks/pre-bash.sh"]
    }],
    "PreCommit": [{
      "matcher": ".*",
      "hooks": ["./.claude/hooks/pre-commit.sh"]
    }]
  }
}
```

---

## Matcher 语法

Matcher 决定哪些操作触发 Hook：

```json
{
  "hooks": {
    "PreToolUse": {
      "matcher": "Write|Edit",  // 匹配 Write 或 Edit
      "hooks": ["hook.sh"]
    },
    "PreToolUse": {
      "matcher": "Bash.*",       // 正则匹配
      "hooks": ["hook.sh"]
    }
  }
}
```

### 常用 Matcher

| Matcher | 匹配 |
|---------|------|
| `.*` | 所有操作 |
| `Write` | 写入文件 |
| `Edit` | 编辑文件 |
| `Bash` | 执行命令 |
| `Read` | 读取文件 |
| `Write\|Edit` | 写入或编辑 |
| `Bash.*deploy.*` | 包含 deploy 的 Bash 命令 |

---

## HTTP Hooks（v2.1.63+）

远程 webhook 端点，无需部署脚本：

```json
{
  "hooks": {
    "TaskCompleted": [{
      "matcher": ".*",
      "hooks": [{
        "type": "http",
        "url": "https://api.example.com/webhook",
        "method": "POST",
        "headers": {
          "Authorization": "Bearer ${WEBHOOK_TOKEN}"
        },
        "body": {
          "event": "task_completed",
          "task": "${task_description}",
          "timestamp": "${timestamp}"
        }
      }]
    }]
  }
}
```

---

## Prompt Hooks

用 LLM 评估用户提示：

```json
{
  "hooks": {
    "UserPromptSubmit": [{
      "matcher": ".*",
      "hooks": [{
        "type": "prompt",
        "prompt": "检查以下用户提示是否包含敏感信息：${user_prompt}\n如果包含，返回 'BLOCK' 并说明原因。"
      }]
    }]
  }
}
```

---

## 最佳实践

### 1. 保持 Hook 简单快速

- 复杂逻辑用子代理或单独脚本
- 避免在 Hook 中执行耗时的操作
- 使用异步处理长时间任务

### 2. 清晰的日志输出

```bash
echo "🔍 检查中..."
echo "✅ 通过"
echo "❌ 失败: 具体原因"
```

### 3. 合理的错误处理

```bash
# 允许部分失败
set +e
some_command
result=$?
set -e

if [ $result -ne 0 ]; then
  echo "⚠️ 警告：部分检查未通过"
fi
```

### 4. 安全注意事项

- 不要在 Hook 中硬编码敏感信息
- 使用环境变量
- 控制文件权限 `chmod 700 hooks/*.sh`

### 5. 测试 Hook

```bash
# 手动测试
echo '{"tool_name":"Write"}' | ~/.claude/hooks/pre-tool-use.sh
```

---

## 常见问题

**Q：Hook 不生效怎么办？**

A：检查：
1. 脚本有执行权限 `chmod +x hook.sh`
2. 配置文件 JSON 格式正确
3. Matcher 匹配目标操作
4. 查看日志输出

**Q：如何临时禁用 Hook？**

A：在 `settings.json` 中注释掉或删除对应配置。

**Q：多个 Hook 如何执行？**

A：按配置顺序依次执行，遇到失败停止。

**Q：Hook 可以修改 AI 的行为吗？**

A：可以。PreToolUse Hook 可以修改输入参数，返回不同的内容影响后续执行。

---

## 下一步

- 深入进阶：[Hooks 进阶用法](14-hooks-advanced.md)
- 团队协作：[Agent Teams](03-agent-teams.md)
- 完整配置：[权限管理](11-permissions.md)
