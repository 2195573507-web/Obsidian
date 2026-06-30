---
title: Nginx 配置案例库
type: wiki
category: ops
updated: 2026-06-30
tags: [nginx, templates, reverse-proxy]
source: hermes-multi-agent-broad-expansion-wave3
---

# Nginx 配置案例库

> 关联：[[运维/Nginx 反向代理与 HTTPS 运维|Nginx 反向代理与 HTTPS 运维]]

## 1. 基础反代

```nginx
location / {
  proxy_pass http://127.0.0.1:3000;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
}
```

## 2. WebSocket

```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

## 3. 上传大小

```nginx
client_max_body_size 100M;
```

## 4. 长请求/流式 AI

```nginx
proxy_read_timeout 3600s;
proxy_send_timeout 3600s;
```

## 5. 安全

`server_tokens off`、限制敏感路径、不要暴露 `.env`、备份文件、内部目录。

## 6. 变更流程

备份配置 → `nginx -t` → reload → curl 验证 → 看 error.log。restart 需确认。
