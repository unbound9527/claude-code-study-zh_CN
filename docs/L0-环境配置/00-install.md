# Claude Code 安装教程

## 本章目标

- 了解 Claude Code 的不同使用方式
- 掌握在各平台安装 Claude Code 的方法
- 完成账号注册和基础配置

---

## Claude Code 有几种用法？

| 使用方式 | 说明 | 推荐场景 |
|---------|------|---------|
| **网页版** | 直接在浏览器使用 | 快速体验、临时使用 |
| **桌面应用** | 下载安装独立应用 | 日常使用、深度工作 |
| **CLI 终端** | 在终端中通过命令使用 | 开发者、自动化任务 |

**推荐**：日常使用建议安装**桌面应用**，可以获得完整功能和更好的体验。

---

## 方式一：网页版（无需安装）

### 使用步骤

1. 打开浏览器，访问 **claude.ai/code**
2. 点击右上角 **"Log in"** 登录，或 **"Sign up"** 注册
3. 使用 Google、Apple 或邮箱账号注册
4. 登录后即可开始对话

### 优缺点

| 优点 | 缺点 |
|------|------|
| 无需安装 | 功能相对桌面版有限 |
| 任何设备都能用 | 依赖网络 |
| 随时可用 | 无本地文件访问能力 |

---

## 方式二：桌面应用（推荐）

### macOS

#### 系统要求

- macOS 11 (Big Sur) 或更高版本
- Apple Silicon (M1/M2/M3) 或 Intel 芯片

#### 安装步骤

**方法一：通过官网下载**

1. 访问 **claude.ai/code**
2. 点击 **"Download for Mac"**
3. 下载 `.dmg` 安装包
4. 打开下载的 `.dmg` 文件
5. 将 **Claude** 拖入 **Applications** 文件夹
6. 首次打开时，右键点击并选择"打开"，确认运行

**方法二：通过 Homebrew**

```bash
brew install --cask claude
```

### Windows

#### 系统要求

- Windows 10 或 Windows 11
- 至少 4GB 内存

#### 安装步骤

1. 访问 **claude.ai/code**
2. 点击 **"Download for Windows"**
3. 下载 `.exe` 安装包
4. 双击运行安装程序
5. 按提示完成安装
6. 首次打开可能需要允许网络连接

### Linux

#### 系统要求

- Ubuntu 20.04 或更高版本
- 其他基于 Debian 的发行版也应该支持

#### 安装步骤

**方法一：通过官网下载**

1. 访问 **claude.ai/code**
2. 点击 **"Download for Linux"**
3. 下载 `.deb`（Debian/Ubuntu）或 `.AppImage` 文件

**方法二：安装 .deb 包（Ubuntu/Debian）**

```bash
# 下载 deb 包后
sudo dpkg -i claude-desktop-app.deb
```

**方法三：通过 npm 安装**

```bash
npm install -g @anthropic-ai/claude-code
```

---

## 方式三：CLI 终端版本

Claude Code 本质上是一个 CLI 工具，通过终端运行。

### npm 全局安装

```bash
# 需要先安装 Node.js (https://nodejs.org/)
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version

# 启动
claude
```

### 其他安装方式

| 包管理器 | 命令 |
|---------|------|
| **npm** | `npm install -g @anthropic-ai/claude-code` |
| **yarn** | `yarn global add @anthropic-ai/claude-code` |
| **pnpm** | `pnpm add -g @anthropic-ai/claude-code` |

### CLI 基本命令

```bash
# 启动交互式对话
claude

# 直接执行单个命令
claude "帮我解释这段代码"

# 使用特定模型
claude --model opus "分析这个文件"

# 查看帮助
claude --help
```

---

## 账号注册与登录

### 注册 Anthropic 账号

1. 访问 **console.anthropic.com**
2. 点击 **"Sign Up"**
3. 填写邮箱、设置密码
4. 验证邮箱
5. （可选）绑定信用卡获取更多额度

### 登录 Claude Code

1. 打开 Claude Code 应用或网页版
2. 点击 **"Log in"**
3. 选择登录方式（网页版或应用内登录）

### 关于免费额度

Claude Code 有免费使用额度：

| 版本 | 额度 | 说明 |
|------|------|------|
| 免费版 | 一定数量消息 | 适合学习和体验 |
| Pro 版 | 无限消息 | $20/月，适合深度用户 |
| Team 版 | 团队共享 | 适合团队协作 |

> **提示**：免费额度用完后，需要升级 Pro 才能继续使用。如果只是学习用途，免费额度通常足够。

---

## 首次配置

### 设置偏好

1. 点击左下角 **设置图标**（⚙️）
2. 选择 **"Preferences"** 或 **"Settings"**
3. 可以设置：
   - **主题**：浅色 / 深色 / 跟随系统
   - **字体大小**：根据喜好调整
   - **快捷键**：查看和自定义常用快捷键

### 连接项目文件夹

Claude Code 的强大之处在于可以访问你的项目文件：

1. 点击左上角 **"Open Project"** 或 **"打开项目"**
2. 选择要工作的文件夹
3. 授权访问权限
4. 即可让 AI 读写你的项目文件

```
推荐：把日常工作文件夹加入 Claude Code，随时获得 AI 辅助
```

---

## 常见问题

### Q: 下载速度很慢怎么办？

A: 可以使用镜像或代理。如果在国内，可以尝试科学上网。

### Q: 安装后打不开？

A: 尝试以下步骤：
1. 重启电脑
2. 检查系统是否满足最低要求
3. 重新下载安装包
4. 查看是否被杀毒软件阻止

### Q: 如何更新到最新版本？

A: Claude Code 通常会自动更新。也可以手动：
- **macOS**：在 App Store 中检查更新
- **Windows**：重新下载安装包覆盖安装
- **CLI**：重新运行安装命令

### Q: 卸载后对话记录还在吗？

A: 网页版和云端同步的对话记录不受影响。本地存储的记录会在卸载后删除。

---

## 跟做

1. 选择适合你的安装方式（网页版或桌面应用）
2. 完成注册登录
3. 打开 Claude Code，熟悉界面
4. （可选）连接一个工作文件夹

---

## 下一步

- [什么是 AI](01-what-is-ai.md)
- [第一次对话](02-first-chat.md)
