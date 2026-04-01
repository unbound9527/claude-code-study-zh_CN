# 安全最佳实践

## 绝对不能告诉 AI

| 类型 | 后果 |
|------|------|
| 密码/密钥 | 被盗用 |
| API Keys | 被滥用，产生费用 |
| 银行卡信息 | 金融诈骗 |
| 身份证信息 | 身份盗用 |

## 正确做法

```
❌ "帮我查密码：[粘贴密码]"
✅ "帮我设计密码策略"
```

## 危险命令

| 命令 | 风险 |
|------|------|
| `rm -rf /` | 删除整个系统 |
| `git push --force` | 覆盖远程历史 |
| `curl` 粘贴敏感信息 | 信息被记录 |

## API 密钥安全

```
❌ 写在代码里：API_KEY = "sk-xxxx"
✅ 用环境变量：API_KEY = process.env.API_KEY
```

## .gitignore

```
.env
*.key
credentials.json
secrets.yml
```

---

**下一步**：[模型选择与切换](10-model-switch.md)
