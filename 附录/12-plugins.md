# Plugins：插件系统

插件是 Claude Code 的功能打包方案，将多个相关功能组合成一个可复用的包。

---

## 什么是插件

插件（Plugins）是一组预配置的功能集合，包含：
- 斜杠命令（Slash Commands）
- 子代理（Subagents）
- MCP 配置
- Hooks 配置

安装插件后，所有功能自动可用。

---

## 内置插件

### 1. PR Review 插件

完整的 PR 审查工作流：

**包含内容**：
- `/pr-review` 斜杠命令
- 代码审查子代理
- 安全扫描 Hooks
- 自动化检查清单

**安装**：
```
/plugin install pr-review
```

**使用**：
```
/pr-review
```

### 2. DevOps Automation 插件

自动化部署和监控：

**包含内容**：
- `/deploy` 部署命令
- `/rollback` 回滚命令
- 部署前检查 Hooks
- 健康检查脚本

**安装**：
```
/plugin install devops-automation
```

### 3. Documentation 插件

文档自动生成：

**包含内容**：
- `/generate-docs` 文档生成
- `/update-changelog` 更新日志
- API 文档模板
- README 生成器

---

## 插件结构

```
my-plugin/
├── commands/
│   ├── command1.md
│   └── command2.md
├── agents/
│   └── agent1.md
├── hooks/
│   └── hooks.json
├── mcp/
│   └── mcp.json
├── SKILL.md          # 插件元数据
└── README.md          # 插件说明
```

### SKILL.md 示例

```markdown
---
name: my-plugin
description: 我的自定义插件
version: 1.0.0
author: Your Name
---

# My Plugin

插件功能说明...

## 使用方式

### 斜杠命令
- /my-command

### 配置要求
- Node.js >= 16
- 特定环境变量...
```

---

## 创建自定义插件

### 1. 创建插件目录结构

```bash
mkdir -p my-plugin/{commands,agents,hooks}
cd my-plugin
```

### 2. 创建斜杠命令

```markdown
# commands/hello.md

# Hello Command

执行打招呼操作。

## 使用方式

```
/hello
```

## 参数

| 参数 | 说明 |
|------|------|
| name | 名字（可选） |

## 示例

```
/hello
/hello Claude
```

## 实现

根据输入显示友好的问候信息。
```

### 3. 创建 Hooks 配置

```json
{
  "name": "my-plugin-hooks",
  "version": "1.0.0",
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write",
      "hooks": ["./hooks/format-on-write.sh"]
    }]
  }
}
```

### 4. 创建 SKILL.md

```markdown
---
name: my-plugin
description: 插件描述
version: 1.0.0
---

# My Plugin

详细说明...

## 安装

复制到 `~/.claude/skills/` 目录。
```

---

## 插件管理

### 查看已安装插件

```
/plugin list
```

### 安装插件

```
/plugin install <plugin-name>
```

### 卸载插件

```
/plugin uninstall <plugin-name>
```

### 更新插件

```
/plugin update <plugin-name>
```

---

## 插件来源

### 官方插件

Claude Code 官方提供的插件，稳定可靠。

### 社区插件

第三方开发者创建的插件，需谨慎评估。

### 自定义插件

根据团队需求自行开发。

---

## 最佳实践

### 1. 保持插件职责单一

每个插件专注于一个功能领域，便于维护和复用。

### 2. 清晰的文档

插件 README 应包含：
- 功能说明
- 安装要求
- 使用示例
- 注意事项

### 3. 版本控制

使用语义化版本号，便于追踪依赖。

### 4. 测试验证

安装后验证所有功能正常工作。

---

## 下一步

- 查看更多高级功能：[Advanced Features](14-advanced-features.md)
- 了解 Hooks 进阶用法：[Hooks 进阶](../L4-精通技巧/14-hooks-advanced.md)
- 配置 MCP：[MCP 协议](../L4-精通技巧/07-mcp.md)
