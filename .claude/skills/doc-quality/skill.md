# 文档质量检查 Skill

## 用途

检查 Markdown 文档的质量问题，包括：
- 断链检测（链接目标不存在）
- 标题层级一致性
- 格式规范检查

## 相关开源项目

| 项目 | Stars | 说明 |
|------|-------|------|
| [markdownlint](https://github.com/DavidAnson/markdownlint) | ⭐ 5k+ | Markdown 格式检查工具 |
| [textlint](https://github.com/textlint/textlint) | ⭐ 3k+ | 可配置的文本检查工具 |
| [AlexJS](https://github.com/get/alex) | ⭐ 5k+ | 写作风格检查工具 |
| [write-good](https://github.com/btford/write-good) | ⭐ 4k+ | 英文写作检查工具 |

## 使用方式

```
/quality [文件路径或目录]
```

不指定路径时，默认检查 `docs/` 目录。

## 检查项目

### 1. 链接检查
- 验证所有 `.md` 链接目标存在
- 检查外部 URL 格式有效性
- 报告断链列表

### 2. 标题层级检查
- 验证标题从 H1 开始
- 检测跳过层级的情况（如 H1 后直接 H3）
- 检查是否有多余的 H1

### 3. 格式检查
- 检查分割线 `---` 使用
- 验证列表符号一致性
- 检查代码块标记

## 实现方式

使用 Agent Teams 并行执行三个子检查：
- link-checker - 链接检查
- heading-analyzer - 标题分析
- format-validator - 格式验证

最后由 reporter 汇总结果。

## 输出格式

```
## 文档质量检查报告

### 链接检查
- 状态：通过 / 失败
- 问题链接：N 个
  - [源文件] [链接] → 问题描述

### 标题层级
- 状态：通过 / 失败
- 问题文件：N 个
  - [文件] 行 N → 问题描述

### 格式规范
- 状态：通过 / 失败
- 问题文件：N 个
  - [文件] 行 N → 问题描述

---
总计：M 个问题
```
