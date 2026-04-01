# Claude Code 工具详解

## 工具 = AI 的手

不只能说话，还能**执行操作**：

```
你说："读取这个文件"
→ Claude Code 使用 [Read]
→ 读取文件
→ 分析后回复
```

## 内置工具

| 工具 | 作用 | 示例 |
|------|------|------|
| **Read** | 读取文件内容 | "看 src/app.js" |
| **Write** | 创建/写入文件 | "创建 index.html" |
| **Edit** | 修改部分内容 | "改第三行" |
| **Bash** | 执行命令 | "运行 npm install" |
| **Glob** | 搜索文件 | "找所有 .js 文件" |
| **Grep** | 搜索内容 | "找含 'login' 的文件" |
| **WebFetch** | 获取网页 | "看这个网页内容" |
| **Agent** | 启动子代理 | "同时处理三件事" |

## Read

让 AI 看文件内容：

```
"看 src/utils.js"
"读这份文档"
"看 App.css 第10-20行"
```

## Write

创建新文件：

```
"创建 index.html"
"保存到 src/app.js"
```

## Edit

精确修改，不覆盖整个文件：

```
"改第三行 hello → world"
"加错误处理"
"删第五段重复"
```

## Bash

运行命令：

```
"npm install"
"python main.py"
"运行测试"
```

⚠️ 危险操作（删文件、git push）需授权。

## Glob / Grep

- **Glob**：找特定类型文件 `"src下所有组件"`
- **Grep**：找文件内容 `"找含 useState 的地方"`

## WebFetch

获取网页：

```
"看 https://example.com"
"获取网页主要信息"
```

## Agent

并行处理：

```
"同时：1.写代码 2.写测试 3.写文档"
```

## 主动指定工具

```
"用 Read 看这个文件"
"用 Bash 运行脚本"
```

不指定时 AI 自动选择。

## 练一练

1. 让 AI 读一个文件
2. 让 AI 创一个文件
3. 让 AI 用 Grep 搜关键词
