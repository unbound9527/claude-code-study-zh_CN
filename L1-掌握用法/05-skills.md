# Claude Code Skill 完全教学：是什么、安装、使用与自定义
Skill（技能）是 Claude Code 最核心的**可复用能力扩展系统**，本质是把专业工作流、提示词、规范、工具调用逻辑封装成标准化模块，让 Claude 从通用助手变成垂直领域专家，实现“一次定义、永久复用、自动触发”。本文从概念、安装、使用、自定义、实战全流程讲解，彻底掌握 Skill 提效神器。

---

## 一、Skill 是什么？核心概念与价值
### 1. 官方定义（一句话）
**Skill = 标准化工作流 + 可复用提示词 + 自动触发规则**
- 载体：**SKILL.md**（Markdown 文件，含 YAML 元数据 + 执行指令）
- 本质：给 Claude 写的**专业工作手册**，告诉它“何时做、做什么、怎么做、用什么工具”
- 定位：**最小能力单元**（类似函数），可安装、可共享、可版本管理

### 2. 与普通提示词、Hooks、Subagent 的区别
| 特性 | 普通提示词 | Skill（技能） | Hooks | Subagent |
|:--- |:--- |:--- |:--- |:--- |
| **复用性** | 一次性 | 永久复用、跨项目 | 事件自动触发 | 独立AI分工 |
| **触发方式** | 手动输入 | 手动 `/命令` + 自动语义匹配 | 事件驱动（Pre/Post） | 主代理分配 |
| **结构** | 自由文本 | 标准化（元数据+工作流） | JSON+脚本 | 独立AI实例 |
| **权限** | 无控制 | 可限制工具（Read/Write/Bash） | 允许/阻止/修改 | 独立上下文 |
| **适用场景** | 临时任务 | 专业流程、规范固化 | 安全、自动化、审计 | 复杂项目协作 |

### 3. 三大核心价值（为什么必学）
✅ **告别重复提示**：团队规范、编码标准、工作流**一次写入，永久生效**
✅ **自动专业能力**：自然语言描述需求，Claude **自动匹配 Skill**，按专家流程执行
✅ **标准化协作**：团队共享 Skill，**统一输出质量、消除沟通成本**

---

## 二、支持版本、模型与权限
### 1. 版本要求
- Claude Code **v1.8+**（2025年后稳定版）
- **所有订阅**：免费版 / Pro / Max 均支持（无功能限制）

### 2. 模型兼容性（关键）
- **全模型支持**：Claude 3（Opus/Sonnet/Haiku）、Claude 2、**国产模型（通义/GLM/DeepSeek）**
- Skill 是**客户端本地功能**，与后端模型无关，**切换国产模型完全可用**

### 3. 权限与作用域
- **全局 Skill**：`~/.claude/skills/`（所有项目生效）
- **项目 Skill**：`./.claude/skills/`（仅当前项目，可 Git 共享）
- **临时 Skill**：会话内创建，重启失效

---

## 三、Skill 核心结构：SKILL.md 全解析
每个 Skill 都是一个目录，**必须含 SKILL.md**（文件名全大写）：
```
.claude/skills/skill-name/
├── SKILL.md       # 核心配置（必选）
├── scripts/       # 可选：Shell/Python/JS脚本
└── references/    # 可选：参考文档、规范
```

### SKILL.md 标准结构（两部分）
```markdown
---
# 第一部分：YAML 元数据（配置头）
name: skill-name       # 唯一标识（小写、连字符，命令名）
description: >         # 触发描述（功能+关键词，决定自动匹配）
  代码审查：检查风格、Bug、性能，用户说"审查代码""code review"时触发
version: 1.0.0         # 版本（可选）
author: xxx            # 作者（可选）
allowed-tools: Read,Grep,Bash,Edit  # 允许的工具（权限控制，可选）
disable-model-invocation: false     # 是否禁止自动触发（可选）
---

# 第二部分：Markdown 执行工作流（核心指令）
## 执行步骤
1. 读取目标文件：`Read $FILE_PATH`
2. 运行 ESLint 检查：`Bash eslint $FILE_PATH`
3. 分析结果，生成审查报告
4. 标注问题代码，给出修复建议

## 输出格式
### 代码审查报告
- **文件**：$FILE_PATH
- **问题数**：X
- **严重级别**：高/中/低
- **修复建议**：...
```

### 关键字段说明
- **name**：命令名（`/skill-name`），必须与目录名一致
- **description**：**自动触发核心**，包含**功能+关键词**，Claude 语义匹配
- **allowed-tools**：最小权限原则，禁止危险操作（如 `rm -rf`）

---

## 四、安装与管理：3种方式（新手首选）
### 方式1：CLI 工具安装（官方/社区 Skill，最快）
#### 1. 安装 Skill 管理器（必备）
```bash
# 全局安装（推荐）
npm install -g skills

# 验证
skills --version
```

#### 2. 安装常用 Skill（示例）
```bash
# 安装「技能发现器」（应用商店，必装）
skills add find-skills -g -y

# 安装代码审查
skills add code-review -g -y

# 安装智能 Git 提交
skills add smart-commit -g -y

# 安装前端规范
skills add frontend-standards -g -y
```
- `-g`：全局安装（所有项目生效）
- `-y`：自动确认

#### 3. 管理命令
```bash
skills list -g          # 查看已安装
skills remove <name> -g # 卸载
skills update -g        # 更新所有
skills search <关键词>  # 搜索社区 Skill
```

### 方式2：插件市场安装（官方插件）
```bash
# Claude Code 内直接执行
/plugin install document-skills@anthropic-agent-skills  # 文档处理
/plugin install code-simplifier@claude-plugins-official # 代码简化
/plugin list                                           # 查看插件
```

