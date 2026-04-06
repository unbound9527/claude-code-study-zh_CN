# Claude Code 创始人 Boris Cherny 分享的隐藏功能

> 来源：[Boris Cherny (@bcherny) Twitter 线程](https://x.com/bcherny/status/2038454336355999749)
> 发布于 2026年3月30日

Boris Cherny 是 Claude Code 项目的创始人，他在 Twitter 上分享了一系列他最常使用但容易被忽略的功能。

---

## 1. 移动端应用

Did you know Claude Code has a mobile app? Personally, I write a lot of my code from the iOS app. It's a convenient way to make changes without opening a laptop.

Claude Code 有移动端应用，Boris 本人经常使用 iOS 应用编写代码，无需打开笔记本电脑即可便捷地修改代码。

**使用方式**：下载 Claude iOS/Android 应用，点击左侧 Code 标签。

---

## 2. 跨设备会话迁移

Move sessions back and forth between mobile/web/desktop and terminal. Run `claude --teleport` or /teleport to continue a cloud session on your machine. Or run /remote-control to control a locally running session from your phone/web.

在移动端/Web/桌面端和终端之间移动会话。使用 `claude --teleport` 或 `/teleport` 在本地继续云端会话，或使用 `/remote-control` 从手机/Web 控制本地运行的会话。

---

## 3. /loop 和 /schedule 定时任务

Two of the most powerful features in Claude Code: /loop and /schedule. Use these to schedule Claude to run automatically at a set interval, for up to a week at a time.

/loop 和 /schedule 是 Claude Code 最强大的功能之一。可以用它们来安排 Claude 按设定的时间间隔自动运行，最长可达一周。

**Boris 的用法**：
- `/loop 5m /babysit` - 自动处理代码审查、自动 rebase 等

---

## 4. Hooks 钩子机制

Use hooks to deterministically run logic as part of the agent lifecycle. For example, use hooks to:
- Dynamically load in context each time you start Claude (SessionStart)
- Log every bash command the model runs (PreToolUse)
- Route permission prompts to WhatsApp

使用 hooks 在代理生命周期中运行确定性逻辑。例如：
- 每次启动 Claude 时动态加载上下文 (SessionStart)
- 记录模型运行的每条 bash 命令 (PreToolUse)
- 将权限提示路由到 WhatsApp

---

## 5. Cowork Dispatch

I use Dispatch every day to catch up on Slack and emails, manage files, and do things on my laptop when I'm not at a computer. When I'm not coding, I'm dispatching.

Boris 每天使用 Dispatch 来处理 Slack 和电子邮件、管理文件，以及不在电脑前时操作笔记本电脑。

Dispatch 是 Claude Desktop 应用的安全远程控制功能，可以使用 MCP、浏览器等。

---

## 6. Chrome 扩展

The most important tip for using Claude Code is: give Claude a way to verify its output. Once you do that, Claude will iterate until the result is great.

使用 Chrome 扩展进行前端工作时，最重要的技巧是：**给 Claude 一种验证其输出的方式**。一旦做到这一点，Claude 会迭代直到结果很棒。

把这想象成任何其他工程师：如果你让人建网站却从不检查结果，你期望得到什么？

---

## 7. Desktop 应用自动测试

Use the Claude Desktop app to have Claude automatically start and test web servers. The Desktop app bundles in the ability for Claude to automatically run your web server and even test it in a built-in browser.

使用 Claude Desktop 应用让 Claude 自动启动和测试 Web 服务器。Desktop 应用打包了让 Claude 自动运行 Web 服务器甚至在内置浏览器中测试它的能力。

---

## 8. Fork 会话

People often ask how to fork an existing session. Two ways:
1. Run /branch from your session
2. From the CLI, run `claude --resume <session-id> --fork-session`

如何 fork 现有会话：
- 在会话中运行 `/branch`
- 从 CLI 运行 `claude --resume <session-id> --fork-session`

---

## 9. /btw 副查询

Use /btw for side queries. I use this all the time to answer quick questions while the agent works.

使用 `/btw` 进行副查询。当代理工作时，随时用它来快速回答问题。

---

## 10. Git Worktrees

Claude Code ships with deep support for git worktrees. Worktrees are essential for doing lots of parallel work in the same repository. I have dozens of Claudes running at all times, and this is how I do it.

Claude Code 深度支持 git worktrees。Worktrees 对于在同一仓库中进行大量并行工作至关重要。Boris 始终运行着数十个 Claude，这就是他的方法。

**使用方式**：`claude -w` 在新的 worktree 中启动会话

---

## 11. /batch 批量处理

/-batch interviews you, then has Claude fan out the work to as many worktree agents as it takes (dozens, hundreds, even thousands) to get it done. Use it for large code migrations and other kinds of parallelizable work.

/batch 会先询问你，然后让 Claude 将工作分派给尽可能多的 worktree 代理（数十个、数百个甚至数千个）来完成。用于大型代码迁移和其他可并行的工作。

---

## 12. --bare 加速 SDK 启动

By default, when you run claude -p (or the TypeScript or Python SDKs) we search for local CLAUDE.md's, settings, and MCPs. But for non-interactive usage, most of the time you want to explicitly specify what to load.

默认情况下，运行 `claude -p`（或 TypeScript/Python SDK）时会搜索本地 CLAUDE.md、设置和 MCP。但对于非交互式使用，通常需要明确指定要加载的内容。

使用 `--bare` 可以将 SDK 启动速度加快高达 10 倍。

---

## 13. --add-dir 访问更多文件夹

When working across multiple repositories, I usually start Claude in one repo and use --add-dir (or /add-dir) to let Claude see the other repo.

在多个仓库之间工作时，Boris 通常在一个仓库中启动 Claude，然后使用 `--add-dir`（或 `/add-dir`）让 Claude 访问其他仓库。这不仅告诉 Claude 关于仓库的信息，还给它编辑权限。

---

## 14. --agent 自定义代理

Use --agent to give Claude Code a custom system prompt & tools. Custom agents are a powerful primitive that often gets overlooked.

使用 `--agent` 为 Claude Code 提供自定义系统提示和工具。自定义代理是一个经常被忽视的强大原语。

**使用方式**：在 `.claude/agents` 中定义新代理，然后运行 `claude --agent=<your agent's name>`

---

## 15. /voice 语音输入

Fun fact: I do most of my coding by speaking to Claude, rather than typing.

有趣的事实：Boris 大部分代码是通过对 Claude 说话而不是打字来完成的。

**使用方式**：
- 在 CLI 中运行 `/voice`，然后按住空格键
- 在 Desktop 上按下语音按钮
- 在 iOS 设置中启用听写

---

## 总结

这些是 Boris Cherny 在日常工作中最依赖的功能，涵盖了从跨设备协作、定时任务、到语音输入等多个方面。掌握这些功能可以显著提升使用 Claude Code 的效率。

原文链接：https://x.com/bcherny/status/2038454336355999749
