# Claude Code 与 Git 协同实战：规范化版本控制指南
在 Claude Code 中高效使用 Git，核心是**分支隔离、原子提交、规范留痕、安全协作**。本文将从环境准备、核心操作、规范模板、高级技巧、安全避坑全流程讲解，帮你实现 AI 编码与版本管理的无缝衔接，让每一次改动都可追溯、可回滚、可评审。

---

## 一、前置准备（必做，避免踩坑）
### 1. 环境依赖安装
- 本地安装 Git（≥2.30），执行 `git --version` 验证，Windows 建议用 Git Bash 终端。
- 若需 GitHub 协作，安装 GitHub CLI（gh）并登录：`gh auth login`。
- 确认 Claude Code 已初始化项目目录：进入工作文件夹后，执行 `claude init` 初始化 AI 上下文。

### 2. 本地 Git 仓库初始化
```bash
# 新建项目并初始化 Git
mkdir my-project && cd my-project
git init  # 初始化本地仓库
git branch -M main  # 重命名默认分支为 main（规范）

# 关联远程仓库（可选，后续推送用）
git remote add origin https://github.com/your-username/your-repo.git
```

### 3. 核心规范（写入 .gitignore 与 CLAUDE.md）
#### .gitignore（必配，避免无效提交）
```
#  Claude 运行缓存
.claude/
claude.log
#  系统/IDE 缓存
.DS_Store
node_modules/
*.iml
*.vscode/
#  敏感信息
.env
secret.key
```

#### CLAUDE.md（跨会话 Git 规范，AI 自动读取）
```markdown
# Git 协作规范
1. 分支策略：main 为生产分支，所有功能/修复基于 main 新建 feature/bugfix 分支
2. 提交规范：遵循 Conventional Commits（feat/fix/docs/style/refactor/test/chore）
3. 提交粒度：原子提交（一个提交只做一件事），禁止一次提交多个不相关改动
4. 分支命名：feature/功能模块-简短描述；bugfix/问题模块-简短描述
5. 禁止操作：直接推送 main 分支、强制覆盖远程提交、提交敏感信息
```

---

## 二、核心 Git 操作（Claude Code 指令模板）
### 1. 分支管理（隔离改动，核心安全屏障）
| 场景 | Claude 指令 | 执行的 Git 命令 |
|------|-------------|------------------|
| 新建功能分支 | 基于 main 新建 feature/user-auth 分支，拉取最新代码 | `git checkout main && git pull origin main && git checkout -b feature/user-auth` |
| 新建修复分支 | 基于 main 新建 bugfix/login-crash 分支，处理登录崩溃问题 | `git checkout main && git pull origin main && git checkout -b bugfix/login-crash` |
| 切换分支 | 切换到 feature/user-profile 分支 | `git checkout feature/user-profile` |
| 同步最新代码 | 同步 main 分支最新改动到当前分支 | `git pull origin main` |
| 删除已合并分支 | 删除本地已合并到 main 的 feature/user-auth 分支 | `git branch -d feature/user-auth` |

### 2. 提交与暂存（原子化，便于回溯）
#### （1）常规提交（规范提交信息）
```
#  Claude 指令
查看当前改动，按 Conventional Commits 规范生成提交信息，提交到 feature/user-auth 分支
#  执行逻辑
git status  # 查看改动文件
git add .  # 暂存所有改动（可替换为具体文件）
git commit -m "feat(auth): 实现用户登录功能，支持手机号+密码校验"  # 规范提交
```

#### （2）暂存未完成改动（Stash，切换分支必备）
```
#  Claude 指令
暂存当前未完成的登录页改动，备注为 WIP: 登录页UI调整，之后切换到 main 分支处理紧急问题
#  执行逻辑
git stash push -m "WIP: 登录页UI调整"  # 暂存
git checkout main  # 切换分支
#  恢复暂存
git checkout feature/user-auth
git stash pop  # 恢复并删除暂存
```

### 3. 提交记录查看与对比（理解改动历史）
| 场景 | Claude 指令 | 执行的 Git 命令 |
|------|-------------|------------------|
| 查看提交历史 | 显示最近 10 次提交记录，简化为一行展示 | `git log --oneline -n 10` |
| 查看具体提交详情 | 查看 commit 为 abc123 的提交改动详情 | `git show abc123` |
| 对比分支差异 | 对比 feature/user-auth 与 main 分支的改动差异 | `git diff main...feature/user-auth` |
| 查看文件历史 | 查看 src/ui/LoginPage.kt 的所有提交记录 | `git log --oneline src/ui/LoginPage.kt` |

### 4. 远程协作（推送/拉取/PR，团队协作核心）
#### （1）推送分支到远程
```
#  Claude 指令
将 feature/user-auth 分支推送到远程仓库，关联远程分支
#  执行逻辑
git push -u origin feature/user-auth  # 首次推送关联远程
git push origin feature/user-auth  # 后续推送
```

