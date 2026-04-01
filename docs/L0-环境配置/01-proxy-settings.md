# 代理配置与网络设置

## 本章目标

- 掌握 Claude Code 的代理配置方法
- 了解 HTTP/SOCKS 代理设置
- 解决网络连接问题

---

## 为什么需要配置代理？

在国内使用 Claude Code 时，由于网络原因可能无法直接连接到 Anthropic API。配置代理可以：
- 解决 API 连接问题
- 提升响应速度
- 避免网络超时

---

## 代理类型

### HTTP 代理

最常见的代理类型，用于 HTTP 请求转发。

```bash
# 设置 HTTP 代理
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"

# 或使用别名
export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"
```

### SOCKS5 代理

支持更多协议，适合需要 UDP 的场景。

```bash
# 设置 SOCKS5 代理
export SOCKS_PROXY="socks5://127.0.0.1:1080"
export ALL_PROXY="socks5://127.0.0.1:1080"
```

---

## Claude Code 代理配置

### 方法一：环境变量（推荐）

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"

# 生效
source ~/.bashrc
```

### 方法二：CLI 参数

```bash
# 启动时指定代理
claude --proxy http://127.0.0.1:7890

# 或
claude --env HTTP_PROXY=http://127.0.0.1:7890
```

### 方法三：配置文件

在 `~/.claude/settings.json` 中设置：

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

| 软件 | 协议 | 端口 | 适用平台 |
|------|------|------|----------|
| **Clash** | HTTP/SOCKS5 | 7890 | Windows/Mac/Linux |
| **Shadowsocks** | SOCKS5 | 1080 | 全平台 |
| **V2Ray** | VMess/Trojan | 1080/8080 | 全平台 |
| **Surge** | HTTP/SOCKS5 | 6153 | Mac/iOS |

---

## 网络问题排查

### 检查代理是否生效

```bash
# 测试代理连通性
curl -x http://127.0.0.1:7890 https://api.anthropic.com

# 查看当前代理设置
echo $HTTP_PROXY
echo $HTTPS_PROXY
```

### 常见问题

| 问题 | 解决方案 |
|------|----------|
| 代理后仍无法连接 | 检查代理软件是否正常运行 |
| 认证失败 | 确认用户名密码正确 |
| 端口被占用 | 更换代理端口 |
| 代理链过长 | 简化代理路径 |

### 验证 API 连接

```bash
# 测试 Anthropic API
curl -x http://127.0.0.1:7890 \
  -H "x-api-key: YOUR_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-opus-4-6","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}' \
  https://api.anthropic.com/v1/messages
```

---

## 跟做

1. 确认本地代理软件正常运行
2. 设置环境变量 `HTTP_PROXY` 和 `HTTPS_PROXY`
3. 测试代理是否生效
4. 验证 Claude Code 能否正常连接

---

## 下一步

- [接入国产化模型](02-domestic-models.md)
