# Claude Code 子代理（Subagent）完全教学：从入门到实战
本文系统讲解 Claude Code 子代理（Subagent）的**概念、创建、调用、配置与实战**，提供可直接复制的命令与配置，帮你用“专业化分工”提升开发效率，适合日常开发、代码审查、项目初始化等场景。

---

## 一、核心概念：什么是 Subagent？
Subagent 是 Claude Code 中**独立的专用 AI 助手**，每个子代理拥有：
- 独立上下文窗口（不污染主会话）
- 自定义系统提示词（专属角色与规则）
- 细粒度工具权限（如只读、可写、终端操作）
- 专属模型（Haiku/Sonnet/Opus，按需选择）

**核心价值**：像团队分工一样，让专业子代理处理特定任务（如代码审查、调试、文档生成），主代理专注统筹，提升效率、保证质量、降低上下文开销。

### 适用场景
- 代码审查/质量把控
- 问题调试/故障定位
- 文档生成/维护
- 项目结构探索/分析
- 测试用例编写/执行
- 自动化脚本/命令生成

---

## 二、两种创建方式（推荐优先用交互式）
### 方式 1：交互式命令 `/agents`（推荐，零配置门槛）
1. 打开 Claude Code（CLI 或 VSCode 插件）
2. 输入命令：`/agents`，进入代理管理界面
3. 选择 **Create new agent** → 选择作用域
   - **Project**：`.claude/agents/`（当前项目专属，可提交 Git 共享）
   - **Personal**：`~/.claude/agents/`（全局通用，所有项目可用）
4. 填写配置（按提示操作）
   - 名称（唯一，如 `code-reviewer`）
   - 描述（用于自动路由匹配）
   - 系统提示（角色与规则，可让 Claude 自动生成）
   - 工具权限（按需勾选，最小权限原则）
   - 模型（Haiku 快速、Sonnet 平衡、Opus 复杂任务）
   - 颜色（UI 区分，可选）
5. 保存完成，自动生效

### 方式 2：手动创建 Markdown 配置文件（灵活可控，适合团队）
Subagent 本质是**带 YAML 前置元数据的 Markdown 文件**，手动创建更灵活。

#### 步骤 1：创建目录
```bash
# 项目级（推荐，纳入版本控制）
mkdir -p .claude/agents

# 用户级（全局通用）
mkdir -p ~/.claude/agents
```

#### 步骤 2：编写配置文件
以「代码审查子代理」为例，创建 `.claude/agents/code-reviewer.md`：
```markdown
---
name: code-reviewer
description: 代码审查专家，专注质量、安全与性能
tools: [Read, Grep, Glob, Bash]  # 授权工具：读文件、搜索、glob匹配、终端命令
model: sonnet  # 平衡性能与能力
permissionMode: default  # 默认权限（只读/可写按需调整）
---
# 角色与规则
你是资深后端代码审查专家，专注 Java/Spring Boot/前端 React 项目。
审查时必须遵循以下步骤：
1. 先运行 `git diff` 查看本次修改范围
2. 重点检查：安全漏洞（如SQL注入、XSS）、性能瓶颈、代码规范、可读性
3. 输出格式：按「Critical（必修）→ Warning（建议）→ Suggestion（参考）」分级，附带具体修改建议与代码示例

# 输出要求
- 语言：中文，简洁明了，每点不超过 50 字
- 必须标注文件路径与行号
- 提供可直接复制的修复代码
```


#### 步骤 3：生效配置
- 重启 Claude Code 会话，或再次运行 `/agents` 手动加载
- 手动创建的文件会自动被扫描，无需额外配置

---

## 三、三种调用方式（显式调用最可控）
### 方式 1：显式调用（推荐，100% 精准）
直接通过 `@agent-name` 或 `use <agent-name>` 语法，指定子代理处理任务，避免自动路由误判。

#### 语法示例
```bash
# 方式 A：@agent-name 语法（VSCode 插件/CLI 通用）
@code-reviewer 审查一下我刚修改的 UserController.java

# 方式 B：use agent-name 语法（更明确）
use code-reviewer 帮我分析 src/main/java/com/xxx/service/OrderService.java 的性能问题

# 方式 C：临时会话调用（CLI 专用，一次性）
claude --agent code-reviewer "审查整个 src/main/java 目录的代码质量"
```


### 方式 2：自动路由（便捷，适合简单场景）
Claude 根据子代理的 `description` 自动匹配任务，无需手动指定。

**示例**：
- 子代理描述：`代码审查专家，专注Java/Spring Boot项目`
- 你输入：`帮我审查刚提交的订单模块代码`
- Claude 自动匹配 `code-reviewer` 子代理处理

⚠️ 注意：自动路由可能误判，复杂任务优先用显式调用。

### 方式 3：默认代理（全局/项目生效）
设置默认代理，会话内所有任务自动委托给该子代理。

#### CLI 临时默认
```bash
claude --agent code-reviewer  # 本次会话默认使用 code-reviewer
```

#### 项目级永久默认（推荐，.claude/settings.json）
```json
{
  "agent": "code-reviewer"
}
```


---