#### （2）拉取远程更新
```
#  Claude 指令
拉取远程 main 分支最新代码，合并到当前分支
#  执行逻辑
git checkout main
git pull origin main
```

#### （3）创建 PR（代码评审，规范流程）
```
#  Claude 指令
基于 feature/user-auth 分支创建 PR，目标分支为 main，PR 标题遵循规范，描述包含改动文件、测试情况、关联 issue
#  执行逻辑
#  1. 准备 PR 信息
git diff --name-only main...feature/user-auth  # 改动文件
git log main..feature/user-auth --oneline  # 提交历史
#  2. 创建 PR（需 GitHub 集成）
gh pr create --base main --head feature/user-auth --title "feat(auth): 用户登录功能" --body "### 改动文件\n- src/auth/login.ts\n- src/auth/token.ts\n### 测试\n✅ 单元测试通过"
```

---

## 三、常见场景实战（直接复制指令）
### 场景 1：AI 新增功能（完整流程）
```
1. 新建分支：基于 main 新建 feature/goods-list 分支，拉取最新代码
2. 编写代码：让 Claude 实现商品列表页（Compose + Coil，Material3 规范）
3. 查看改动：检查 src/ui/GoodsList.kt 的改动，确认无敏感信息
4. 原子提交：按规范提交，备注为 feat(goods): 实现商品列表，支持分页+图片加载
5. 推送远程：推送 feature/goods-list 到远程，创建 PR 关联 #123  issue
```

### 场景 2：修复线上 Bug（紧急回滚流程）
```
1. 切回 main：切换到 main 分支，拉取最新代码
2. 新建修复分支：新建 bugfix/login-timeout 分支
3. 定位问题：查看最近提交记录，定位登录超时原因
4. 修复提交：修复代码，提交为 fix(auth): 处理登录超时异常，增加 10s 超时重试
5. 推送验证：推送分支到远程，创建 PR 合并到 main，验证修复效果
6. 清理分支：合并后删除 feature/login-timeout 本地/远程分支
```

### 场景 3：代码回滚与撤销（误操作补救）
| 误操作类型 | Claude 指令 | 安全补救方案 |
|------------|-------------|--------------|
| 撤销最后一次提交（保留改动） | 撤销最近 1 次提交，保留代码到工作区 | `git reset --soft HEAD~1` |
| 撤销最后一次提交（不保留改动） | 彻底回滚到上一次提交，删除当前提交 | `git reset --hard HEAD~1`（谨慎使用！） |
| 撤销文件改动 | 撤销 src/ui/LoginPage.kt 的本地改动，恢复到最近提交 | `git checkout -- src/ui/LoginPage.kt` |
| 撤销远程提交（已推送） | 撤销远程 feature/user-auth 分支的最后 2 次提交（需团队同意） | `git reset --hard HEAD~2 && git push -f origin feature/user-auth`（禁止强制推送 main 分支！） |

---

## 四、高级技巧（提升效率）
### 1. 自定义 Git 别名（简化指令）
在终端执行以下命令，将常用 Git 操作简化为短指令：
```bash
#  查看提交历史（图形化）
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
#  生成规范提交（结合 Claude）
git config --global alias.cc "!git diff --staged | claude -p '生成符合 Conventional Commits 的提交信息' | xargs git commit -m"
```
使用：`git lg`（查看历史）、`git cc`（生成规范提交）。

### 2. 批量清理无用分支
```
#  Claude 指令
清理本地已合并到 main 分支的无用 feature/* 分支，保留 main 和 develop 分支
#  执行逻辑
git checkout main
git pull origin main
git branch --merged main | grep -E "feature/*" | xargs git branch -d
```

### 3. 敏感文件彻底删除（历史清理）
```
#  Claude 指令
从 Git 历史中彻底删除 .env 文件，避免敏感信息泄露
#  执行逻辑
git filter-branch --tree-filter 'rm -f .env' HEAD
git push --force-with-lease origin main  # 谨慎使用！
```

---

## 五、安全避坑（核心禁忌）
1. **禁止直接操作 main 分支**：所有改动必须通过 feature/bugfix 分支开发，经评审后合并。
2. **禁止提交敏感信息**：密钥、密码、Token 等必须放入 .gitignore，禁止硬编码。
3. **禁止强制推送 main 分支**：强制推送会覆盖远程历史，导致团队协作混乱，仅可在个人 feature 分支使用 `--force-with-lease`。
4. **不做无意义提交**：禁止“update”“fix bug”等模糊提交信息，必须遵循 Conventional Commits 规范。
5. **定期同步 main 分支**：开发过程中频繁拉取 main 分支最新代码，避免合并冲突堆积。

---

## 六、总结
Claude Code + Git 的核心价值是**让 AI 编码可追溯、可评审、可回滚**。通过**分支隔离规范、原子提交规范、远程协作规范**，你可以实现 AI 编码与团队协作的无缝衔接，每一次改动都清晰留痕，大幅降低协作风险。

核心口诀：**分支隔离不碰主，原子提交留痕迹，规范提交好回溯，安全协作不踩坑**。