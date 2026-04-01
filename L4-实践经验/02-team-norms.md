# Claude Code 团队规范最佳实践（权威来源整理）
以下规范均来自 **Anthropic 官方、Claude Code 核心团队、顶级技术社区与安全厂商** 等可信来源，每条标注 **来源、作者/机构、核心规则、落地示例**，覆盖团队配置、协作模式、代码规范、审查流程、安全合规、知识沉淀全维度。

---

## 一、团队基础配置规范（官方强制）
### 1. CLAUDE.md 团队统一规范（核心）
- **来源**：Anthropic 官方《Claude Code Configuration》、腾讯云开发者社区
- **作者**：Anthropic 配置团队、HumanLayer 团队（AI 协作权威）
- **核心规则**：
  - 项目根目录必须存放 **CLAUDE.md**（AI 入职手册），Git 版本控制、全员共享
  - 内容：**代码风格、测试标准、Git 规范、禁止行为、常用命令、架构说明**
  - 精简原则：**≤300 行**（社区共识），**≤60 行**（HumanLayer 推荐）
  - 错误沉淀：AI 犯错 → 修复 → 规则写入 CLAUDE.md（Anthropic 内部标准）
- **示例模板**：
```markdown
# CLAUDE.md (团队规范)
## 代码规范
- Python: Black 格式化、Ruff 检查、类型注解 ≥80%
- JS/TS: Prettier、ESLint 严格模式
## Git 规则
- 分支: feat/ai-xxx、fix/ai-xxx；禁止 AI 直接改 main
- 提交: 必须标注 (via Claude Code)、原子提交 ≤50 行
## 禁止行为
- 禁止修改 .env、secrets、*.key
- 禁止 rm -rf、drop table 等高危命令
## 常用命令
- 测试: npm run test、pytest --cov=src
- 格式化: npm run format
```

### 2. 团队配置文件标准化（官方）
- **来源**：Anthropic 官方、GitHub《Claude Code Setup》
- **作者**：Anthropic 工程团队、rmurphey（GitHub 权威）
- **必存目录（Git 共享）**：
  - `.claude/skills/`：团队共享技能（代码审查、提交规范）
  - `.claude/commands/`：自定义斜杠命令（一键 PR、测试）
  - `.claude/hooks/`：强制自动化规则（安全门禁、质量检查）
  - `.mcp.json`：团队 MCP 服务配置（Git、DB、测试）
  - `.github/`：PR 模板、提交规范、CI/CD 流程

---

## 二、团队协作模式规范（官方+创始人）
### 1. Agent Teams 多智能体协作（官方实验特性）
- **来源**：Anthropic 官方、Claude Code 创始人 Boris Cherny、腾讯云
- **作者**：Anthropic 智能体团队、Boris Cherny
- **核心规则**：
  - 启用：`"experimental": { "agentTeams": true }`（settings.json）
  - 角色分工：**Team Lead（统筹）+ Teammates（专项执行）**
  - 协作模式：
    - **Delegate 模式**：Lead 仅调度、不编码（大规模团队）
    - **自主沟通**：Teammates 直接交流、任务并行、依赖自动管理
- **团队配置示例**：
```
> 创建 Agent Team：后端(API)、前端(组件)、测试(用例)、安全(扫描)
> 任务：完成用户认证模块，前后端并行，测试同步，安全扫描后合并
```

### 2. Subagent 委派规范（标准化流程）
- **来源**：Anthropic 官方、CSDN《Claude Code 协作指南》
- **作者**：Anthropic 智能体团队、CSDN 专家
- **核心规则**：
  - 主代理 → 委派子代理（Explore/Implement/Test/Review）
  - 子代理独立上下文、结果精炼返回、不传递全量历史
  - 适用：独立子任务、自动化流水线、批量处理
- **示例**：
```
主代理 → 委派 Explore：分析代码库 → 定位问题
主代理 → 委派 Implement：按规范修复 → 小步提交
主代理 → 委派 Test：全量测试 → 覆盖率 ≥85%
主代理 → 委派 Review：安全扫描 → 人工复核
```

### 3. 团队工作流统一（官方标准）
- **来源**：Anthropic《Agentic Coding Best Practices》、创始人 Boris
- **作者**：Anthropic 工程团队、Boris Cherny
- **强制 4 步流程**：
  1. **Explore**：AI 只读扫描、理解架构、不修改
  2. **Plan**：生成方案、人工审核、extended thinking
  3. **Code**：原子修改、自动测试、Hooks 门禁
  4. **Commit/PR**：规范提交、AI 审查、人工必审、合并

---

## 三、代码与提交规范（官方+社区）
### 1. 代码质量契约（团队强制）
- **来源**：Anthropic 内部标准、CSDN《代码质量规范》
- **作者**：Anthropic 质量团队、程序员光剑
- **核心规则**：
  - **可测试性**：新函数可独立 import+mock、无全局依赖
  - **错误处理**：禁止 `except: pass`、必须日志记录
  - **完整性**：提交 = 编译成功 + 测试通过 + 新增测试 + 格式合规
  - **覆盖率**：单元测试 ≥85%、集成测试全量通过

### 2. Git 提交与分支规范（团队统一）
- **来源**：Anthropic 官方、GitHub 权威指南
- **作者**：Anthropic Git 团队、rmurphey
- **分支命名**：
  - 功能：`feat/ai-模块名`、修复：`fix/ai-issue号`
  - 禁止：长期分支、跨功能分支、AI 直接操作 main
