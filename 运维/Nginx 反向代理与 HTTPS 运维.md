---
title: Nginx 反向代理与 HTTPS 运维
type: deep-wiki
category: ops
updated: 2026-06-30
tags: [nginx, https, reverse-proxy]
source: hermes-deep-expansion
---

# Nginx 反向代理与 HTTPS 运维

> 关联：[[运维/Nginx 配置案例库|Nginx 配置案例库]]、[[运维/Nginx 502 504 专项排查|Nginx 502 504 专项排查]]

## 1. 角色

Nginx 是公网入口边界：TLS 终止、HTTP 跳转、server_name 路由、反向代理、日志、限流、静态文件。后端服务尽量绑定 127.0.0.1，由 Nginx 暴露。

## 2. 配置结构

Ubuntu 常见路径：`/etc/nginx/nginx.conf`、`sites-available`、`sites-enabled`。修改前备份，启用站点用软链接。

## 3. 基础反代

```nginx
location / {
  proxy_pass http://127.0.0.1:3000;
  proxy_http_version 1.1;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
}
```

WebSocket 加 Upgrade/Connection。长流式 AI 请求调大 proxy_read_timeout。

## 4. HTTPS

证书需要 DNS 正确、80/443 可达、server_name 匹配、证书 SAN 包含域名、自动续期可用。续期后 Nginx reload 才会加载新证书。

## 5. 502/504

502：连不上 upstream。查 error.log、proxy_pass、端口、容器/systemd。
504：upstream 超时。查后端卡死、DB 慢查询、外部 API、超时配置。

## 6. 安全

隐藏版本、限制上传大小、禁止访问 `.env`/备份/隐藏文件、管理入口加鉴权、日志脱敏、必要时限流。

## 7. 变更 SOP

备份配置 → 修改 → `nginx -t` → reload → curl 本地和公网 → 看 error.log → 更新文档。不要未经测试直接 restart。
