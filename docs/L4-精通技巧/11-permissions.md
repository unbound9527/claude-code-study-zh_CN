# 权限管理

## 本章目标

- 理解 Claude Code 的权限系统
- 知道如何配置合理的权限
- 能根据需求调整权限级别

---

## 什么是权限管理？

**权限 = 允许或禁止 AI 执行特定操作**

Claude Code 可以做很多事：
- 读取文件
- 写入文件
- 执行命令
- 连接网络
- 等等

**权限管理让你控制 AI 能做什么、不能做什么。**

---

## 权限类型

| 权限 | 说明 | 风险 |
|------|------|------|
| **Read** | 读取文件内容 | 低 |
| **Write** | 创建或修改文件 | 中 |
| **Edit** | 修改文件部分内容 | 中 |
| **Bash** | 执行系统命令 | 高 |
| **WebFetch** | 获取网页内容 | 中 |
| **WebSearch** | 搜索网络 | 低 |
| **Agent** | 启动子代理 | 高 |

---

## 默认权限

Claude Code 有**默认的权限设置**：

| 权限 | 默认状态 |
|------|----------|
| Read | ✅ 允许 |
| Write | ✅ 允许 |
| Edit | ✅ 允许 |
| Bash | ⚠️ 部分允许 |
| WebFetch | ⚠️ 需要授权 |
| WebSearch | ⚠️ 需要授权 |
| Agent | ✅ 允许 |

---

## 危险操作

### 需要特别注意的操作

| 操作 | 风险等级 | 说明 |
|------|----------|------|
| `rm -rf /` | 🔴 极高 | 删除整个系统 |
| `rm -rf node_modules` | 🟠 高 | 删除项目依赖 |
| `git push --force` | 🟠 高 | 覆盖远程历史 |
| `curl` 发送敏感信息 | 🟠 高 | 信息泄露 |
| `chmod 777` | 🟡 中 | 权限过大 |

---

## 配置权限

### 在 settings.json 中配置

```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit"],
    "deny": ["rm -rf", "git push --force"]
  }
}
```

### 权限级别

#### 严格模式（最安全）

```json
{
  "permissions": {
    "allow": ["Read"],
    "deny": ["Write", "Edit", "Bash"]
  }
}
```

适用：学习、研究、只读任务

#### 中等模式（日常使用）

```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit"],
    "deny": [
      "rm -rf",
      "git push --force",
      "sudo"
    ]
  }
}
```

适用：大多数日常任务

#### 宽松模式（可信赖环境）

```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Bash", "WebFetch"],
    "deny": ["rm -rf /"]
  }
}
```

适用：自己电脑、完全信任的环境

---

## 实时权限请求

当 AI 要执行危险操作时，会请求你的授权：

```
⚠️ AI 请求执行以下操作：

[Bash] rm -rf node_modules/

是否允许？
[y] 允许 [n] 拒绝 [a] 始终允许此操作 [d] 始终拒绝
```

### 选项说明

| 选项 | 作用 |
|------|------|
| `y` | 允许这次 |
| `n` | 拒绝这次 |
| `a` | 允许并记住（本次会话） |
| `d` | 拒绝并记住（本次会话） |

---

## 自定义权限规则

### 1. 按命令类型拒绝

```json
{
  "permissions": {
    "deny": [
      "rm -rf",           // 禁止递归删除
      "git push --force", // 禁止强制推送
      "sudo",            // 禁止管理员权限
      "chmod 777"        // 禁止最大权限
    ]
  }
}
```

### 2. 按目录拒绝

```json
{
  "permissions": {
    "deny": [
      "rm -rf /System",
      "rm -rf /usr",
      "rm -rf ~/.ssh"
    ]
  }
}
```

### 3. 按文件类型拒绝

```json
{
  "permissions": {
    "deny": [
      "curl .*\\.env.*",      // 禁止访问 .env 文件
      "curl .*password.*"     // 禁止包含 password 的请求
    ]
  }
}
```

---

## 项目级权限

可以在项目的 `CLAUDE.md` 中指定权限：

```markdown
# 项目权限

## 允许的操作
- 读写 src/ 目录
- 执行 npm 命令

## 禁止的操作
- 删除任何 node_modules
- 修改配置文件
- git push
```

---

## 权限检查建议

### 开发前检查

1. 查看当前权限配置
2. 确认危险命令在拒绝列表
3. 重要操作前手动确认

### 定期审计

```bash
# 查看权限配置
cat ~/.claude/settings.json | grep permissions
```

---

## 跟做

1. 打开 `~/.claude/settings.json`
2. 查看当前的权限配置
3. 添加至少一条禁止规则

---

## 练一练

1. 设置严格模式，只允许读取
2. 设置中等等级，允许读写但禁止危险命令
3. 测试不同权限下的 AI 行为

---

## 小测验

**1. 哪个权限风险最高？**
- A. Read
- B. Write
- C. Bash

**2. `rm -rf /` 为什么危险？**
- A. 会删除系统文件
- B. 会删除当前目录
- C. 会重启电脑

**3. 权限配置在哪里？**
- A. package.json
- B. settings.json
- C. README.md

---

## 答案

1-C | 2-A | 3-B

---

## 下一步

- [MCP：连接外部世界](07-mcp.md)