## 四、核心配置详解（Frontmatter 字段）
Subagent 配置通过 Markdown 头部 YAML 定义，核心字段如下：

| 字段 | 必需 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| `name` | ✅ | 唯一标识（小写，无空格） | `code-reviewer` |
| `description` | ✅ | 描述（用于自动路由） | `Java代码审查，专注安全与性能` |
| `tools` | ✅ | 授权工具列表 | `[Read, Grep, Glob, Bash]` |
| `model` | ✅ | 模型选择 | `haiku`/`sonnet`/`opus` |
| `permissionMode` | ❌ | 权限模式 | `default`（默认）/`read-only`（只读） |
| `skills` | ❌ | 专属技能标签 | `[api-conventions, error-handling]` |
| `color` | ❌ | UI 显示颜色 | `blue`/`green`/`red` |

### 常用工具说明
- `Read`：读取文件内容（必备）
- `Grep`：跨文件搜索内容（代码搜索必备）
- `Glob`：按模式匹配文件（如 `src/**/*.java`）
- `Bash`：执行终端命令（谨慎授权，仅必要时开启）
- `Write`：写入文件（仅修改类代理需要）

---

## 五、实战示例：3 个常用子代理配置
### 示例 1：代码审查子代理（前面已给，补充优化）
```markdown
---
name: code-reviewer
description: Java/Spring Boot代码审查，专注安全、性能与规范
tools: [Read, Grep, Glob, Bash]
model: sonnet
---
# 角色
你是资深 Java 工程师，专注 Spring Boot 项目代码审查。

# 审查规则
1. 安全：禁止硬编码密钥、检查SQL注入风险、验证权限控制
2. 性能：检查N+1查询、慢SQL、不合理缓存
3. 规范：遵循阿里巴巴Java开发手册，变量命名、注释规范
4. 可维护性：单一职责、代码复用、异常处理

# 输出
按 Critical/ Warning/ Suggestion 分级，附文件路径、行号与修复代码
```


### 示例 2：项目探索子代理（内置 Explore 自定义优化）
```markdown
---
name: project-explorer
description: 快速探索项目结构，分析模块依赖
tools: [Glob, Grep, Read, Bash]
model: haiku  # 快速低延迟，适合探索
---
# 角色
你是项目架构分析师，快速梳理项目结构与模块依赖。

# 工作流程
1. 用 Glob 列出核心目录（src/main/java, src/main/resources, pom.xml）
2. 用 Grep 搜索核心接口/类（如 @RestController、@Service）
3. 输出：项目结构树、核心模块列表、关键依赖关系
```


### 示例 3：测试用例生成子代理
```markdown
---
name: test-generator
description: 生成Java单元测试用例，覆盖边界条件
tools: [Read, Write, Glob]
model: sonnet
---
# 角色
你是测试专家，为Java方法生成 JUnit 5 + Mockito 单元测试。

# 生成规则
1. 覆盖正常流程、异常流程、边界值
2. 模拟依赖（@Mock），避免外部依赖
3. 测试类命名：原类名 + Test（如 UserService → UserServiceTest）
4. 放在 src/test/java 对应包路径下
```


---

## 六、管理与维护（高效迭代子代理）
### 查看/编辑子代理
```bash
# 进入管理界面（所有方式通用）
/agents
```
- 查看：所有子代理（内置/用户级/项目级）
- 编辑：修改已有子代理配置
- 删除：移除无用子代理
- 刷新：重新加载配置文件

### 优先级规则（避免冲突）
1. CLI 临时代理（`--agent`） > 项目级代理 > 用户级代理 > 内置代理
2. 项目级代理会覆盖用户级同名代理

### 版本控制（团队协作）
- 项目级子代理（`.claude/agents/`）提交到 Git，团队共享统一配置
- 用户级子代理（`~/.claude/agents/`）个人专用，不提交 Git

---

## 七、常见问题与避坑
1. **子代理不生效**
   - 检查文件名是否为 `.md` 格式
   - 检查 `name` 是否唯一、无空格
   - 重启会话或运行 `/agents` 手动刷新

2. **权限不足/工具未授权**
   - 遵循最小权限原则，仅授予必要工具
   - 写文件类代理需添加 `Write` 工具
   - 终端命令类代理需添加 `Bash` 工具，谨慎使用

3. **自动路由误判**
   - 复杂任务优先用显式调用（`@agent-name`）
   - 优化 `description`，让描述更精准

4. **上下文泄漏**
   - 每个子代理独立上下文，不会污染主会话
   - 避免在子代理中执行跨项目操作，防止权限越界

---

## 八、总结与最佳实践
1. **先基础后进阶**：先用交互式 `/agents` 创建常用代理（审查、探索、测试），熟悉后手动配置
2. **显式调用优先**：复杂任务用 `@agent-name`，保证可控性
3. **最小权限原则**：仅授予必要工具，降低风险
4. **团队共享配置**：项目级代理提交 Git，统一团队规范
5. **持续迭代**：根据使用反馈优化子代理的系统提示与工具权限

通过 Subagent，你可以把 Claude Code 打造成“专属开发团队”，每个子代理各司其职，大幅提升开发效率、代码质量与协作体验。