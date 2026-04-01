# Claude Code 安装教程

## 两种使用方式

| 方式 | 说明 | 本项目推荐 |
|------|------|-----------|
| **CLI 终端** | 通过终端命令使用 | ✅ **推荐** |
| **桌面应用** | 安装独立软件 | 备选 |

> **本项目作者推荐 CLI 方式**：后续所有演示均基于 CLI，配置灵活、可自定义程度高。

---

## ⚠️ 重要注意事项

### 网络要求

**CLI 方式依赖网络**。Claude Code 官方服务器在海外，国内直接访问可能：
- 速度慢或不稳定
- 连接失败

**解决方案**：配置代理（详见 [代理设置](01-proxy-settings.md)）

### Anthropic 公司对华态度

Anthropic（Claude 母公司）**尚未向中国内地开放**：
- 中国大陆手机号无法注册
- 中国大陆 IP 可能被限制访问
- 信用卡支付可能受限

**国内用户建议**：
- 使用代理服务
- 准备海外手机号（如 Google Voice）或借用
- 或考虑接入国产模型（详见 [国产模型接入](02-domestic-models.md)）

### 想体验原生 Claude 模型？

**方式一：自行注册（推荐有条件的用户）**
- 准备海外手机号 + 支持海外支付的信用卡
- 访问 [console.anthropic.com](https://console.anthropic.com) 注册

**方式二：购买现成账号**
- **闲鱼/淘宝搜索**："Claude API Key" 或 "Anthropic 账号"
- 价格范围：¥10-50/月（视额度而定）
- **注意**：选择信誉好的卖家，注意账号安全

**本项目默认使用国产模型方案**，兼顾合规和成本

---

## 前置要求：Node.js 安装

CLI 版需要 Node.js。

### Windows

1. 下载 [nodejs.org](https://nodejs.org/) LTS 版
2. 双击安装（默认配置即可）
3. 验证：
```bash
node --version
npm --version
```

### macOS / Linux

```bash
# 推荐使用 nvm 管理版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
nvm use --lts
```

---

## 方式一：CLI 安装（推荐）

### 安装

```bash
npm install -g @anthropic-ai/claude-code
```

### 验证

```bash
claude --version
```

看到版本号即为安装成功（如 `claude 1.0.10`）

### 启动

```bash
claude
```

---

## 方式二：桌面应用

### macOS

1. 下载 [claude.ai/code](https://claude.ai/code)
2. 下载 `.dmg`，拖入 Applications
3. 首次运行需右键"打开"

### Windows

1. 下载 [claude.ai/code](https://claude.ai/code)
2. 运行 `.exe` 安装包

### Linux

```bash
# deb 包
sudo dpkg -i claude-desktop-app.deb

# 或 npm
npm install -g @anthropic-ai/claude-code
```

---

## 账号注册

1. 访问 [console.anthropic.com](https://console.anthropic.com)
2. Sign Up（需海外手机号验证）
3. 获取 API Key（CLI 使用）

### CLI 登录

```bash
claude
# 首次会提示输入 API Key
```

---

## 首次配置

### CLI 配置

```bash
# 配置代理（国内必须）
export HTTPS_PROXY=http://127.0.0.1:7890

# 设置 API Key
export ANTHROPIC_API_KEY=your-api-key
```

### 桌面应用配置

1. 打开应用 → Settings
2. 登录账号或输入 API Key
3. 选择主题（浅色/深色）
4. 打开项目文件夹

---

## 跟做

1. 安装 Node.js（验证 `node --version`）
2. 安装 Claude CLI：
```bash
npm install -g @anthropic-ai/claude-code
```
3. 验证版本：
```bash
claude --version
```
4. 尝试启动：
```bash
claude
```

> 如果连接失败，先看 [代理设置](01-proxy-settings.md)

---

## 下一步

- [代理设置](01-proxy-settings.md)
