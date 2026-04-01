# 通知系统集成

## 本章目标

- 了解 Claude Code 的通知功能
- 知道如何配置桌面通知
- 能集成各种通知渠道

---

## 什么是通知系统？

**通知 = AI 完成重要任务后提醒你**

当你让 AI 做一件需要时间的事：
- 运行测试
- 编译代码
- 执行长时间任务

通知可以让你在任务完成后立刻知道，而不用一直盯着屏幕。

---

## 桌面通知

### Claude Code 内置通知

当 AI 执行长时间操作时，会自动弹出通知：

```
✅ 任务完成
[AI] 完成了代码审查
```

### 手动触发通知

```
"帮我运行测试，完成后通知我"
```

---

## 通知配置

### macOS 配置

```bash
# 允许通知
System Preferences → Notifications → Claude Code → 允许

# 或者使用命令行
osascript -e 'display notification "任务完成" with title "Claude"'
```

### Windows 配置

```powershell
# PowerShell 通知
[Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime] | Out-Null
$template = [Windows.UI.Notifications.ToastNotificationManager]::GetTemplateContent([Windows.UI.Notifications.ToastTemplateType]::ToastText02)
$textNodes = $template.GetElementsByTagName("text")
$textNodes.Item(0).AppendChild($template.CreateTextNode("Claude Code")) | Out-Null
$textNodes.Item(1).AppendChild($template.CreateTextNode("任务完成")) | Out-Null
$toast = [Windows.UI.Notifications.ToastNotification]::new($template)
[Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier("Claude").Show($toast)
```

---

## Hook 通知

使用 postToolUse hook 发送通知：

```bash
#!/bin/bash
# post-tool-notify.sh

TOOL_NAME=$1
RESULT=$2

# 检查是否是长时间操作
if [[ "$TOOL_NAME" == "Bash" && "$RESULT" == *"completed"* ]]; then
    # macOS
    osascript -e 'display notification "Bash 命令完成" with title "Claude"'

    # Linux
    # notify-send "Claude" "Bash 命令完成"

    # Windows
    # powershell -Command "[Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier('Claude').Show($toast)"
fi
```

---

## 集成外部通知服务

### Slack 通知

```bash
#!/bin/bash
# slack-notify.sh

curl -X POST -H 'Content-type: application/json' \
--data '{"text":"Claude Code 任务完成"}' \
https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

### 邮件通知

```bash
#!/bin/bash
# email-notify.sh

echo "Claude Code 任务完成" | mail -s "任务通知" your@email.com
```

### 钉钉/企业微信

```bash
#!/bin/bash
# dingtalk-notify.sh

curl 'https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{"msgtype": "text", "text": {"content": "Claude Code 任务完成"}}'
```

---

## 通知最佳实践

| 原则 | 说明 |
|------|------|
| 只通知重要的 | 不要所有操作都通知 |
| 简洁明了 | 通知内容要一眼看懂 |
| 及时 | 任务完成后立即通知 |
| 可操作 | 通知要包含下一步行动 |

---

## 跟做

1. 检查你的操作系统是否允许 Claude Code 发送通知
2. 配置一个简单的 postToolUse 通知 hook

---

## 小测验

**1. 通知的主要作用是？**
- A. 让 AI 更聪明
- B. 在任务完成时提醒你
- C. 加快执行速度

**2. 通知应该什么时候发送？**
- A. 所有操作
- B. 重要操作完成时
- C. 从不发送

---

## 答案

1-B | 2-B

---

## 下一步

- [附录：常见问题](../附录/常见问题.md)
