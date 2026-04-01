# Claude Code Git 最佳实践（权威来源汇总）
以下实践均来自 **Anthropic 官方、Claude Code 核心团队、安全厂商、顶级技术社区** 等可信来源，每条标注 **来源、作者/机构、核心规则、操作示例**，覆盖分支、提交、协作、安全、自动化全流程。

---

## 一、分支管理最佳实践（官方+创始人）
### 1. 独立 AI 协作分支（安全核心）
- **来源**：Anthropic 官方文档、CSDN《Claude Code Git 集成全流程》
- **作者**：Anthropic 安全团队、CSDN 技术专家 canjun_wen
- **核心规则**：
  - **禁止 AI 直接操作 main/master**（官方强制）
  - 新功能/修改必须创建 **AI 专属分支**
  - 命名规范：`feat/ai-collab-功能名`、`fix/ai-collab-issue号`
- **操作示例**：
  ```bash
  git checkout main && git pull origin main       # 同步最新主分支
  git checkout -b feat/ai-collab-user-auth       # 创建AI分支
  claude git checkout feat/ai-collab-user-auth    # 通知Claude切换
  ```

### 2. 原子分支（小任务、单功能）
- **来源**：Anthropic 工程博客、GitHub《Claude Code Best Practices》
- **作者**：Anthropic 工程团队、rmurphey（GitHub 权威）
- **核心规则**：
  - 1 分支 = 1 功能/1 Bug 修复（≤200 行改动）
  - 禁止跨功能分支、长期存活分支
  - 合并后立即删除远程/本地分支

### 3. 分支权限隔离（企业级）
- **来源**：Trail of Bits、七牛云安全实践
- **作者**：Trail of Bits（顶级安全公司）
- **核心规则**：
  - main：**只读**，AI 无写入/合并权限
  - dev：AI 可读写，需人工审核合并
  - feat/ai-*：AI 完全操作，人工复核

---

## 二、提交（Commit）最佳实践（官方+社区）
### 1. 原子提交（官方强制）
- **来源**：Anthropic《Agentic Coding Best Practices》、GitHub 权威指南
- **作者**：Anthropic 开发者关系团队、rmurphey
- **核心规则**：
  - 1 提交 = 1 逻辑改动（≤50 行）
  - 先规划提交边界、再修改代码（禁止事后拆分）
  - 每步修改 → 立即提交 → 推送（小步迭代）
- **反例**：1 次提交改 10 个文件、跨 3 功能

### 2. 规范提交信息（Conventional Commits）
- **来源**：Anthropic 官方、DEV Community、GitHub
- **作者**：Anthropic 提示工程团队、DEV Community 专家
- **标准格式**：
  ```
  <type>(<scope>): <subject> (<via Claude Code>)
  例：feat(auth): add JWT token validation (via Claude Code)
  ```
- **type 规范**：feat/fix/docs/style/refactor/test/chore
- **配置**：`.gitmessage`、`.github/COMMIT_CONVENTION.md`

### 3. AI 提交追溯（可审计）
- **来源**：Anthropic 安全文档、GitHub《Git Attribution》
- **作者**：Anthropic 安全团队
- **规则**：
  - 所有 AI 生成提交 **必须标注 (via Claude Code)**
  - 团队共享 `CLAUDE.md` 强制提交规范
  - 禁止 AI 伪造作者信息、隐藏 AI 痕迹

---

## 三、工作流（GitHub Flow）最佳实践（官方+创始人）
### 1. 官方标准 AI 工作流（Anthropic 核心）
- **来源**：Anthropic《Claude Code Workflow》、创始人 Boris Cherny
- **作者**：Anthropic 核心团队、Boris Cherny（Claude Code 创始人）
- **6 步标准流程**：
  1. **开分支**：`feat/ai-collab-xxx`（AI 专属）
  2. **任务规划**：Claude 生成任务清单、确认范围
  3. **代码修改**：小步迭代、每改 1 文件 → 测试
  4. **原子提交**：规范信息、标注 AI 来源
  5. **创建 PR**：自动生成描述、关联 Issue
  6. **人工审核**：必须人工 review → 合并 → 删除分支

### 2. 自动化斜杠命令（创始人推荐）
- **来源**：Boris Cherny 13 条秘籍、GitHub 工作流
- **作者**：Boris Cherny（Claude Code 创始人）
- **推荐命令**：
  - `/start`：创建 AI 分支、同步主分支
  - `/commit-smart`：自动分析 diff、规范提交、运行测试
  - `/create-pr`：生成 PR 描述、关联 Issue、检查清单
  - `/review-pr`：AI 代码审查、安全扫描
