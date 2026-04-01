# 内容同步 Skill

## 用途

同步和维护文档内容的一致性：
- 自动更新 README.md 目录索引
- 修复和验证交叉引用链接
- 保持文档间引用关系最新

## 相关开源项目

| 项目 | Stars | 说明 |
|------|-------|------|
| [VuePress](https://github.com/vuejs/vuepress) | ⭐ 23k+ | Vue 官方静态网站生成器 |
| [VitePress](https://github.com/vuejs/vitepress) | ⭐ 12k+ | Vite 驱动的静态网站生成器 |
| [docsify](https://github.com/docsifyjs/docsify) | ⭐ 23k+ | 轻量级文档网站生成器 |
| [GitBook](https://github.com/GitbookIO/gitbook) | ⭐ 35k+ | 现代文档平台 |

## 使用方式

```
/sync-readme        # 更新 README 目录
/check-crossref    # 检查交叉引用
/fix-links [文件]  # 自动修复指定文件的断链
```

## 功能说明

### README 更新 (/sync-readme)

扫描 `docs/` 目录结构，生成最新目录并更新 `README.md`：

1. 遍历所有章节目录（L1-L6 + 附录）
2. 按目录结构生成导航
3. 检测新增/删除的文档
4. 更新 README 中的"快速导航"部分

### 交叉引用检查 (/check-crossref)

验证文档间的相互引用是否有效：

1. 提取所有跨文档链接
2. 验证链接目标存在
3. 报告无效引用

### 断链修复 (/fix-links)

自动修复断链：
1. 对于移动/重命名的文件，更新所有引用
2. 对于删除的文件，标记为待处理
3. 生成修复报告

---

## 实现架构

使用 Agent Teams 并行处理：

```
content-sync Team
├── Scanner Agent     → 读取目录结构
├── Link Checker     → 验证链接有效性
├── README Updater   → 更新 README
└── Reporter         → 生成报告
```

## 输出格式

### /sync-readme 输出

```markdown
## README 同步报告

**检测到目录**:
- L1-认识工具: N 个文档
- L2-掌握用法: N 个文档
- ...

**更新内容**:
- 新增文档: [列表]
- 删除文档: [列表]
- README 状态: 已更新 / 无需更新
```

### /check-crossref 输出

```markdown
## 交叉引用检查报告

**检查链接数**: N
**有效链接**: M
**断链数**: K

#### 断链详情

| 源文件 | 链接 | 建议修复 |
|--------|------|----------|
| xxx.md | [link](../L2/xx.md) | 文件已移动至... |
```
