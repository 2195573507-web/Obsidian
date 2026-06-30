---
title: systemd failed unit 处理手册
type: wiki
category: ops
updated: 2026-06-30
tags: [systemd, failed]
source: hermes-multi-agent-broad-expansion-wave4
---

# systemd failed unit 处理手册

> 关联：[[运维/systemd 服务管理与生产运行规范|systemd 服务管理与生产运行规范]]

## 处理步骤

```bash
systemctl --failed --no-pager
systemctl status --no-pager <unit>
journalctl -u <unit> --since '24 hours ago' --no-pager
systemctl cat <unit>
```

判断是否影响业务、是否一次性任务、是否 timer 触发、是否网络启动等待类噪音。

## 不要做

不要看到 failed 就 disable；不要 reset-failed 掩盖问题；先记录原因。
