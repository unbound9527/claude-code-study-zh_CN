# 权限管理

## 权限类型

| 权限 | 风险 |
|------|------|
| Read | 低 |
| Write/Edit | 中 |
| Bash | 高 |
| WebFetch | 中 |

## 危险操作

| 操作 | 风险 |
|------|------|
| `rm -rf /` | 🔴 极高 |
| `git push --force` | 🟠 高 |
| `curl` 敏感信息 | 🟠 高 |

## 配置

`settings.json`：
```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit"],
    "deny": ["rm -rf", "git push --force"]
  }
}
```

## 权限级别

| 级别 | 配置 | 适用 |
|------|------|------|
| 严格 | 只读 | 学习研究 |
| 中等 | 读写，禁危险命令 | 日常使用 |
| 宽松 | 全部允许 | 信赖环境 |

## 实时请求

```
⚠️ 请求执行：[Bash] rm -rf node_modules/
[y]允许 [n]拒绝 [a]始终允许 [d]始终拒绝
```

---

**下一步**：[MCP：连接外部世界](07-mcp.md)
