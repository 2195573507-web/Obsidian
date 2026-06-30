---
title: Linux 系统管理与故障排查 SOP
type: wiki
category: ops
updated: 2026-06-30
tags: [linux, ops, troubleshooting]
source: hermes-multi-agent-knowledge-expansion
---

# Linux 系统管理与故障排查 SOP

> 关联：[[server/服务器运维 Runbook|服务器运维 Runbook]]、[[运维/Docker 服务部署与容器运维|Docker 服务部署与容器运维]]、[[运维/Nginx 反向代理与 HTTPS 运维|Nginx 反向代理与 HTTPS 运维]]

## 1. 登录后的第一组命令

```bash
date
hostname
whoami
pwd
uname -a
cat /etc/os-release
uptime
```

目的：确认机器、用户、系统版本、运行时长和负载。不要在未确认服务器身份时执行修改命令。

## 2. 资源排查

CPU：`uptime`、`top`、`ps aux --sort=-%cpu | head -20`。Load 长期高于 CPU 核数才算明显压力。

内存：`free -h`、`ps aux --sort=-%mem | head -20`。Linux 会用内存做 cache，应重点看 available 和 swap。

磁盘：`df -h`、`df -ih`、`du -xhd1 / | sort -h`。inode 满和空间满一样会导致服务异常。

网络：`ip a`、`ip r`、`ss -tulpn`、`curl -Iv https://domain`。

## 3. 日志排查

systemd 日志：

```bash
journalctl -u <service> -n 200 --no-pager
journalctl -u <service> --since '1 hour ago' --no-pager
journalctl -p err --since '24 hours ago' --no-pager
```

认证日志重点看 SSH failed/accepted。Nginx 重点看 502/504/upstream timed out。

## 4. 常见故障路径

网站打不开：DNS → UFW/安全组 → Nginx 80/443 → proxy_pass → 后端端口 → 应用日志。

SSH 断连：公网 IP → 安全组 → UFW → sshd 状态 → authorized_keys → fail2ban。

磁盘满：先定位，不要直接删；优先日志 vacuum、apt cache、确认无用构建缓存。Docker volume 清理需确认。

## 5. 安全边界

只读命令可直接执行；`rm`、`kill`、`systemctl restart/stop`、`ufw` 改规则、Docker prune、数据库写操作需确认。
