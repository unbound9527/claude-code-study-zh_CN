# 接入国产化模型

## 本章目标

- 了解国内主流 AI 模型服务商
- 掌握通过 MCP 接入国产模型的方法
- 理解各平台的收费模式

---

## 为什么接入国产模型？

- **合规需求**：满足国内数据合规要求
- **成本优化**：国产模型价格通常更优惠
- **响应速度**：国内节点延迟更低
- **服务稳定**：避免跨境网络波动

---

## 国内主流模型服务商

### 收费模式概览

| 服务商 | 代表模型 | 计费方式 | 价格水平 |
|--------|----------|----------|----------|
| **智谱 AI** | GLM-4/GLM-4V | 按 token 计费 | ★★☆☆☆ |
| **百度文心** | ERNIE 4.0/ERNIE Bot | 按 token 计费 | ★★☆☆☆ |
| **阿里通义** | Qwen 2.5/Qwen-VL | 按 token 计费 | ★★☆☆☆ |
| **讯飞星火** | Spark 4.0 | 按调用量计费 | ★★★☆☆ |
| **腾讯混元** | Hunyuan | 按 token 计费 | ★★☆☆☆ |
| **字节豆包** | Doubao | 按 token 计费 | ★☆☆☆☆ |
| **Kimi (月之暗面)** | Moonshot | 按 token 计费 | ★★☆☆☆ |
| **MiniMax** | abab 6.5 | 按 token 计费 | ★☆☆☆☆ |
| **DeepSeek** | DeepSeek V3/DeepSeek Coder | 按 token 计费 | ★☆☆☆☆ |

---

## 各平台详细说明

### 1. 智谱 AI (GLM)

**官网**：https://www.zhipuai.cn/

**定价示例**：
| 模型 | 输入 | 输出 |
|------|------|------|
| GLM-4 | ¥0.001/千token | ¥0.003/千token |
| GLM-4V | ¥0.006/千token | ¥0.006/千token |

**特点**：
- 中文理解能力强
- 性价比高
- 提供 API 和 SDK

**MCP 接入**：
```bash
npm install -g @modelcontextprotocol/server-zhipuai
```

---

### 2. 百度文心一言 (ERNIE)

**官网**：https://cloud.baidu.com/product/wenxin.html

**定价示例**：
| 模型 | 输入 | 输出 |
|------|------|------|
| ERNIE-4.0 | ¥0.12/千token | ¥0.12/千token |
| ERNIE-Speed | ¥0.004/千token | ¥0.008/千token |

**特点**：
- 百度自研文心大模型
- 强大的中文创作能力
- 丰富的 API 接口

**MCP 接入**：
```bash
npm install -g @modelcontextprotocol/server-baidu
```

---

### 3. 阿里通义千问 (Qwen)

**官网**：https://tongyi.aliyun.com/

**定价示例**：
| 模型 | 输入 | 输出 |
|------|------|------|
| Qwen-2.5-72B | ¥0.004/千token | ¥0.012/千token |
| Qwen-VL | ¥0.008/千token | ¥0.008/千token |

**特点**：
- 开源模型表现优秀
- 阿里云生态集成好
- 多模态能力强

**MCP 接入**：
```bash
npm install -g @modelcontextprotocol/server-qwen
```

---

### 4. DeepSeek

**官网**：https://www.deepseek.com/

**定价示例**：
| 模型 | 输入 | 输出 |
|------|------|------|
| DeepSeek V3 | ¥0.001/千token | ¥0.002/千token |
| DeepSeek Coder | ¥0.001/千token | ¥0.003/千token |

**特点**：
- 价格最低
- 代码能力突出
- 专注于 AI 基础设施

**MCP 接入**：
```bash
npm install -g @modelcontextprotocol/server-deepseek
```

---

### 5. Kimi (月之暗面)

**官网**：https://kimi.moonshot.cn/

**定价示例**：
| 模型 | 输入 | 输出 |
|------|------|------|
| Moonshot V1-8K | ¥0.012/千token | ¥0.012/千token |
| Moonshot V1-32K | ¥0.024/千token | ¥0.024/千token |

**特点**：
- 支持超长上下文（128K）
- 联网搜索能力强
- 用户体验好

---

### 6. 腾讯混元

**官网**：https://cloud.tencent.com/product/hunyuan

**定价示例**：
| 模型 | 输入 | 输出 |
|------|------|------|
| Hunyuan-pro | ¥0.1/千token | ¥0.1/千token |
| Hunyuan-standard | ¥0.006/千token | ¥0.006/千token |

**特点**：
- 腾讯生态集成
- 稳定性好
- 多模态支持

---

## 通过 MCP 接入国产模型

### 通用配置流程

1. **获取 API Key**
   - 注册对应平台账号
   - 开通 API 服务
   - 获取 Secret Key

2. **安装 MCP Server**
   ```bash
   npm install -g @modelcontextprotocol/server-xxx
   ```

3. **配置 settings.json**
   ```json
   {
     "mcpServers": {
       "domestic-model": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-xxx"],
         "env": {
           "API_KEY": "your-api-key"
         }
       }
     }
   }
   ```

---

## 开源 MCP 服务器推荐

### 官方 MCP Servers
- [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) ⭐ 3.2k+
  - 官方维护的 MCP 服务器集合
  - 包括 filesystem、git、github、postgres 等

### 社区 MCP Servers
- [MCP-Servers](https://github.com/cline/mcp-servers) ⭐ 1k+
  - Cline 社区维护的服务器集合

### 国产 MCP 生态
- [MCP-Go](https://github.com/ModelContextProtocol/mcp-go) ⭐ 200+
  - Go 语言实现的 MCP 服务器开发框架

---

## 成本优化建议

### 1. 选择合适的模型

| 场景 | 推荐模型 | 原因 |
|------|----------|------|
| 简单问答 | DeepSeek | 价格最低 |
| 日常开发 | 智谱/通义 | 性价比平衡 |
| 复杂推理 | Claude/GPT | 能力更强 |
| 长文档 | Kimi | 超长上下文 |

### 2. 缓存策略

```json
{
  "cache": {
    "enabled": true,
    "ttl": 3600,
    "maxSize": "100MB"
  }
}
```

### 3. 批量处理

合并多个请求减少 API 调用次数。

---

## 合规注意事项

- **数据安全**：确认模型服务商的数据处理政策
- **内容审核**：了解平台的内容过滤机制
- **使用限制**：遵守各平台的 Rate Limit

---

## 常见问题

| 问题 | 解答 |
|------|------|
| API 调用失败？ | 检查 API Key 是否正确，网络是否通畅 |
| 响应速度慢？ | 考虑使用国内节点或更换服务商 |
| 费用超预算？ | 设置用量告警，使用缓存 |

---

## 跟做

1. 注册一个国产模型平台账号
2. 申请 API Key
3. 配置 MCP Server
4. 测试调用

---

## 下一步

- [国产 MCP Server 搭建](../L5-工程化学习/03-MCP协议：扩展Agent能力.md)
