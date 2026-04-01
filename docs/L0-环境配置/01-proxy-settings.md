# 代理配置与网络设置

## 为什么需要代理

Claude Code 官方 API 服务器在海外，国内直连会：
- 连接超时
- 响应缓慢
- 完全无法访问

如果想使用原生的Claude Code API，配置代理是**国内用户必须完成**的步骤。如果使用国产大模型则跳至 [代理设置](02-domestic-models.md)

---

## 代理配置

### 环境变量（推荐）

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"

# 生效
source ~/.bashrc
```

### CLI 参数

```bash
# 启动时指定代理
claude --proxy http://127.0.0.1:7890
```

### 配置文件

`~/.claude/settings.json`：
```json
{
  "proxy": {
    "http": "http://127.0.0.1:7890",
    "https": "http://127.0.0.1:7890"
  }
}
```

---

## 常见代理软件

| 软件 | 端口 | 说明 |
|------|------|------|
| **Clash** | 7890 | 全平台，支持规则分流 |
| **Clash Verge** | 7890 | Clash 的现代化界面版 |
| **Shadowsocks** | 1080 | 轻量级 SOCKS5 代理 |
| **V2Ray** | 1080/8080 | 支持多种协议 |
| **Surge** | 6153 | Mac/iOS 专业代理 |

> **端口号仅供参考**，请以你实际配置的端口为准

---

## 验证配置

### 测试代理

```bash
curl -x http://127.0.0.1:7890 https://api.anthropic.com
```

### 测试 API

```bash
curl -x http://127.0.0.1:7890 \
  -H "x-api-key: YOUR_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-opus-4-6","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}' \
  https://api.anthropic.com/v1/messages
```

### 查看当前代理

```bash
echo $HTTP_PROXY
echo $HTTPS_PROXY
```

---

## 常见问题

| 问题 | 解决 |
|------|------|
| 代理后仍无法连接 | 确认代理软件正常运行 |
| 认证失败 | 检查用户名密码 |
| 端口被占用 | 更换代理端口 |
| clash 规则导致直连 | 添加 Anthropic 域名到规则 |

---

## 跟做

1. 确认本地代理软件正常运行
2. 设置环境变量：
```bash
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"
```
3. 测试代理：
```bash
curl -x http://127.0.0.1:7890 https://api.anthropic.com
```

---

## 下一步

- [接入国产化模型](02-domestic-models.md)