### 方式3：手动创建（自定义 Skill，进阶）
#### 1. 创建目录与文件
```bash
# 项目级（推荐团队共享）
mkdir -p .claude/skills/code-explainer
touch .claude/skills/code-explainer/SKILL.md
```

#### 2. 编写 SKILL.md（见第三节）
#### 3. 重启 Claude Code 加载

---

## 五、使用 Skill：3种触发方式（全自动）
### 1. 手动斜杠命令（最精准）
直接输入 `/` + Skill 名，支持参数：
```bash
# 基础用法
/code-review src/utils.js  # 审查文件
/smart-commit              # 智能提交
/explain src/index.js      # 解释代码

# 带参数
/frontend-design --framework nextjs --style tailwind 响应式仪表盘
```

### 2. 自然语言自动触发（最省心）
**无需命令**，描述需求，Claude **自动匹配 description 关键词**：
```bash
# 自动触发 code-review
帮我审查一下这个文件的代码质量

# 自动触发 smart-commit
帮我提交代码，写规范的提交信息

# 自动触发 code-explainer
解释这段代码的执行逻辑
```

### 3. 组合调用（工作流）
```bash
# 先审查，再提交
/code-review src/app.js && /smart-commit

# 先分析，再重构
/project-explorer && /refactor
```

### 4. 查看可用 Skill
```bash
# Claude Code 内
/skills          # 列出所有可用
/skill-check     # 校验 Skill 语法
```

---

## 六、实战：5个高频 Skill（直接复制）
### 实战1：代码解释器（自动触发）
**文件**：`.claude/skills/code-explainer/SKILL.md`
```markdown
---
name: code-explainer
description: 用ASCII流程图+通俗类比解释代码，用户问"解释代码""执行流程"时触发
allowed-tools: Read
---
# 代码解释规则
1. 一句话总结核心功能
2. 绘制ASCII执行流程图
3. 生活化类比（如"像快递分拣"）
4. 分步骤拆解+边界说明

## 输出格式
### 功能概述
[一句话]
### 执行流程
```
[ASCII图]
```
### 通俗类比
[类比]
### 分步解析
1. ...
```

### 实战2：智能Git提交（必装）
```markdown
---
name: smart-commit
description: 自动分析变更，生成Conventional Commits提交信息
allowed-tools: Bash,Read
---
# 提交步骤
1. git diff --staged 分析变更
2. 识别类型：feat/fix/docs/refactor/test
3. 生成描述（英文+中文）
4. 执行 git commit -m "类型: 描述"
```

### 实战3：前端规范强制（团队必备）
```markdown
---
name: frontend-standards
description: 强制React+Tailwind规范，写前端代码/审查时触发
allowed-tools: Read,Edit
---
# 前端规范
- 组件：PascalCase，单文件单组件
- 样式：仅Tailwind，禁止内联
- 状态：Zustand，禁止Redux
- 测试：覆盖率≥80%
```

### 实战4：API文档生成
```markdown
---
name: api-doc
description: 从代码生成API文档（路径、参数、响应、错误码）
allowed-tools: Read
---
# 文档生成步骤
1. 提取Endpoint（方法+路径）
2. 解析请求参数（类型/必填/说明）
3. 定义响应格式（成功/失败）
4. 列出错误码与示例
```

### 实战5：项目健康检查
```markdown
---
name: project-audit
description: 全面检查项目：依赖、漏洞、规范、性能
allowed-tools: Bash,Read,Glob
---
# 检查清单
1. npm audit 查漏洞
2. ESLint 查规范
3. 检测未使用代码
4. 性能瓶颈分析
5. 生成优化报告
```

---

## 七、运行表现与效果（你会看到）
### 1. 自动匹配
- 输入需求 → Claude 扫描所有 Skill 的 `description` → **自动加载匹配 Skill**
- 终端显示：`[Skill Loaded] code-explainer`

### 2. 按流程执行
- 严格遵循 SKILL.md 工作流，**步骤化执行**
- 自动调用工具（Read/Bash/Edit），无需干预
- 输出**标准化格式**，统一质量

### 3. 权限管控
- 超出 `allowed-tools` 的操作 **直接阻止**
- 安全可控，避免误操作

---

## 八、最佳实践与避坑
### 1. 最佳实践
- **最小够用**：每个 Skill 专注**单一任务**（单一职责）
- **精准描述**：`description` 包含**功能+关键词**，提升自动触发率
- **权限最小化**：`allowed-tools` 仅开放必要工具
- **团队共享**：项目 Skill 提交 Git，统一规范
- **定期清理**：卸载不常用 Skill，减少上下文负担

### 2. 常见问题
#### 问题1：Skill 不显示/不触发
- 重启 Claude Code（必须完全退出）
- 检查 `name` 与目录名一致
- 校验 `description` 包含关键词
- 用 `/skill-check` 校验语法

#### 问题2：自动触发混乱
- 优化 `description`，**减少歧义关键词**
- 设 `disable-model-invocation: true`，仅手动触发

#### 问题3：国产模型下失效
- **不可能失效**：Skill 是客户端功能，与模型无关
- 检查安装路径与重启

---

## 九、总结
### 核心要点
- **Skill = 可复用专业工作流**，SKILL.md 是核心
- **3种触发**：手动 `/命令`、自动语义匹配、组合调用
- **全模型支持**：Claude 3 / 国产模型均可用
- **核心价值**：**告别重复提示、自动专业能力、标准化协作**

### 一句话建议
**所有项目必装 4 个 Skill**：`code-review`、`smart-commit`、`code-explainer`、`project-audit`，效率提升 **50%+**，质量完全标准化。