# Claude.md 的作用

项目根目录的**项目说明书**，Claude Code 打开项目时自动读取。

---

## 核心作用

| 作用 | 说明 |
|------|------|
| 提供背景 | 让 AI 了解项目是什么、做什么 |
| 设置规则 | 编码规范、Git 流程、业务逻辑 |
| 减少重复 | 不用每次对话都解释一遍 |
| 保持一致 | 每次回答都符合项目上下文 |

---

## 文件位置

```
项目/
├── CLAUDE.md           ← 项目级记忆（必需）
├── .claude/
│   ├── settings.json   ← 用户级设置
│   └── skills/        ← 自定义技能
├── src/
└── README.md
```

### 优先级

当多个记忆文件同时存在时：

```
目录级 CLAUDE.md > 项目根目录 CLAUDE.md > 用户级 ~/.claude/CLAUDE.md
```

---

## Claude.md vs 对话说明

| 特性 | Claude.md | 对话中说明 |
|------|-----------|------------|
| 持久性 | 永久保存 | 会话结束消失 |
| 作用范围 | 当前项目 | 仅当前对话 |
| 更新方式 | 手动编辑 | "请记住..." 指令 |
| 团队共享 | 支持 | 不支持 |
| 内容容量 | 大量结构化信息 | 简短指令 |

**最佳实践**：Claude.md 记录稳定的项目信息，对话记忆处理临时的一次性信息。

---

## 完整模板

参考生产级项目的 CLAUDE.md 结构：

```markdown
# 项目名称

一句话描述项目做什么。

---

## 项目概况

### 项目类型
- Web 应用 / API 服务 / CLI 工具 / 移动应用

### 核心功能
1. 功能 A
2. 功能 B
3. 功能 C

### 目标用户
- 用户群体 A
- 用户群体 B

---

## 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 前端 | React 18 + TypeScript | 18.x |
| 后端 | Node.js + Express | 20.x |
| 数据库 | PostgreSQL + Redis | PG 15 |
| 部署 | Docker + K8s | latest |

---

## 项目结构

```
src/
├── api/           # API 路由层
├── services/     # 业务逻辑层
├── models/        # 数据模型
├── utils/         # 工具函数
└── config/        # 配置文件
```

---

## 代码规范

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `userName` |
| 常量 | UPPER_SNAKE | `MAX_RETRY` |
| 类名 | PascalCase | `UserService` |
| 文件名 | kebab-case | `user-service.ts` |
| React 组件 | PascalCase | `UserCard.tsx` |

### 编码规则

- 使用 `async/await`，不用 `.then()`
- 错误处理必须 `try/catch`
- 公共方法必须有 JSDoc 注释
- 禁止 `console.log`，用日志框架

### Git 规范

```
分支命名：
- feature/xxx     # 新功能
- fix/xxx         # Bug 修复
- hotfix/xxx      # 紧急修复
- refactor/xxx    # 重构

Commit 格式：
- feat: 新功能
- fix: 修复 Bug
- docs: 文档更新
- style: 代码格式
- refactor: 重构
- test: 测试
```

---

## API 规范

### 响应格式

```typescript
// 成功响应
{
  "success": true,
  "data": { ... },
  "message": "操作成功"
}

// 错误响应
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "用户不存在"
  }
}
```

### 状态码

| 状态码 | 含义 |
|--------|------|
| 200 | 成功 |
| 201 | 创建成功 |
| 400 | 参数错误 |
| 401 | 未授权 |
| 403 | 禁止访问 |
| 404 | 资源不存在 |
| 500 | 服务器错误 |

---

## 常用命令

```bash
# 安装依赖
npm install

# 开发模式
npm run dev

# 运行测试
npm test

# 运行测试（监听模式）
npm run test:watch

# 构建
npm run build

# 代码检查
npm run lint

# 代码修复
npm run lint:fix
```

---

## 环境变量

```bash
# .env.example（必需环境变量）
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-secret-key
API_RATE_LIMIT=100

# .env.local（本地覆盖，不提交）
PORT=3000
LOG_LEVEL=debug
```

---

## 工作流程

### 开发流程

```
1. 创建分支：git checkout -b feature/xxx
2. 开发功能
3. 运行测试：npm test
4. 提交代码：git commit
5. 推送：git push
6. 创建 PR
7. Code Review 通过后合并
```

### Bug 修复流程

```
1. 创建分支：git checkout -b fix/xxx
2. 复现问题
3. 修复 Bug
4. 添加测试
5. 提交并推送
6. 创建 PR
```

---

## 业务规则

### 用户相关
- 用户名唯一，不区分大小写
- 密码至少 8 位，需包含数字和字母
- 登录失败 5 次后锁定 30 分钟

### 权限相关
- 管理员可以管理所有资源
- 普通用户只能操作自己的资源
- 访客只有读取权限

---

## 已知问题和变通方案

| 问题 | 变通方案 |
|------|----------|
| 旧版 API 兼容性问题 | 使用 v2 API |
| 缓存穿透 | 双重检测锁 |
| 大文件上传超时 | 分片上传 |

---

## 依赖注意

| 依赖 | 版本要求 | 备注 |
|------|----------|------|
| Node.js | >= 18 | 需要 ESM 支持 |
| PostgreSQL | >= 14 | 需要 JSONB 支持 |
| Redis | >= 6 | 用于会话存储 |

---

## 联系方式

| 角色 | 联系人 | 职责 |
|------|--------|------|
| 技术负责人 | @lead | 架构决策 |
| 产品负责人 | @pm | 产品需求 |
| 运维支持 | @ops | 部署问题 |

---

## 创建步骤

1. 在项目根目录创建 `CLAUDE.md`
2. 复制上面的模板
3. 根据实际情况修改
4. 提交到版本控制
5. 团队成员共享

---

## 下一步

- 查看实际案例：[记忆功能](../L4-精通技巧/01-memory.md)
- 了解子代理：[子代理](../L4-精通技巧/02-subagent.md)
- 深入学习：[任务分解](01-break-tasks.md)
