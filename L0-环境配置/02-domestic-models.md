# 接入国产化模型

## 为什么用国产模型

- **合规**：满足国内数据合规
- **成本**：比 Claude API 便宜
- **稳定**：国内服务器，延迟低
- **便捷**：无需代理，即开即用

---

## MiniMax（作者自用推荐）

### 简介

MiniMax 是国内领先的 AI 模型服务商，提供高性价比的 API 服务。

**作者使用 49元/月 Token Plan 套餐**，整体能力逊色于Claude，但按调用次数计算，足够个人日常开发使用，复杂项目可作为辅助。

### 价格

| 套餐 | 价格 | 说明 |
|------|------|------|
| Token Plan | ¥49/月 | 🔥 **推荐**，含语音/音乐/视频/图片生成权益 |
| 标准 API | 按量计费 | 适合用量不固定的用户 |

### 特点

- 性价比高
- 国内直连，无需代理
- 支持多种模态（文本、语音、视频等）
- API 稳定

### 获取 API Key

1. 访问 [platform.minimax.com](https://platform.minimax.com) 注册账号
2. 购买[token plan](https://platform.minimaxi.com/subscribe/token-plan)，或通过作者分享的链接购买，可以享[9折](https://platform.minimaxi.com/subscribe/token-plan?code=Gnf6KDMG1h&source=link)
3. 进入账户管理 → Token Plan → 复制Token Plan Key

### 官网配置步骤

```bash
https://platform.minimaxi.com/docs/guides/text-ai-coding-tools
```

---

## 其他国产模型（简要）

| 服务商 | 代表模型 | 说明 |
|--------|----------|------|
| **DeepSeek** | DeepSeek V3 | 价格低，代码能力强 |
| **智谱 AI** | GLM-4 | 中文理解好 |
| **阿里通义** | Qwen 2.5 | 开源模型优秀 |
| **百度文心** | ERNIE 4.0 | 百度自研 |
| **Kimi** | Moonshot | 超长上下文 128K |

> 如需接入，可访问各平台官网获取 API Key
---

## 下一步

- [开始学习](../L1-认识工具/01-what-is-ai.md)
