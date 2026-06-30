---
title: Docker 服务部署与容器运维
type: deep-wiki
category: ops
updated: 2026-06-30
tags: [docker, container, deployment]
source: hermes-deep-expansion
---

# Docker 服务部署与容器运维

> 关联：[[运维/Docker Compose 模板库|Docker Compose 模板库]]、[[运维/Docker 数据卷备份专项|Docker 数据卷备份专项]]

## 1. 核心对象

Image 是模板，Container 是运行实例，Volume/Bind Mount 是数据，Network 是通信边界，Compose 是编排文件。生产问题大多出在环境变量、端口、volume、网络、日志和资源限制。

## 2. Compose 原则

后端只给 Nginx 访问时端口绑定 `127.0.0.1`。数据库/Redis 不发布公网端口。镜像 tag 固定，不盲用 latest。日志配置 max-size/max-file。数据目录显式挂载并备份。

## 3. 诊断命令

```bash
docker ps -a
docker stats --no-stream
docker logs --tail 200 <container>
docker inspect <container>
docker system df
docker volume ls
```

## 4. 常见故障

Restarting：看 logs、ExitCode、OOMKilled、env、volume 权限、端口冲突。Running 但不可用：看 healthcheck、容器内监听、宿主端口、Nginx proxy_pass。

## 5. 数据安全

`docker compose down -v`、`docker volume rm`、`docker volume prune`、`docker system prune -a --volumes` 都可能删数据。数据库优先用数据库工具备份，不只 tar volume。

## 6. 发布

pull/up 前确认 compose 路径、备份、旧镜像 tag、迁移风险。发布后检查 ps、logs、health、Nginx、业务接口。
