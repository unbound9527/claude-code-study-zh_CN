# Hooks 原理与官方指南

## 执行流程

```
用户输入 → LLM处理 → [pre_tool Hook] → 工具执行 → [post_tool Hook] → 输出
```

## 官方 Hooks 类型

| Hook | 时机 | 参数 |
|------|------|------|
| preToolUse | 执行前 | `$1=工具名, $2=参数` |
| postToolUse | 执行后 | `$1=工具名, $2=结果, $3=耗时` |
| preCommit | 提交前 | 无 |
| postCommit | 提交后 | `$1=提交信息, $2=hash` |

**返回值**：`0` = 允许，`1` = 阻止

## 配置

`~/.claude/settings.json`：
```json
{
  "hooks": {
    "preToolUse": "./hooks/pre-tool-use.sh",
    "preCommit": "./hooks/pre-commit.sh"
  }
}
```

## 示例：安全检查

```bash
#!/bin/bash
TOOL_NAME=$1
ARGS=$2

if echo "$ARGS" | grep -qE "(rm -rf|sudo)"; then
  echo "⚠️ 检测到危险操作"
  exit 1
fi
exit 0
```

## 最佳实践

- 快速退出，不要阻塞
- 错误处理完善
- 危险操作必须确认

---

**下一步**：[附录：MCP 官方文档](../附录/mcp-official.md)
