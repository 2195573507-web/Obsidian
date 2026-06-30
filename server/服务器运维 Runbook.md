---
title: 服务器运维 Runbook
type: deep-wiki
category: server
updated: 2026-06-30
tags: [server, runbook, ops, nginx, docker]
source: hermes-deep-expansion
---

# 服务器运维 Runbook

> 关联：[[server/服务器架构 Wiki|服务器架构 Wiki]]、[[server/服务器 FAQ 与故障排查手册|服务器 FAQ 与故障排查手册]]、[[运维/服务器应急预案总表|服务器应急预案总表]]

## 1. 运维原则

所有运维动作遵循：先只读诊断，再判断风险，再生成方案，最后执行。删除、停服、重启、开放端口、修改 SSH/Nginx/证书/Docker volume/数据库均需确认。

## 2. 每日快速巡检

```bash
date
uptime
free -h
df -h
df -ih
systemctl --failed --no-pager
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
sudo ufw status verbose
```

检查点：

- Load 是否持续接近 CPU 核数；
- available 内存是否充足；
- `/` 和 `/www` 是否超过 80%；
- inode 是否异常；
- Docker 是否 unhealthy/restarting；
- systemd failed 是否新增；
- UFW 是否只开放预期端口。

## 3. 公网入口巡检

```bash
for d in monitor agent notes files daily server collab; do
  curl -I --max-time 10 https://$d.zmjjkkk.fun || true
done
```

预期：HTTP 301 到 HTTPS，HTTPS 200 或预期登录页。若 502，看 Nginx upstream；若超时，看 DNS/UFW/Nginx；若证书错误，看 certbot 和 server_name。

## 4. Nginx SOP

只读检查：

```bash
systemctl status --no-pager nginx
sudo nginx -t
sudo nginx -T 2>&1 | grep -E 'server_name|proxy_pass|ssl_certificate'
sudo tail -n 100 /var/log/nginx/error.log
```

变更流程：备份配置 → 修改 → `nginx -t` → reload → curl 验证 → 看 error.log。restart 比 reload 风险更高。

## 5. Docker SOP

```bash
docker ps -a
docker stats --no-stream
docker system df
docker logs --tail 200 <container>
```

判断容器是否健康不能只看 running，还要看 healthcheck、日志、HTTP health、数据库连接和重启次数。数据库容器、Redis、volume 操作必须特别谨慎。

## 6. systemd SOP

```bash
systemctl --failed --no-pager
systemctl status --no-pager <unit>
journalctl -u <unit> --since '24 hours ago' --no-pager
systemctl cat <unit>
```

一次性 timer 失败和业务服务失败不同。`fwupd-refresh`、`wait-online` 这类可能是低优先级，但新增 failed 要记录原因。

## 7. SSH 安全巡检

```bash
journalctl -u ssh --since '24 hours ago' --no-pager | grep -Ei 'Failed|Accepted|Invalid' | tail -100
last -a | head -50
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries'
```

大量 Failed 是公网常见扫描；未知 Accepted 是高风险。发现未知登录时先保留证据，不要清日志。

## 8. 证书巡检

```bash
sudo certbot certificates
echo | openssl s_client -servername monitor.zmjjkkk.fun -connect monitor.zmjjkkk.fun:443 2>/dev/null | openssl x509 -noout -dates -issuer -subject
```

证书小于 30 天进入关注，小于 14 天优先处理。续期失败先看 DNS、80/443、Nginx ACME 路径和 certbot timer。

## 9. 事故分级

| 级别 | 例子 | 响应 |
|---|---|---|
| P0 | 全站不可用、SSH 失联、数据库损坏 | 立即止血，保留证据 |
| P1 | 关键入口 502、容器重启循环 | 尽快处理，避免扩大 |
| P2 | 证书 30 天内、磁盘 >80%、SSH 扫描 | 当日处理或计划 |
| P3 | 文档、优化、低风险告警 | 排期 |

## 10. 变更前确认清单

- [ ] 影响域名？
- [ ] 影响端口？
- [ ] 影响数据？
- [ ] 是否有备份？
- [ ] 如何验证？
- [ ] 如何回滚？
- [ ] 是否涉及 SSH/UFW/Nginx/证书/Docker volume？
- [ ] 是否需要低峰期？

## 11. 巡检记录模板

```markdown
# 巡检记录 - YYYY-MM-DD
- 入口：
- Nginx：
- Docker：
- systemd failed：
- 磁盘：
- 内存：
- 证书：
- SSH：
- 风险项：
- 后续动作：
```
