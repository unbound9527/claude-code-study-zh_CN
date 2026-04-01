# 信息获取 Skill

## 用途

当用户提供外部文档链接时，自动提取有用信息并补充到项目中。

## 相关开源项目

| 项目 | Stars | 说明 |
|------|-------|------|
| [Playwright](https://github.com/microsoft/playwright) | ⭐ 65k+ | 浏览器自动化测试和爬虫 |
| [Puppeteer](https://github.com/puppeteer/puppeteer) | ⭐ 85k+ | Chrome 控制工具 |
| [Scrapy](https://github.com/scrapy/scrapy) | ⭐ 48k+ | Python 爬虫框架 |
| [Readability.js](https://github.com/mozilla/readability) | ⭐ 12k+ | 提取页面正文内容 |
| [Turndown](https://github.com/mixmark-io/turndown) | ⭐ 8k+ | HTML 转 Markdown 工具 |
| [jsdom](https://github.com/jsdom/jsdom) | ⭐ 20k+ | JavaScript DOM 实现 |

## 使用方式

```
/fetch-info <URL> [目标文档]
```

### 参数说明
- `URL`（必需）：要提取信息的网页链接
- `目标文档`（可选）：建议放置的目标位置，如 `docs/L4-精通技巧/07-mcp.md`

## 工作流程

### Step 1: 获取网页内容
- 使用 webFetch 工具获取页面 HTML
- 转换为 Markdown 格式

### Step 2: 分析内容类型
识别文档类型并决定处理方式：
- **官方文档** → 更新对应章节
- **博客教程** → 添加到附录或相关章节
- **示例代码** → 提取到"练一练"部分
- **新闻公告** → 生成更新报告

### Step 3: 内容转换
- HTML 转 Markdown
- 提取核心概念
- 整理示例代码
- 保持标题层级

### Step 4: 集成到项目
- 确定目标位置
- 按文档格式要求组织
- 插入交叉引用
- 更新相关章节的"下一步"

## 输出格式

```markdown
## 信息提取报告

**来源**: https://example.com/page
**类型**: 官方文档 / 博客教程 / 示例代码
**建议位置**: docs/L4-精通技巧/07-mcp.md

### 提取内容

[转换后的 Markdown 内容]

### 建议的交叉引用

- 本节引用自：[来源标题](URL)
- 相关章节：[链接列表]

### 操作确认

是否将此内容添加到项目？
- 输入 "是" 确认
- 输入 "否" 取消
- 输入 "修改" 调整目标位置
```

## 使用示例

```
/fetch-info https://docs.anthropic.com/claude-code/concepts/mcp
```

## 注意事项

- 保持原有信息的准确性
- 添加"本节参考"标注
- 遵守版权和使用条款
