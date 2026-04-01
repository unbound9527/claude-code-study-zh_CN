# Advanced Features：高级功能

Claude Code 的高级功能，释放 AI 的全部潜力。

---

## 规划模式（Planning Mode）

规划模式让你在执行前先创建详细计划，审阅后再执行。

### 激活方式

```
/plan 实现用户认证系统
```

或 CLI 模式：

```bash
claude --permission-mode plan
```

### 适用场景

**推荐使用**：
- 复杂的多文件重构
- 新功能实现
- 架构变更
- 数据库迁移

**不推荐**：
- 简单 Bug 修复
- 格式调整
- 单文件编辑

### 工作流程

```
1. 你：/plan [任务描述]
2. Claude：创建详细计划
3. 你：审阅计划（yes/no/修改）
4. Claude：执行计划
```

### 规划模式配置

```json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

---

## 扩展思考（Extended Thinking）

扩展思考让 Claude 在复杂问题上花更多时间推理。

### 激活方式

键盘快捷键：`Alt + T`（macOS: `Option + T`）

### 思考层级（仅 Opus 4.6）

| 层级 | 符号 | 适用场景 |
|------|------|----------|
| 低 | ○ | 简单查询 |
| 中 | ◐ | 普通任务 |
| 高 | ● | 复杂问题 |
| 最高 | ●+ | 深度推理（仅 Opus） |

### CLI 设置

```bash
# 设置思考 token 上限
export MAX_THINKING_TOKENS=16000

# 设置思考层级
export CLAUDE_CODE_EFFORT_LEVEL=high
```

---

## 自动模式（Auto Mode）

自动模式使用后台分类器在执行前审查每个操作。

### 启用方式

```bash
claude --enable-auto-mode
```

### 默认阻止的操作

| 操作 | 示例 |
|------|------|
| 管道安装 | `curl \| bash` |
| 敏感数据外传 | API 密钥经网络发送 |
| 生产部署 | 目标生产环境的部署命令 |
| 批量删除 | 大目录的 `rm -rf` |
| IAM 变更 | 权限和角色修改 |
| 强制推送 | `git push --force main` |

---

## 后台任务（Background Tasks）

后台任务让你在执行长时间操作时继续其他工作。

### 启动后台任务

```bash
# 在命令前加 &
claude "分析这个大型代码库" &

# 或使用 nohup
nohup claude "大规模重构" &
```

### 管理后台任务

```
/jobs        # 列出运行中的任务
/stop <id>   # 停止任务
```

---

## 权限模式（Permission Modes）

| 模式 | 说明 |
|------|------|
| `default` | 默认模式 |
| `acceptEdits` | 允许编辑，拒绝危险操作 |
| `plan` | 执行前需要计划审批 |
| `auto` | 自动模式，后台分类器审查 |
| `dontAsk` | 不询问，直接执行 |
| `bypassPermissions` | 绕过所有权限检查 |

### 切换权限模式

键盘快捷键：`Shift + Tab`

---

## 无头模式（Headless Mode）

无头模式用于 CI/CD 和脚本自动化。

### 基本用法

```bash
# 打印模式（单次查询）
claude -p "审查这段代码"

# 从文件读取
claude -p "$(cat code.txt)"

# JSON 输出
claude -p "列出函数" --output-format json
```

### CI/CD 示例

```yaml
# GitHub Actions
- name: Code Review
  run: |
    claude -p "Review PR changes" \
      --output-format json \
      > review.json
```

---

## 会话管理

| 命令 | 说明 |
|------|------|
| `claude -c` | 继续最近会话 |
| `claude -r "session" "query"` | 按名称恢复会话 |
| `claude --fork-session` | 复制会话探索 |

### 会话复制用途

- 在保留原会话的同时创建分支
- 并行尝试不同方案
- 测试破坏性变更

---

## 交互功能

### 键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `Tab` | 自动补全 |
| `↑/↓` | 命令历史 |
| `Esc + Esc` | 打开检查点 |
| `Ctrl + G` | 在外部编辑器打开计划 |
| `Alt + T` | 切换扩展思考 |
| `Shift + Tab` | 切换权限模式 |

### 多行输入

```
在 REPL 中输入多行：
1. 输入三引号 """
2. 输入内容
3. 闭合三引号 """
```

---

## 语音输入

### 激活方式

在支持的平台上，点击输入框的麦克风图标。

### 语言支持

支持 20 种语言的语音转文字（STT）。

### 使用技巧

- 说话清晰，语速适中
- 涉及代码时建议手动输入
- 嘈杂环境下可能识别不准

---

## Git Worktrees

在隔离的 worktree 分支上并行工作。

### 创建 Worktree

```bash
git worktree add feature/new-feature
cd ../new-feature
claude
```

### 优势

- 同一仓库的多个并行分支
- 无需切换主目录
- 独立的工作环境

---

## 沙箱模式（Sandboxing）

操作系统级隔离，限制文件和网络访问。

### 使用场景

- 测试未知代码
- 执行第三方脚本
- 隔离危险操作

### 配置

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": "readonly",
    "network": "none"
  }
}
```

---

## 企业托管设置

企业环境可通过以下方式部署配置：

| 方式 | 平台 |
|------|------|
| plist | macOS |
| Registry | Windows |
| 托管文件 | Linux |

### 配置示例

```json
{
  "managed": {
    "source": "https://config.example.com/claude.json",
    "updateInterval": 3600
  }
}
```

---

## 下一步

- 深入 Hooks：[Hooks 进阶](../L4-精通技巧/14-hooks-advanced.md)
- 查看 CLI 参考：[附录 CLI](13-advanced-cli.md)
- 了解 MCP：[MCP 协议](../L4-精通技巧/07-mcp.md)
