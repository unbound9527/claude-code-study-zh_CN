# Hooks：自动化钩子

## 什么是 Hooks

在 Claude Code 执行某些操作时自动触发预设脚本。

```
无 Hooks：你让 AI 做 A → AI 直接做 A

有 Hooks：你让 AI 做 A → [触发 Hook] → 检查/修改/记录 → 执行 A
```

## Hook 类型

| Hook | 触发时机 | 用途 |
|------|---------|------|
| `preToolUse` | 工具执行前 | 确认、拦截 |
| `postToolUse` | 工具执行后 | 记录日志 |
| `preCommit` | Git 提交前 | 代码检查、测试 |
| `postCommit` | Git 提交后 | 通知、部署 |

## 配置

`~/.claude/settings.json`：
```json
{
  "hooks": {
    "preToolUse": "./hooks/pre-tool-use.sh",
    "postToolUse": "./hooks/post-tool-use.sh",
    "preCommit": "./hooks/pre-commit.sh"
  }
}
```

## 示例：提交前检查

```bash
#!/bin/bash
# preCommit hook

echo "运行检查..."
npx eslint src/ --quiet || exit 1
npm test || exit 1

echo "✅ 检查通过"
exit 0
```

## 最佳实践

- 保持简单
- 非危险操作直接放行
- 清晰日志

---

**下一步**：[附录：常见问题](../附录/常见问题.md)
