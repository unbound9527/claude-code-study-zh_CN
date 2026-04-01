# Hooks：自动化钩子

## 本章目标

- 理解什么是 Hooks
- 知道 Hooks 能做什么
- 能配置简单的 Hook

---

## 什么是 Hooks？

**Hooks = 钩子程序**

Hooks 让你在 Claude Code 执行某些操作时自动触发你预设的脚本。

```
没有 Hooks 时：
你让 AI 做 A → AI 直接做 A

有 Hooks 时：
你让 AI 做 A → [触发 Hook] → 检查/修改/记录 → 执行 A
```

---

## Hooks 类型

| Hook 类型 | 触发时机 | 常见用途 |
|-----------|----------|----------|
| `preToolUse` | 工具执行前 | 确认、记录、拦截 |
| `postToolUse` | 工具执行后 | 记录日志、后续处理 |
| `preCommit` | Git 提交前 | 代码检查、测试 |
| `postCommit` | Git 提交后 | 通知、部署 |

---

## 常见 Hook 场景

### 1. 安全确认 Hook

**场景**：执行危险命令前二次确认

```bash
#!/bin/bash
# preToolUse hook

TOOL_NAME=$1
ARGS=$2

# 检查危险命令
if echo "$ARGS" | grep -qE "rm -rf|sudo su"; then
    echo "⚠️ 警告：即将执行危险操作"
    echo "命令：$TOOL_NAME $ARGS"
    echo "请在 Claude Code 中确认是否继续"
fi
```

### 2. 日志记录 Hook

**场景**：记录所有操作

```bash
#!/bin/bash
# postToolUse hook

TOOL_NAME=$1
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
LOG_FILE="$HOME/.claude/logs/operations.log"

echo "[$TIMESTAMP] $TOOL_NAME" >> "$LOG_FILE"
```

### 3. 提交前检查 Hook

**场景**：Git 提交前自动检查

```bash
#!/bin/bash
# preCommit hook

echo "🔍 运行提交前检查..."

# 运行 ESLint
if ! npx eslint src/ --quiet; then
    echo "❌ ESLint 检查失败"
    exit 1
fi

# 运行测试
if ! npm test; then
    echo "❌ 测试失败"
    exit 1
fi

echo "✅ 检查通过"
exit 0
```

---

## 配置 Hooks

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

### 目录结构

```
~/.claude/
├── settings.json
└── hooks/
    ├── pre-tool-use.sh
    ├── post-tool-use.sh
    ├── pre-commit.sh
    └── post-commit.sh
```

---

## Hook 脚本模板

### preToolUse 模板

```bash
#!/bin/bash
# 钩子名称：preToolUse
# 参数：$1=工具名, $2=参数

TOOL_NAME=$1
ARGS=$2

# 检查逻辑
if [ 条件 ]; then
    echo "警告信息"
    exit 1  # 阻止执行
fi

exit 0  # 允许执行
```

### postToolUse 模板

```bash
#!/bin/bash
# 钩子名称：postToolUse
# 参数：$1=工具名, $2=执行结果

TOOL_NAME=$1
RESULT=$2

# 记录或其他处理
echo "[$(date)] $TOOL_NAME 完成" >> ~/claude-hooks.log

exit 0
```

---

## Windows 用户

Windows 上可以用 `.bat` 或 `.ps1` 文件：

### preToolUse.bat

```batch
@echo off
set TOOL_NAME=%1
set ARGS=%2

echo [Hook] %TOOL_NAME% 将被执行
exit /b 0
```

---

## 调试 Hooks

### 启用调试

```bash
export CLAUDE_HOOKS_DEBUG=1
claude
```

### 测试 Hook

```bash
# 直接运行脚本
./hooks/pre-tool-use.sh Read "package.json"
echo "Exit code: $?"
```

---

## 最佳实践

| 原则 | 说明 |
|------|------|
| 保持简单 | Hook 逻辑不要太复杂 |
| 快速退出 | 非危险操作直接放行 |
| 清晰日志 | 方便追踪问题 |
| 错误处理 | Hook 失败时给清晰提示 |

---

## 跟做

1. 创建一个简单的 preToolUse Hook
2. 配置到 settings.json
3. 测试它是否生效

---

## 练一练

1. 创建一个记录所有操作的日志 Hook
2. 创建一个在执行删除命令前提醒的 Hook
3. 创建一个提交前自动运行检查的 Hook

---

## 小测验

**1. Hook 是在什么时候触发的？**
- A. AI 回复时
- B. 执行特定操作时
- C. 打开对话时

**2. preToolUse 和 postToolUse 的区别是？**
- A. 没有区别
- B. 执行前 vs 执行后
- C. Windows vs Mac

**3. Hook 配置在哪里？**
- A. package.json
- B. settings.json
- C. README.md

---

## 答案

1-B | 2-B | 3-B

---

## 下一步

- [附录：常见问题](../附录/常见问题.md)
