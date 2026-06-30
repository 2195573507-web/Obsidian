---
title: Docker 数据卷备份专项
type: wiki
category: ops
updated: 2026-06-30
tags: [docker, backup, volume]
source: hermes-multi-agent-broad-expansion-wave4
---

# Docker 数据卷备份专项

> 关联：[[运维/Docker Compose 模板库|Docker Compose 模板库]]、[[运维/备份恢复演练|备份恢复演练]]

## 1. 盘点

```bash
docker volume ls
docker inspect <container> | grep -A 30 Mounts
docker system df -v
```

## 2. 备份原则

数据库优先用数据库工具备份；普通文件 volume 可 tar；备份前确认一致性；恢复必须演练。

## 3. 高风险

`docker compose down -v`、`docker volume rm`、`docker volume prune` 会删除数据。执行前必须确认。
