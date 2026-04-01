# Hooks 原理与官方指南

## 本章目标

- 深入理解 Hooks 的工作原理
- 掌握官方推荐的 Hooks 配置
- 能编写功能完整的 Hook 脚本

---

## Hooks 工作原理

### Hook 在执行流程中的位置

```
用户输入
    ↓
LLM 处理
    ↓
┌─────────────────────────────────────┐
│          PRE-TOOL HOOK              │
│   (工具执行前触发)                   │
│   - 检查参数                        │
│   - 记录日志                        │
│   - 请求确认                        │
│   - 阻止执行                        │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│          TOOL EXECUTION             │
│          (实际执行工具)               │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│         POST-TOOL HOOK               │
│   (工具执行后触发)                   │
│   - 记录结果                        │
│   - 后续处理                        │
│   - 触发通知                        │
└─────────────────────────────────────┘
    ↓
LLM 处理结果
    ↓
返回给用户
```

---

## 官方 Hooks 类型

### 1. preToolUse Hook

**触发时机**：工具执行前

**参数**：
```bash
$1 = 工具名称 (Read, Write, Bash, etc.)
$2 = 工具参数 (JSON 格式)
```

**返回值**：
- `0` = 允许执行
- `1` = 阻止执行
- 修改参数可以影响工具执行

### 2. postToolUse Hook

**触发时机**：工具执行后

**参数**：
```bash
$1 = 工具名称
$2 = 执行结果
$3 = 执行耗时（毫秒）
```

### 3. preCommit Hook

**触发时机**：Git 提交前

**参数**：无

**返回值**：
- `0` = 允许提交
- `1` = 阻止提交

### 4. postCommit Hook

**触发时机**：Git 提交后

**参数**：
```bash
$1 = 提交信息
$2 = 提交 hash
```

---

## 官方推荐配置

### 安全检查 Hook

```bash
#!/bin/bash
# ~/.claude/hooks/pre-tool-use.sh

TOOL_NAME=$1
ARGS=$2

# 日志记录
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Tool: $TOOL_NAME" >> ~/.claude/logs/hooks.log

# 危险命令检查
case "$TOOL_NAME" in
  "Bash")
    # 检查危险操作
    if echo "$ARGS" | grep -qE "(rm -rf|sudo su|curl.*password)"; then
      echo "⚠️ 检测到潜在危险操作"
      echo "命令：$TOOL_NAME $ARGS"
      exit 1  # 阻止执行
    fi
    ;;
  "Write"|"Edit")
    # 检查是否写入敏感文件
    if echo "$ARGS" | grep -qE "(\.env|\.key|secrets)"; then
      echo "⚠️ 检测到写入敏感文件"
      exit 1
    fi
    ;;
esac

exit 0  # 允许执行
```

### 提交前检查 Hook

```bash
#!/bin/bash
# ~/.claude/hooks/pre-commit.sh

echo "🔍 运行提交前检查..."

# 检查是否有敏感信息
if git diff --cached | grep -qE "(password|api_key|secret)"; then
  echo "❌ 错误：检测到敏感信息"
  exit 1
fi

# 运行测试（如果有）
if [ -f "package.json" ]; then
  echo "📦 运行测试..."
  npm test || { echo "❌ 测试失败"; exit 1; }
fi

echo "✅ 检查通过"
exit 0
```

### 性能监控 Hook

```bash
#!/bin/bash
# ~/.claude/hooks/post-tool-use.sh

TOOL_NAME=$1
RESULT=$2
DURATION=$3

# 记录到日志
echo "[$(date)] $TOOL_NAME 完成，耗时: ${DURATION}ms" >> ~/.claude/logs/performance.log

# 慢操作警告（超过 30 秒）
if [ $DURATION -gt 30000 ]; then
  echo "⚠️ 警告：$TOOL_NAME 执行超过 30 秒"
fi

exit 0
```

---

## 官方配置格式

在 `~/.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "preToolUse": "./hooks/pre-tool-use.sh",
    "postToolUse": "./hooks/post-tool-use.sh",
    "preCommit": "./hooks/pre-commit.sh",
    "postCommit": "./hooks/post-commit.sh"
  }
}
```

