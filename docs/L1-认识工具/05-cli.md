# 什么是 CLI？命令行界面入门

## 本章目标

- 理解什么是 CLI（命令行界面）
- 区分 CLI 和 GUI（图形界面）
- 掌握基本的 CLI 操作
- 为使用 Claude Code 打下基础

---

## 什么是 CLI？

**CLI = Command Line Interface（命令行界面）**

CLI 是一种通过文本命令来操作计算机的方式。你输入文字命令，电脑执行相应操作。

### 生活中的比喻

| 界面类型 | 比喻 |
|----------|------|
| GUI | 点餐时看菜单图片，手指点击下单 |
| CLI | 点餐时说出命令："来一份宫保鸡丁，加辣" |

### Claude Code 是 CLI 工具

Claude Code 运行在**终端（Terminal）**里，你通过输入文字命令来和它交流。

---

## CLI vs GUI

| 特点 | CLI（命令行） | GUI（图形界面） |
|------|---------------|-----------------|
| 操作方式 | 打字输入命令 | 鼠标点击图标 |
| 速度 | 快（熟练后） | 慢（需要多次点击） |
| 学习曲线 | 陡（要记命令） | 平缓（直观可见） |
| 自动化 | 容易 | 困难 |
| 占用资源 | 少 | 多 |
| 远程操作 | 方便 | 需要额外设置 |

### 直观对比

**GUI（图形界面）：**
```
点击 "开始" → 点击 "设置" → 点击 "系统" → 点击 "关于"
```

**CLI（命令行）：**
```
输入：systeminfo
一步到位
```

---

## 为什么 Claude Code 使用 CLI？

### CLI 的优势

1. **精确**：命令可以指定非常详细的参数
2. **可编程**：容易写脚本自动化重复任务
3. **轻量**：不需要图形渲染，占用资源少
4. **强大**：很多系统功能只有 CLI 提供

### Claude Code 的 CLI 特点

```
你：帮我写一个计算器程序
Claude Code：直接生成代码文件
           ↓
           执行命令测试
           ↓
           运行结果展示
```

---

## 打开你的终端

### Windows

| 方法 | 操作 |
|------|------|
| 方法 1 | 按 `Win + R`，输入 `cmd`，回车 |
| 方法 2 | 按 `Win + X`，选择"终端" |
| 方法 3 | 在开始菜单搜索 "PowerShell" |

### Mac

| 方法 | 操作 |
|------|------|
| 方法 1 | 按 `Command + Space`，搜索 "Terminal" |
| 方法 2 | 在 Launchpad 中找到"终端" |

### Linux

| 方法 | 操作 |
|------|------|
| 方法 1 | 按 `Ctrl + Alt + T` |
| 方法 2 | 在应用菜单中找到"终端" |

---

## 基础命令

### Windows (CMD/PowerShell)

| 命令 | 作用 | 示例 |
|------|------|------|
| `dir` | 列出文件 | `dir` |
| `cd` | 切换目录 | `cd Documents` |
| `mkdir` | 创建文件夹 | `mkdir myproject` |
| `del` | 删除文件 | `del file.txt` |
| `type` | 查看文件内容 | `type readme.md` |

### Mac/Linux (bash/zsh)

| 命令 | 作用 | 示例 |
|------|------|------|
| `ls` | 列出文件 | `ls` |
| `cd` | 切换目录 | `cd Documents` |
| `mkdir` | 创建文件夹 | `mkdir myproject` |
| `rm` | 删除文件 | `rm file.txt` |
| `cat` | 查看文件内容 | `cat readme.md` |

---

## 跟做练习

### Windows 用户

1. 打开终端（Win + R → 输入 `cmd` → 回车）
2. 输入 `cd Desktop` 切换到桌面
3. 输入 `mkdir claude-test` 创建测试文件夹
4. 输入 `cd claude-test` 进入该文件夹
5. 输入 `dir` 确认当前目录

### Mac/Linux 用户

1. 打开终端（Command/Ctrl + Space → 搜索 "Terminal"）
2. 输入 `cd Desktop` 切换到桌面
3. 输入 `mkdir claude-test` 创建测试文件夹
4. 输入 `cd claude-test` 进入该文件夹
5. 输入 `ls` 确认当前目录

---

## 练一练

1. 在终端中创建一个新文件夹，命名为 `my-agent`
2. 进入该文件夹
3. 确认当前位置
4. （可选）尝试创建一个简单的文本文件

---

## 小测验

**1. CLI 是什么意思？**
- A. Computer Learning Interface
- B. Command Line Interface
- C. Code Line Interpreter

**2. GUI 和 CLI 的主要区别是？**
- A. CLI 只能用鼠标操作
- B. GUI 用图形图标，CLI 用文字命令
- C. 没有区别

**3. Claude Code 使用什么界面？**
- A. 图形界面（GUI）
- B. 命令行界面（CLI）
- C. 触摸屏界面

---

## 答案

1-B | 2-B | 3-B

---

---

## 本章小结

**L1 认识工具**帮你建立了对 AI 和 Claude Code 的基础认知：

| 核心概念 | 关键 takeaway |
|---------|--------------|
| AI 本质 | AI 是根据海量训练数据组织回答的"超级图书馆"，不是真正"理解" |
| Claude Code | 能对话、能用工具的 AI 助手，不只是聊天 |
| CLI 优势 | 命令行比图形界面更精确、更适合自动化 |
| 工具调用 | Claude Code 能通过 Read/Write/Bash 等工具"做事情" |

**本章重点理解**：
- AI 不是万能的，它有知识截止日期和上下文限制
- 但通过工具调用，这些局限可以被克服
- Claude Code 是你的"AI 代理"，要学会"指挥"它工作

---

## 下一步

- [第一次对话](02-first-chat.md)
