---
title: "2026-06-21 ServerHub 现状盘点与第一版上线记录"
tags: ['ops', 'backup']
updated: 2026-06-24
---

# 2026-06-21 ServerHub 现状盘点与第一版上线记录

## 访问地址

- 主入口：http://server.zmjjkkk.fun/
- 本地验证：http://127.0.0.1:8000/
- 旧子域保留：monitor.zmjjkkk.fun、agent.zmjjkkk.fun、notes.zmjjkkk.fun、files.zmjjkkk.fun

## 本次变更

- 新建全新前端目录：`/opt/serverhub/frontend/`
- 新建回滚文档：`/opt/serverhub/docs/rollback.md`
- `server-home.service` 仍作为入口服务，保留原只读聚合 API。
- `/` 改为返回 `/opt/serverhub/frontend/index.html`。
- `/serverhub-assets/` 挂载新前端静态资源。
- 禁用旧入口里的 `/api/ops/restart/{unit}` 写操作，当前返回 403。
- 未修改 Nginx 配置。
- 未移动或删除底层服务目录。

## 备份路径

- 迁移备份：`/root/serverhub-migration/backup/20260621-172856`
- 最新备份指针：`/root/serverhub-migration/latest-backup.txt`

备份包含：

- Nginx 配置：`nginx.conf`、`sites-available`、`sites-enabled`
- systemd unit 与 `systemctl cat` 输出
- 旧前端目录：server-home、hermes-dashboard/static、hermes-system-monitor/static、obsidian-knowledge-web/static、server-file-web/static
- 盘点文件：服务状态、监听端口、磁盘、`nginx -T`

## 验证结果

- `python3 -m py_compile /opt/server-home/main.py`：通过
- `systemctl restart server-home.service`：通过
- `curl http://127.0.0.1:8000/`：返回 ServerHub 新页面
- `curl -I http://127.0.0.1:8000/serverhub-assets/styles.css`：HTTP 200
- `curl http://127.0.0.1:8000/api/state`：10/10 服务 online
- `/api/ops/restart/nginx?confirm=RESTART`：HTTP 403，写操作已禁用
- `curl -H 'Host: server.zmjjkkk.fun' http://127.0.0.1/`：返回 ServerHub 新页面
- `nginx -t`：通过
- 相关服务状态：server-home、monitor、agent、notes、files、memory、openclaw、nginx 均 active

## 遗留风险

- 9000、9100、9200、9300、9400、8000 当前服务进程监听 `0.0.0.0`，虽然 Nginx 反代走本地地址，但仍建议后续用防火墙或服务绑定收敛公网直连面。
- `server-file-web` 的根目录配置为 `/`，第一版仍作为旧服务入口保留；后续应加强认证、只读边界和根目录白名单。
- `shared-agent-memory` 服务进程监听 `0.0.0.0:9400`，必须继续避免写接口公网暴露。
- 当前只接入 `server.zmjjkkk.fun`，尚未新增 `hub.zmjjkkk.fun`。
- Nginx 未新增 Basic Auth/rate limit，本次未动生产 Nginx 以降低中断风险。

## 回滚

```bash
BACKUP=/root/serverhub-migration/backup/20260621-172856
cp -a "$BACKUP/server-home/main.py" /opt/server-home/main.py
cp -a "$BACKUP/server-home/services.yml" /opt/server-home/services.yml
systemctl restart server-home.service
```

本次未修改 Nginx；如后续修改 Nginx，先恢复备份到临时位置对比，再执行 `nginx -t`，通过后 reload。

## 后续建议

1. 阶段 C：先加统一入口 Basic Auth / 限制敏感路径。
2. 收敛直连端口：优先处理 9400 memory、9300 files、9100 agent。
3. 新增 `hub.zmjjkkk.fun` server block，保留 `server.zmjjkkk.fun` 回滚入口。
4. 将 `/api/state` 拆成更明确的只读 ServerHub API：health、services、metrics、tasks、backups、security。
5. 为前端补浏览器截图验收和移动端检查。
