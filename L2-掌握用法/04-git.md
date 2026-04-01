# 什么是 Git？

## Git 是什么

版本控制系统，管理你的修改历史。不用再靠 `论文_v1.docx`、`论文_最终版.docx` 这种命名。当你痛苦于难以对修改进行追踪时，Git 是你的救星，属于Claude Code使用的必备软件。

## 核心概念

| 概念 | 说明 |
|------|------|
| **仓库** | 一个项目 = 一个仓库 |
| **提交（Commit）** | 拍快照，保存当前状态 |
| **分支（Branch）** | 从主线分叉 |
| **合并（Merge）** | 分叉合并回主线 |

## Git vs GitHub

| | 是什么 | 在哪 |
|--|--------|------|
| **Git** | 版本控制工具 | 本地 |
| **GitHub** | 存放仓库的网站 | 云端 |

## Claude Code 能帮你

| 操作 | 说明 |
|------|------|
| `git status` | 看改了什么 |
| `git commit` | 保存版本 |
| `git log` | 看历史 |
| `git branch` | 开分支 |
| `git merge` | 合并代码 |

## 常用命令

```bash
git init                    # 初始化仓库
git add .                   # 暂存修改
git commit -m "信息"         # 提交
git status                  # 查看状态
git push                    # 推送到云端
git pull                    # 拉取更新
```

---

**下一步**：[分解复杂任务](../L3-熟练运用/01-break-tasks.md)
