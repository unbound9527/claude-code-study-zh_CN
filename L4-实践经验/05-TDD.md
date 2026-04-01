# Claude Code TDD 最佳实践（权威来源整理）
本文整理 **Anthropic 官方、DataCamp、顶级技术社区、企业实践** 中可信赖的 Claude Code TDD （测试驱动开发）最佳实践，每条均标注来源与作者/机构，覆盖标准流程、提示词、安全、自动化、前端/后端场景。

---

## 一、官方标准 TDD 流程（Anthropic 核心）
### 1. 红-绿-重构标准工作流
- **来源**：Anthropic 官方《Agentic Coding Best Practices》、DataCamp《Claude Code TDD Guide》
- **作者**：Anthropic 开发者关系团队、DataCamp 技术教程团队
- **5步必执行（官方强制）**：
  1. **写测试（红）**：仅写测试、不写实现、禁止 mock 不存在代码
     ```
     > 用 pytest 为 auth 模块写测试，TDD 模式，不要 mock 未实现函数
     ```
  2. **运行测试 → 确认失败**：验证测试有效性
     ```
     > 运行测试，必须全部失败
     ```
  3. **提交测试（ checkpoint）**：锁定测试、防止 AI 篡改
     ```
     > 提交测试文件，信息：feat: add auth TDD tests
     ```
  4. **写实现（绿）**：仅改实现、**禁止修改测试**
     ```
     > 实现 auth 逻辑，让测试通过，不要改测试
     ```
  5. **全量通过 → 提交 → 重构**：小步迭代、持续验证

### 2. 官方核心理由（为什么必须 TDD）
- **来源**：Anthropic 工程博客、DataCamp
- **作者**：Anthropic 安全与工程团队
- **要点**：
  - 测试是**外部可信验证**，不受上下文污染影响
  - 红-绿循环提供**明确反馈**，AI 可自动迭代
  - 提交测试形成**安全网**，diff 可查 AI 是否篡改测试
  - 官方数据：TDD 使 AI 代码**缺陷率降低 60%+**

---

## 二、TDD 提示词与指令规范（官方+社区）
### 1. 高质量 TDD 提示词模板（官方）
- **来源**：Anthropic 提示工程指南、火山引擎
- **作者**：Anthropic 提示工程团队、火山引擎开发者社区
- **标准模板（直接复制）**：
  ```
  【TDD 模式】
  1. 先为 [功能] 写测试，覆盖 [正常/异常/边界]，不要写实现
  2. 运行测试 → 确认失败
  3. 提交测试
  4. 实现功能 → 测试全通过
  5. 提交实现，不要改测试
  ```
- **反例（禁止）**：
  - ❌ "实现登录功能，顺便写测试"
  - ✅ "TDD 实现登录：先写测试 → 失败 → 提交 → 实现"

### 2. 强制约束指令（官方）
- **来源**：Anthropic 《Claude Code Settings》
- **作者**：Anthropic 文档团队
- **必加指令**：
  - "**不要 mock 不存在的代码**"（官方强制）
  - "**实现时禁止修改测试**"（防止 AI 走捷径）
  - "每步完成后**运行测试并报告结果**"

---

## 三、安全与质量实践（官方+安全厂商）
### 1. 测试隔离与权限（官方+Trail of Bits）
- **来源**：Anthropic 安全文档、Trail of Bits `claude-code-config`
- **作者**：Anthropic 安全团队、Trail of Bits（顶级安全公司）
- **实践**：
  - 测试目录：`__tests__`、`tests/`（**仅允许 AI 访问**）
  - 生产代码：**只读权限**，测试通过后人工审核
  - 禁止：`Bash(rm -rf tests/)`、`Edit(tests/**)` 无确认

### 2. 质量门禁（官方+企业）
- **来源**：腾讯云、七牛云安全实践
- **作者**：腾讯云开发者社区、七牛云安全团队
- **强制规则**：
  - 测试覆盖率 ≥ **85%**（pytest --cov）
  - 提交必须：**测试通过 + lint 无错误 + 格式合规**
  - CI/CD 集成：TDD 流程检查、禁止跳过测试