- **共享配置**：命令存入 `.claude/commands/`，Git 共享团队

---

## 四、代码审查与合并（官方+企业）
### 1. AI 改动必审（安全红线）
- **来源**：Anthropic 安全团队、SlowMist 安全清单
- **作者**：Anthropic 安全团队、SlowMist（慢雾安全）
- **强制规则**：
  - **所有 AI 代码必须 100% 人工审核**（生产禁用自动合并）
  - 审核要点：diff 正确性、无安全漏洞、符合规范
  - 禁止：`--dangerously-skip-review`、无审核合并

### 2. PR 模板与检查清单（官方）
- **来源**：Anthropic 官方、GitHub《PR Best Practices》
- **作者**：Anthropic 文档团队
- **PR 模板（.github/pull_request_template.md）**：
  ```markdown
  ## 变更说明
  ## 关联 Issue: #xxx
  ## ✅ 测试清单
  - [ ] 单元测试通过
  - [ ] 集成测试通过
  - [ ] 代码审查通过
  - [ ] AI 改动人工复核
  ## Claude 会话 ID: xxx
  ```

### 3. 冲突智能解决（官方）
- **来源**：Anthropic 工程实践、掘金《Claude Code Git 指南》
- **作者**：Anthropic 工程团队、掘金技术专家
- **流程**：
  ```
  > 解决当前合并冲突，优先保留新功能，兼容原有错误处理
  Claude: 分析冲突 → 理解意图 → 智能合并 → 运行测试 → 确认
  ```

---

## 五、安全与权限实践（官方+安全厂商）
### 1. Git 权限最小化（官方）
- **来源**：Anthropic《Claude Code Permissions》、Trail of Bits
- **作者**：Anthropic 安全团队、Trail of Bits
- **配置示例**：
  ```json
  {
    "git": {
      "allow": ["checkout", "status", "diff", "fetch", "push"],
      "deny": ["push -f", "reset --hard", "rebase -i", "branch -D"],
      "mainProtection": true
    }
  }
  ```

### 2. 敏感文件保护（企业级）
- **来源**：Anthropic、七牛云安全
- **作者**：Anthropic 安全团队、七牛云安全团队
- **规则**：
  - `.gitignore` + `.claudeignore` 双重屏蔽：`.env`、`*.key`、`secrets/`
  - 禁止 AI 读取/提交敏感文件、配置
  - 提交前自动扫描：`git diff --check`、安全审计

### 3. 提交签名（企业安全）
- **来源**：Anthropic GitHub Action 文档
- **作者**：Anthropic CI/CD 团队
- **实践**：
  ```yaml
  uses: anthropics/claude-code-action@main
  with:
    use_commit_signing: true  # 自动签名、GitHub 验证
  ```

---

## 六、历史管理与审计（官方+社区）
### 1. 完整历史追溯（可审计）
- **来源**：Anthropic、GitHub 社区
- **作者**：Anthropic 工程团队
- **实践**：
  - 禁止 `git rebase`、`git commit --amend` 修改 AI 历史
  - 保留所有 AI 提交、会话 ID、操作日志
  - 用 Claude 审计历史：`分析近10次AI提交，检查安全问题`

### 2. Git 日志利用（效率）
- **来源**：Anthropic 官方、DataCamp
- **作者**：Anthropic 开发者关系团队
- **用法**：
  ```
  > 查看v2.1.0版本所有AI改动
  > 查找登录功能bug引入的提交
  > 分析近30天提交趋势、AI参与度
  ```

---

## 七、权威来源清单（可直接验证）
1. **Anthropic 官方**：《Claude Code Overview》《Security》《Workflow》
2. **Boris Cherny**：Claude Code 创始人，13 条使用秘籍
3. **Trail of Bits**：顶级安全公司，`claude-code-config` 安全规范
4. **SlowMist**：慢雾安全，AI 代码安全清单
5. **GitHub 权威**：rmurphey、awattar 等社区最佳实践
6. **DEV Community**：Git 协作与追溯规范
7. **掘金/CSDN**：国内顶级技术社区实战指南

---

## 一句话总结
**Claude Code Git 黄金法则**：**独立分支、原子提交、规范信息、人工必审、全程追溯**（Anthropic+创始人 Boris 联合推荐）；个人抓流程规范，企业加安全隔离与审计。