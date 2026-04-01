# 构建发布 Skill

## 用途

自动化构建静态网站并发布。

## 相关开源项目

### 静态网站生成器

| 项目 | Stars | 说明 |
|------|-------|------|
| [VuePress](https://github.com/vuejs/vuepress) | ⭐ 23k+ | Vue 官方文档网站生成器 |
| [VitePress](https://github.com/vuejs/vitepress) | ⭐ 12k+ | Vite 驱动的静态网站生成器 |
| [docsify](https://github.com/docsifyjs/docsify) | ⭐ 23k+ | 轻量级文档网站生成器，零配置 |
| [GitBook](https://github.com/GitbookIO/gitbook) | ⭐ 35k+ | 现代文档和书籍出版平台 |
| [Hugo](https://github.com/gohugoio/hugo) | ⭐ 70k+ | 最快的静态网站生成器 |
| [Jekyll](https://github.com/jekyll/jekyll) | ⭐ 48k+ | GitHub Pages 默认生成器 |
| [Docusaurus](https://github.com/facebook/docusaurus) | ⭐ 55k+ | Facebook 的文档网站生成器 |

### 部署平台

| 项目 | 说明 |
|------|------|
| [Vercel](https://vercel.com/) | 极简部署平台 |
| [Netlify](https://www.netlify.com/) | 静态网站托管 |
| [GitHub Pages](https://pages.github.com/) | GitHub 原生托管 |

## 使用方式

```
/build        # 构建静态网站
/preview      # 本地预览
/deploy       # 部署到 GitHub Pages
```

## 功能说明

### /build - 构建网站

使用静态网站生成器（如 docsify、VuePress、GitBook）构建网站：
1. 检测项目中的配置文件
2. 安装依赖（如需要）
3. 执行构建命令
4. 输出到 `_site` 或 `dist` 目录

### /preview - 本地预览

启动本地服务器预览网站：
1. 检测网站根目录
2. 启动 HTTP 服务器
3. 输出预览地址（通常是 http://localhost:3000）

### /deploy - 部署

发布到 GitHub Pages：
1. 构建生产版本
2. 推送到 gh-pages 分支
3. 等待部署完成

## 当前项目状态

本项目目前是纯 Markdown 文档，建议使用以下方案之一：

| 方案 | 工具 | 特点 |
|------|------|------|
| 最简方案 | docsify | 零配置，直接加载 md 文件 |
| 文档方案 | VuePress | 支持侧边栏、搜索 |
| 专业方案 | GitBook | 完整书籍出版功能 |

## 推荐：docsify 快速部署

```bash
# 1. 安装 docsify
npm install -g docsify-cli

# 2. 初始化
docsify init ./docs

# 3. 本地预览
docsify serve docs

# 4. 构建（用于发布）
docsify build docs
```

## 输出格式

### /build 输出

```markdown
## 构建报告

**工具**: docsify
**状态**: 成功 / 失败
**输出目录**: _site/
**文件数**: N
**构建时间**: X 秒

[查看构建结果](#)
```

### /preview 输出

```markdown
## 本地预览

**地址**: http://localhost:3000
**按 Ctrl+C 停止预览**
```
