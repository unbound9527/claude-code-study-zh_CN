# Claude Code 完整教学：安装、代理配置与国产大模型切换
本文从**安装、代理配置、切换国产大模型**三方面，提供国内可用的完整步骤，覆盖 CLI 与 VSCode 插件，适配 Windows/macOS/Linux，让你在国内顺畅使用 Claude Code。

---

## 一、安装 Claude Code（CLI + VSCode 插件）
### 1.1 安装 CLI（命令行核心工具）
#### macOS/Linux
```bash
# 官方一键脚本（推荐）
curl -fsSL https://claude.ai/install.sh | bash

# 或 Homebrew
brew install --cask claude-code

# 验证
claude --version
```

#### Windows（PowerShell 管理员）
```powershell
# 官方一键安装
irm https://claude.ai/install.ps1 | iex

# 或 winget
winget install Anthropic.ClaudeCode

# 验证
claude --version
```

### 1.2 安装 VSCode 插件
1. 打开 VSCode → 扩展（Ctrl+Shift+X / Cmd+Shift+X）
2. 搜索 **Claude Code**（官方：Anthropic）
3. 点击 **Install**
4. 重启 VSCode 生效

---

## 二、代理配置
如需使用原生 Claude API，需要配置代理。

### 2.1 临时代理（当前终端生效）
```bash
# macOS/Linux
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export NO_PROXY="localhost,127.0.0.1"

# Windows PowerShell
$env:HTTP_PROXY="http://127.0.0.1:7890"
$env:HTTPS_PROXY="http://127.0.0.1:7890"
```

### 2.2 永久代理（推荐，写入配置文件）
#### macOS/Linux（~/.zshrc 或 ~/.bashrc）
```bash
echo 'export HTTP_PROXY=http://127.0.0.1:7890' >> ~/.zshrc
echo 'export HTTPS_PROXY=http://127.0.0.1:7890' >> ~/.zshrc
echo 'export NO_PROXY="localhost,127.0.0.1"' >> ~/.zshrc
source ~/.zshrc
```

#### Windows（永久环境变量）
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY","http://127.0.0.1:7890","User")
[Environment]::SetEnvironmentVariable("HTTPS_PROXY","http://127.0.0.1:7890","User")
```

### 2.3 Claude 配置文件代理（~/.claude/settings.json）
```json
{
  "env": {
    "HTTP_PROXY": "http://127.0.0.1:7890",
    "HTTPS_PROXY": "http://127.0.0.1:7890"
  }
}
```

---

## 三、切换国产大模型（兼容 Anthropic 协议）
Anthropic 明确将中国大陆列为不支持地区，直接访问 Claude API 会 403 / 超时。海外账号、共享账号、中转服务随时被封禁，数据与进度不可控。这里介绍几种国产大模型的配置方式。

### 3.1 配置文件路径
- macOS/Linux: `~/.claude/settings.json`
- Windows: `%USERPROFILE%\.claude\settings.json`

### 3.2 主流国产模型配置

**作者使用 49元/月 Token Plan 套餐**，整体能力逊色于Claude，但按调用次数计算，足够个人日常开发使用，复杂项目可作为辅助。通过作者分享的链接购买，可以享[9折](https://platform.minimaxi.com/subscribe/token-plan?code=Gnf6KDMG1h&source=link)。


#### 1）智谱 GLM（推荐）

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "你的智谱API Key",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

#### 2）通义千问（阿里）
```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "你的通义API Key",
    "ANTHROPIC_BASE_URL": "https://dashscope.aliyuncs.com/anthropic/v1",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "qwen-coder-plus"
  }
}
```

#### 3）DeepSeek
```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "你的DeepSeek API Key",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/v1",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-coder-33b-instruct"
  }
}
```

### 3.3 VSCode 插件配置
1. 打开设置（Ctrl+, / Cmd+,）
2. 搜索 `Claude Code: Environment Variables`
3. 点击 `编辑 in settings.json`
4. 填入：
```json
{
  "claude-code.environmentVariables": [
    {
      "name": "ANTHROPIC_AUTH_TOKEN",
      "value": "你的API Key"
    },
    {
      "name": "ANTHROPIC_BASE_URL",
      "value": "https://open.bigmodel.cn/api/anthropic"
    }
  ]
}
```

### 3.4 图形化工具：cc-switch（一键切换）
```bash
# 安装
npm install -g cc-switch

# 启动
cc-switch
```
- 界面添加国产模型 API 信息
- 一键切换，无需手动改 JSON


---

## 四、启动与验证
```bash
# 启动 CLI
claude

# 测试
> 写一个Python Hello World
```
出现代码即配置成功。

---

## 五、常见问题
1. **网络错误**：检查代理端口（默认 7890），确保代理开启“增强模式”
2. **认证失败**：API Key 正确、Base URL 带 `/v1` 或 `/anthropic`
3. **模型不响应**：关闭不必要遥测：`"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"`

---

## 六、总结
国内使用 Claude Code 三步：**安装 → 代理 → 换国产模型**。优先用 **智谱 GLM** 或 **通义千问**，稳定兼容、成本低。按本文配置即可零障碍使用 AI 编程助手。