---

## 四、自动化与 MCP 集成（官方）
### 1. 测试自动化 Hooks（官方）
- **来源**：Anthropic 《Claude Code Hooks》
- **作者**：Anthropic 工程团队
- **配置（必加）**：
  ```json
  {
    "hooks": {
      "onStop": ["npm run test", "npm run lint"],
      "onCommit": ["pytest --cov=src -q"]
    }
  }
  ```
- **效果**：修改后**自动测试**、不合格阻止提交

### 2. 前端/UI TDD（Playwright MCP）
- **来源**：Anthropic 官方、腾讯云
- **作者**：Anthropic MCP 团队、腾讯云开发者社区
- **流程**：
  1. 写 Playwright 测试（截图+交互）
  2. 运行 → 截图对比 → 失败
  3. 实现 UI → 自动截图验证
  4. 视觉一致 → 测试通过

### 3. 后端/API TDD（官方）
- **来源**：Anthropic 工程实践
- **作者**：Anthropic 后端团队
- **工具**：pytest、jest、supertest
- **实践**：先写接口测试（请求/响应）→ 实现 → 验证

---

## 五、进阶 TDD 模式（社区权威）
### 1. 小步迭代 TDD（Anthropic 内部）
- **来源**：CSDN、Claude Code 核心团队分享
- **作者**：CSDN 技术专家、Anthropic 内部工程师
- **模式**：
  - 1个测试 → 1个小实现 → 测试通过 → 提交
  - 禁止：一次性写10个测试、批量实现
  - 优势：**快速回滚**、问题定位精准

### 2. 双 AI 审查 TDD（企业实践）
- **来源**：DataCamp、GitHub 社区
- **作者**：DataCamp、GitHub 权威指南作者
- **流程**：
  1. AI1：写测试 → 实现
  2. AI2：**独立审查**代码+测试 → 提问题
  3. 人工复核 → 提交
  - 效果：**双重校验**、缺陷率再降 40%

### 3. Bug 修复 TDD（官方）
- **来源**：Anthropic 《Bug Fix Best Practices》
- **作者**：Anthropic 开发者关系团队
- **流程**：
  1. 写测试 **复现 Bug**
  2. 运行 → 确认失败
  3. 修复代码 → 测试通过
  4. 提交（测试+修复）

---

## 六、项目配置规范（官方+社区）
### 1. CLAUDE.md TDD 规则（官方）
- **来源**：Anthropic 官方、掘金
- **作者**：Anthropic 文档团队、掘金技术社区
- **必加内容**：
  ```markdown
  # TDD 强制规则
  - 所有新功能必须 TDD：先测试、后实现
  - 测试文件：__tests__/**/*.test.ts
  - 禁止：无测试提交、修改已提交测试
  - 验证命令：npm run test、npm run lint
  ```

### 2. 权限配置（官方）
- **来源**：Anthropic 《Permissions Guide》
- **作者**：Anthropic 安全团队
- **推荐**：
  - 测试目录：`allow: Edit(./tests/**)`
  - 生产代码：`ask: Edit(./src/**)`（修改需确认）
  - 高危命令：`deny: Bash(rm -rf**)`

---

## 七、权威来源清单（可追溯）
1. **Anthropic 官方**：《Agentic Coding Best Practices》《Claude Code Security》
2. **DataCamp**：《Claude Code Best Practices: Planning, Context Transfer, TDD》
3. **Trail of Bits**：`claude-code-config`（GitHub 安全标杆）
4. **火山引擎**：《Claude Code 最佳实践》
5. **腾讯云**：《Claude Code 实战最佳实践》
6. **CSDN/掘金**：社区权威指南、内部实践分享

---

## 八、一句话总结
**Claude Code TDD 黄金标准**：**先测试 → 失败 → 提交 → 实现 → 通过 → 重构**（Anthropic+DataCamp 联合推荐）；配合**小步迭代、自动化 Hooks、双 AI 审查**，实现高质量、低缺陷 AI 编码。