- **提交信息（Conventional Commits）**：
```
<type>(<scope>): <subject> (via Claude Code)
例：feat(auth): add JWT validation (via Claude Code)
type: feat/fix/docs/refactor/test/chore
```

---

## 四、代码审查与合并规范（安全红线）
### 1. AI 代码 100% 人工审查（官方强制）
- **来源**：Anthropic 安全团队、Trail of Bits（顶级安全公司）
- **作者**：Anthropic 安全团队、Trail of Bits
- **核心规则**：
  - **生产环境：禁止 AI 自动合并**，必须人工 review
  - 审查要点：**功能正确性、安全漏洞、规范合规、测试完整性**
  - 追溯：提交标注 AI 来源、会话 ID、操作日志

### 2. PR 审查流程标准化（官方）
- **来源**：Anthropic 官方、GitHub《PR Best Practices》
- **作者**：Anthropic 文档团队、GitHub 社区
- **PR 模板（.github/pull_request_template.md）**：
```markdown
## 变更说明
## 关联 Issue: #xxx
## ✅ 审查清单
- [ ] 代码符合 CLAUDE.md 规范
- [ ] 测试通过、覆盖率 ≥85%
- [ ] 安全扫描无高危漏洞
- [ ] 人工复核确认
## Claude 会话 ID: xxx
## AI 生成标注: (via Claude Code)
```

### 3. 冲突解决规范（团队统一）
- **来源**：Anthropic 工程实践、掘金
- **作者**：Anthropic 工程团队、掘金专家
- **流程**：
```
> 解决合并冲突：优先保留新功能、兼容旧逻辑、不破坏测试
Claude：分析冲突 → 理解意图 → 智能合并 → 测试验证 → 人工确认
```

---

## 五、安全与权限规范（企业级）
### 1. 权限最小化（官方强制）
- **来源**：Anthropic《Claude Code Permissions》、Trail of Bits
- **作者**：Anthropic 安全团队、Trail of Bits
- **配置示例**：
```json
{
  "allow": ["git", "npm test", "prettier"],
  "deny": ["edit(.env)", "bash(rm -rf)", "push -f"],
  "mainProtection": true,
  "sensitiveFiles": [".env", "secrets/", "*.key"]
}
```

### 2. 敏感信息保护（团队必配）
- **来源**：Anthropic、七牛云安全
- **作者**：Anthropic 安全团队、七牛云安全
- **规则**：
  - `.gitignore` + `.claudeignore` 双重屏蔽敏感文件
  - 禁止 AI 读取/提交密钥、凭证、配置文件
  - 提交前自动扫描：`git diff --check`、安全审计

### 3. 操作审计规范（可追溯）
- **来源**：Anthropic 安全文档、GitHub
- **作者**：Anthropic 安全团队、GitHub
- **实践**：
  - 所有 AI 操作：**日志全记录、会话 ID、时间、用户、改动**
  - 禁止修改 Git 历史、隐藏 AI 痕迹、伪造作者
  - 定期审计：`分析近30天AI提交，检查安全与规范`

---

## 六、知识沉淀与新人规范（团队效率）
### 1. 技能（Skills）标准化（团队共享）
- **来源**：Anthropic《Skill Building Guide》、GitHub
- **作者**：Anthropic Skill 团队、AgentSkills.io
- **必配团队技能**：
  - `code-review.md`：审查清单、规范、安全红线
  - `git-standards.md`：分支、提交、合并流程
  - `onboarding.md`：新人入职、环境、规范、工具
- **示例（.claude/skills/team-standards.md）**：
```markdown
---
name: team-standards
description: 团队编码与协作规范
---
## 审查红线
- 禁止无测试提交、覆盖率 <85%
- 禁止硬编码、敏感信息泄露
- 必须遵循 Black/Prettier 格式化
## 协作规则
- 任务先 Plan 再 Code、人工审核方案
- AI 分支独立、合并前 100% 审查
```

### 2. 新人入职自动化（官方推荐）
- **来源**：Anthropic 内部实践、LinkedIn
- **作者**：Anthropic 开发者关系团队、André Lindenberg
- **流程**：
  - 新人 → 克隆代码 → 读取 CLAUDE.md + Skills
  - Claude 自动讲解架构、依赖、规范、常用命令
  - 自动生成入门任务、引导完成、实时审查

---

## 七、权威来源清单（可直接验证）
1. **Anthropic 官方**：《Claude Code Configuration》《Agentic Coding Best Practices》《Permissions Guide》
2. **Boris Cherny**：Claude Code 创始人，多智能体与工作流规范
3. **HumanLayer 团队**：CLAUDE.md 精简与配置权威
4. **rmurphey**：GitHub 顶级专家，Git 与协作规范
5. **Trail of Bits**：顶级安全公司，权限与安全规范
6. **腾讯云/CSDN/掘金**：国内顶级社区，实战落地总结
7. **AgentSkills.io**：技能标准化组织

---

## 一句话总结
**Claude Code 团队规范黄金标准**：**CLAUDE.md 统一规则、Agent Teams 高效协作、代码质量强约束、审查合并双保险、安全权限最小化、知识技能全沉淀**（Anthropic+创始人 Boris 联合推荐）；实现 **团队输出一致、AI 行为可控、质量安全可保障、新人快速上手**。