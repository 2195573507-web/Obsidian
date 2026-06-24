---
title: "ServerHub 上线记录"
tags: ['ops', 'backup']
updated: 2026-06-24
---

# ServerHub 上线记录

- 时间：2026-06-22
- 主入口：`http://server.zmjjkkk.fun/`
- 健康：`http://127.0.0.1:8000/api/health`
- 备份目录：`/root/serverhub-migration/backup/20260622T013049Z`

## 变更内容
- 旧 `server-home` 首页已备份并由全新的 ServerHub 控制台替换。
- `server-home.service` 继续承载 8000，但启动目标已切到 `/opt/serverhub/backend.py`。
- 新控制台整合了已有 Web：监控、Agent、知识库、文件、记忆、任务、备份和安全状态。
- 写操作默认关闭，控制台保持只读。

## 关键文件
- `/opt/serverhub/backend.py`
- `/opt/serverhub/static/index.html`
- `/opt/serverhub/services.yml`
- `/etc/systemd/system/server-home.service`

## 回滚
```bash
systemctl stop server-home.service
cp -a /root/serverhub-migration/backup/20260622T013049Z/opt/server-home /opt/server-home
cp -a /root/serverhub-migration/backup/20260622T013049Z/systemd/server-home.service /etc/systemd/system/server-home.service
systemctl daemon-reload
systemctl start server-home.service
```

## 备份内容
- 原 `/opt/server-home`
- 原 systemd 单元与 Nginx 全量导出
- 当前端口和服务清单