---

## 高级用法

### 1. 修改工具参数

```bash
#!/bin/bash
# 修改 Write 工具的目标路径

TOOL_NAME=$1
ARGS=$2

if [ "$TOOL_NAME" = "Write" ]; then
  # 检查是否写入临时目录
  if echo "$ARGS" | grep -q "/tmp/"; then
    # 重定向到项目目录
    echo "已重定向写入路径"
  fi
fi

exit 0
```

### 2. 条件执行

```bash
#!/bin/bash

TOOL_NAME=$1
ARGS=$2

# 只有在特定条件下才执行检查
if [ "$CLAUDE_ENV" = "PRODUCTION" ]; then
  # 生产环境更严格的检查
  case "$TOOL_NAME" in
    "Bash")
      if echo "$ARGS" | grep -qE "(rm -rf|drop table)"; then
        echo "❌ 生产环境禁止危险操作"
        exit 1
      fi
      ;;
  esac
fi

exit 0
```

### 3. 异步通知

```bash
#!/bin/bash

TOOL_NAME=$1
ARGS=$2

# 异步发送通知，不阻塞执行
(
  if [ "$TOOL_NAME" = "Bash" ]; then
    # 发送 Slack 通知
    curl -X POST -H 'Content-type: application/json' \
      --data "{\"text\":\"Bash 命令执行: $ARGS\"}" \
      https://hooks.slack.com/services/XXX &
  fi
) &

exit 0
```

---

## 官方最佳实践

| 实践 | 说明 |
|------|------|
| 快速退出 | Hook 应该快速执行，不要阻塞 |
| 错误处理 | 妥善处理各种错误情况 |
| 日志记录 | 记录关键操作便于调试 |
| 条件检查 | 只检查必要的条件 |
| 安全优先 | 危险操作必须确认 |

---

## 调试 Hooks

### 启用调试

```bash
export CLAUDE_HOOKS_DEBUG=1
claude
```

### 测试 Hook

```bash
# 直接运行测试
./hooks/pre-tool-use.sh Read '{"filePath":"/test.txt"}'
echo "Exit code: $?"

# 测试日志
cat ~/.claude/logs/hooks.log
```

---

## 跟做

1. 创建一个 `preToolUse` Hook
2. 添加安全检查逻辑
3. 配置到 `settings.json`
4. 测试 Hook 是否生效

---

## 小测验

**1. preToolUse Hook 在什么时候触发？**
- A. 工具执行后
- B. 工具执行前
- C. 用户发送消息前

**2. Hook 返回什么表示阻止执行？**
- A. 0
- B. 1
- C. -1

**3. Hook 应该如何执行？**
- A. 慢速详细
- B. 快速退出
- C. 随意执行

---

## 答案

1-B | 2-B | 3-B

---

---

## 本章小结

**L4 精通技巧**带你深入 Claude Code 的高级功能，从"会用"到"精通"：

| 高级功能 | 核心价值 |
|---------|---------|
| Memory | AI 的"长期记忆"，跨会话记住重要信息 |
| Subagent | 分身术，同时处理多个独立任务 |
| Agent Teams | 团队协作，多个 AI 分工完成复杂任务 |
| MCP | 扩展 AI 能力，连接任何外部服务/数据库/API |
| Hooks | 自动化钩子，在关键时机插入自定义逻辑 |
| 安全/权限 | 保护代码安全，控制 AI 操作范围 |

**Hooks 执行流程**：
```
用户输入 → [pre_task Hook] → LLM处理 → [pre_tool Hook] → 工具执行 → [post_tool Hook] → 输出
                                    ↓
                            工具出错 → [on_tool_error Hook]
```

**MCP 架构**：
```
Claude Code ←→ MCP Client ←→ MCP Server ←→ 外部资源
                              (数据库/API/文件系统...)
```

**本章重点记住**：L4 的每个功能都是"扩展 AI 能力边界"的利器。组合使用可以实现高度自定义的 AI 工作流。

---

## 下一步

- [附录：MCP 官方文档](../附录/mcp-official.md)
