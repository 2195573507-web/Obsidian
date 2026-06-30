---
title: Docker Compose 模板库
type: wiki
category: ops
updated: 2026-06-30
tags: [docker, compose, templates]
source: hermes-multi-agent-broad-expansion-wave3
---

# Docker Compose 模板库

> 关联：[[运维/Docker 服务部署与容器运维|Docker 服务部署与容器运维]]

## 1. Web App 模板

```yaml
services:
  app:
    image: myapp:1.0
    restart: unless-stopped
    env_file: [.env]
    ports: ["127.0.0.1:3000:3000"]
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
    logging:
      driver: json-file
      options: {max-size: "50m", max-file: "5"}
```

## 2. App + Postgres + Redis

使用内部网络，Postgres/Redis 不发布公网端口。数据卷显式命名并备份。

## 3. 健康检查

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
```

## 4. 风险

不要 `latest` 无版本控制；不要把 `.env` 提交 Git；不要 `down -v` 误删数据。
