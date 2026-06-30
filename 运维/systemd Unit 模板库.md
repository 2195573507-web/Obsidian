---
title: systemd Unit 模板库
type: wiki
category: ops
updated: 2026-06-30
tags: [systemd, templates]
source: hermes-multi-agent-broad-expansion-wave3
---

# systemd Unit 模板库

> 关联：[[运维/systemd 服务管理与生产运行规范|systemd 服务管理与生产运行规范]]

## 1. FastAPI 模板

```ini
[Service]
User=www-data
WorkingDirectory=/opt/app
EnvironmentFile=/opt/app/.env
ExecStart=/opt/app/.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
Restart=on-failure
RestartSec=5
```

## 2. One-shot 任务

```ini
[Service]
Type=oneshot
ExecStart=/opt/jobs/run.sh
```

## 3. Timer

```ini
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
```

## 4. 加固项

`NoNewPrivileges=true`、`PrivateTmp=true`、`ProtectSystem=full`、`ReadWritePaths=`。逐项启用并验证。
