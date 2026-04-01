# CLI 参考

Claude Code 命令行完整参考。

---

## 运行模式

### 交互模式（默认）

启动对话式 REPL：

```bash
claude
claude "帮我实现用户登录"
```

### 打印模式

单次查询后退出，适合脚本化：

```bash
claude -p "审查这段代码"
claude --print "解释这个函数"
```

---

## 主要命令

| 命令 | 说明 |
|------|------|
| `claude` | 启动交互式 REPL |
| `claude "query"` | 启动并附带初始提示 |
| `claude -p "query"` | 打印模式：查询后退出 |
| `claude -c` | 继续最近对话 |
| `claude -r "session" "query"` | 按名称恢复指定会话 |
| `claude update` | 更新到最新版本 |
| `claude mcp` | 配置 MCP 服务器 |
| `claude auth login/logout/status` | 身份验证管理 |

---

## 常用参数

### 模式参数

| 参数 | 说明 |
|------|------|
| `-p, --print` | 非交互打印模式 |
| `-c, --continue` | 加载最近会话 |
| `-r, --resume <session>` | 恢复指定会话 |
| `--fork-session <name>` | 复制会话创建分支 |

### 模型参数

| 参数 | 说明 |
|------|------|
| `--model <model>` | 选择模型（opus/sonnet/haiku） |
| `--model opusplan` | 使用 Opus 规划，Sonnet 执行 |

### 输出参数

| 参数 | 说明 |
|------|------|
| `--output-format <format>` | 输出格式（text/json/stream-json） |
| `--verbose` | 启用详细日志 |

### 权限参数

| 参数 | 说明 |
|------|------|
| `--permission-mode <mode>` | 权限模式（plan/auto/acceptEdits） |
| `--enable-auto-mode` | 启用自动模式 |
| `--tools <tools>` | 限制可用工具 |
| `--max-turns <n>` | 限制代理回合数 |
| `--bare` | 最小化模式 |

### 思考参数

| 参数 | 说明 |
|------|------|
| `--effort <level>` | 思考层级（low/medium/high/max） |
| `--no-thinking` | 禁用扩展思考 |

---

## 环境变量

| 变量 | 说明 |
|------|------|
| `ANTHROPIC_API_KEY` | API 认证密钥 |
| `ANTHROPIC_MODEL` | 默认模型 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 思考努力级别 |
| `CLAUDE_CODE_SIMPLE` | 最小化模式开关 |
| `MAX_THINKING_TOKENS` | 思考 token 上限 |

---

## 会话管理

### 继续对话

```bash
# 继续最近对话
claude -c

# 恢复指定会话
claude -r "feature-auth" "继续实现"
```

### 复制会话

```bash
# 在保留原会话的同时创建分支
claude --fork-session "try-alternative" --resume feature-auth
```

### 查看会话列表

```bash
claude sessions list
```

### 删除会话

```bash
claude sessions delete <session-name>
```

---

## MCP 配置

```bash
# 添加 MCP 服务器
claude mcp add github -- npx -y @modelcontextprotocol/server-github

# 列出已配置的 MCP
claude mcp list

# 移除 MCP
claude mcp remove github
```

---

## 身份验证

```bash
# 登录
claude auth login

# 登出
claude auth logout

# 查看状态
claude auth status
```

---

## 管道使用

### 基本管道

```bash
# 分析代码
cat error.log | claude -p "分析这个错误"

# 从文件输入
claude -p "$(cat code.ts)"

# 输出到文件
claude -p "审查代码" > review.txt
```

### JSON 输出

```bash
# 结构化输出
claude -p "列出所有函数" --output-format json

# 解析 JSON
claude -p "审查代码" --output-format json | jq '.suggestions'
```

### CI/CD 集成

```bash
# GitHub Actions 示例
- name: Code Review
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: |
    claude -p "Review PR changes" --output-format json > review.json

# GitLab CI 示例
code_review:
  script:
    - claude -p "审查代码" > review.json
```

---

## 常用示例

### 快速解释

```bash
claude -p "解释这个 Python 函数: $(cat example.py)"
```

### 代码审查

```bash
claude -p "审查这个 PR 的代码质量" --output-format json
```

### Bug 定位

```bash
cat error.log | claude -p "帮我定位这个错误的原因"
```

### 生成文档

```bash
claude -p "为这个 API 生成 OpenAPI 文档"
```

### 重构建议

```bash
claude -p "分析这个类的设计问题并提出改进建议"
```

---

## 快捷键（交互模式）

| 快捷键 | 功能 |
|--------|------|
| `Tab` | 自动补全 |
| `↑/↓` | 命令历史 |
| `Ctrl + C` | 取消当前输入 |
| `Ctrl + D` | 退出 REPL |
| `Ctrl + L` | 清屏 |
| `Esc + Esc` | 打开检查点 |
| `Ctrl + G` | 在编辑器打开 |
| `Alt + T` | 切换思考模式 |
| `Shift + Tab` | 切换权限模式 |

---

## 配置文件

Claude Code 配置文件位置：

| 位置 | 作用域 |
|------|--------|
| `~/.claude/settings.json` | 用户级 |
| `.claude/settings.json` | 项目级 |
| `.claude/settings.local.json` | 本地覆盖 |

---

## 下一步

- 了解高级功能：[Advanced Features](14-advanced-features.md)
- 配置 Hooks：[Hooks 系统](../L4-精通技巧/08-hooks.md)
- 权限管理：[权限配置](../L4-精通技巧/11-permissions.md)
