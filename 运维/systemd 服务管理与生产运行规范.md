---
title: systemd 服务管理与生产运行规范
type: wiki
category: ops
updated: 2026-06-30
tags: [systemd, linux, ops]
source: hermes-multi-agent-knowledge-expansion
---

# systemd 服务管理与生产运行规范

> 关联：[[运维/Linux 系统管理与故障排查 SOP|Linux 系统管理与故障排查 SOP]]、[[server/服务器运维 Runbook|服务器运维 Runbook]]

## 1. systemd 作用

systemd 负责开机启动、进程守护、失败重启、日志归集、依赖管理、定时任务。生产服务优先用 systemd 或 Docker restart policy，不要裸 `nohup`。

## 2. 常用命令

```bash
systemctl status --no-pager <service>
systemctl --failed --no-pager
journalctl -u <service> -n 200 --no-pager
systemctl cat <service>
systemd-analyze verify /etc/systemd/system/<service>.service
```

start/stop/restart/enable/disable 是变更操作，需确认影响。

## 3. Unit 模板

```ini
[Unit]
Description=FastAPI App
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/myapp
EnvironmentFile=/opt/myapp/.env
ExecStart=/opt/myapp/.venv/bin/uvicorn app:app --host 127.0.0.1 --port 8000
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

## 4. 常见失败

`203/EXEC` 多为路径或权限错误；`No such file` 多为 ExecStart/WorkingDirectory 错；环境变量缺失看 EnvironmentFile；端口冲突看 `ss -tulpn`。

## 5. 生产规范

- 不要所有服务都用 root；
- ExecStart 用绝对路径；
- 明确 WorkingDirectory；
- 配置 RestartSec；
- 日志进 journal；
- 资源限制逐步加；
- 安全加固如 ProtectSystem 要逐项验证。
