# 通知系统集成

## 通知作用

AI 完成重要任务后提醒你（运行测试、编译代码等）。

## 桌面通知

Claude Code 内置通知，长时间操作完成时自动弹出。

## Hook 通知

postToolUse Hook：
```bash
#!/bin/bash
if [[ "$TOOL_NAME" == "Bash" && "$RESULT" == *"completed"* ]]; then
    osascript -e 'display notification "完成" with title "Claude"'
fi
```

## 外部通知

**Slack**：
```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Claude Code 任务完成"}' \
  https://hooks.slack.com/services/XXX
```

**邮件**：
```bash
echo "任务完成" | mail -s "通知" your@email.com
```

---

**下一步**：[附录：常见问题](../附录/常见问题.md)
