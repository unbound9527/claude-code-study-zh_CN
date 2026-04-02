# 教学文章：在 Claude Code 中连接微信（官方协议+开源工具，保姆级教程）
**作者**：技术派实战教程（整合自微信官方 ClawBot 文档、Johnixr、cc-connect 开源项目、CSDN/掘金实战）
**适用场景**：通过微信远程控制本地 Claude Code、随时随地写代码/查问题、消息实时互通、支持斜杠命令与文件操作

---

## 一、核心原理（看懂再动手）
目前 Claude Code 连接微信，**全部基于微信官方 ClawBot iLink 协议**（非逆向、合规安全）：
- 微信（ClawBot 聊天窗口） ↔ 桥接工具（Channel/MCP） ↔ 本地 Claude Code
- 数据流：微信发消息 → iLink 接口 → 桥接转发 → Claude Code 处理 → 原路返回微信
- 优势：**官方支持、稳定安全、满血能力、支持文件/命令/测试**

---

## 二、准备工作（必做）
1. **环境**
   - Node.js ≥18（安装：https://nodejs.org）
   - 本地已安装并激活 Claude Code（`claude --version` 正常）
   - 微信更新至 **8.0.7+**（支持 ClawBot 插件）
2. **验证**
   - 终端执行：`claude chat` → 能正常对话
   - 微信：我 → 设置 → 插件 → 看到 **微信 ClawBot**

---

## 三、方案1：MCP 通道（最稳定，非Claude模型不可用）
### 工具：claude-code-wechat-channel（GitHub 200+星，作者 Johnixr）
**来源**：iLink 协议、Anthropic MCP 规范

### 步骤1：扫码绑定微信
```bash
npx claude-code-wechat-channel setup
```
- 终端弹出二维码 → 微信扫码 → 点击「连接」
- 凭证保存：`~/.claude/channels/wechat/account.json`

### 步骤2：生成 MCP 配置
```bash
npx claude-code-wechat-channel install
```
- 当前目录生成 `.mcp.json` → 告诉 Claude Code 加载微信通道

### 步骤3：启动服务（关键）
```bash
claude --dangerously-load-development-channels server:wechat
```
- 看到「微信通道已启动」→ 成功

### 步骤4：微信使用
- 打开微信 → 找到 **ClawBot** 对话
- 发消息：`帮我写一个Python快速排序`
- Claude Code 终端收到 → 处理 → 自动回复微信


---

## 四、方案2：cc-connect（极简1行，新手友好）
### 工具：cc-connect（作者 chenhg5，开源）
**来源**：掘金实战、cc-connect 官方文档

### 步骤1：安装 cc-connect
```bash
npm install -g cc-connect@beta
cc-connect --version  # 验证
```

### 步骤2：绑定微信
```bash
cc-connect weixin setup
```
- 扫码 → 连接 → 自动配置

### 步骤3：启动服务
```bash
cc-connect start
```

### 步骤4：微信直接聊天
- ClawBot 发消息 → 自动转发 Claude Code

---

## 五、方案3：wechat-claude-code Skill（功能最全）
### 工具：wechat-claude-code（作者 Wechat-ggGitHub）
**来源**：GitHub 开源、Claude Code Skill 规范

### 步骤1：安装 Skill
```bash
git clone https://github.com/Wechat-ggGitHub/wechat-claude-code.git ~/.claude/skills/wechat-claude-code
cd ~/.claude/skills/wechat-claude-code
npm install
```

### 步骤2：扫码绑定
```bash
npm run setup
```

### 步骤3：后台启动
```bash
npm run daemon -- start
```

### 特色功能
- 实时进度推送、思考预览、中断任务、文件操作

---

## 六、微信常用命令（实战）
```
/new              # 新开会话
/view 文件        # 查看文件
/ls 目录          # 列目录
/run npm test     # 执行命令
/help             # 帮助
```

---

## 七、常见问题（避坑）
1. **扫码失败**
   - 微信版本太低 → 更新到 8.0.7+
   - 网络问题 → 切换 Wi-Fi
2. **收不到消息**
   - 重启服务：`cc-connect restart`
   - 检查 `.mcp.json` 配置
3. **权限问题**
   - macOS：允许终端控制
   - Windows：管理员运行

---

## 八、权威来源
1. **微信官方**：ClawBot iLink API 文档
2. **Johnixr**：claude-code-wechat-channel（GitHub）
3. **chenhg5**：cc-connect（掘金/GitHub）
4. **Wechat-ggGitHub**：wechat-claude-code Skill
5. **CSDN/掘金**：实战教程（2026.3）

---

## 九、总结
- **方案1**：官方 MCP → 最稳定、推荐
- **方案2**：cc-connect → 极简、新手
- **方案3**：Skill → 功能最全
- **全部合规**：基于微信官方协议，安全可靠