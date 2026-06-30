---
title: 服务器 FAQ 与故障排查手册
type: faq
category: server
updated: 2026-06-30
tags: [server, faq, troubleshooting, docker, nginx, ssh, memory]
source: hermes-multi-agent-knowledge-expansion
---

# 服务器 FAQ 与故障排查手册

> 关联：[[MOC|知识库导航]]、[[server/服务器架构 Wiki|服务器架构 Wiki]]、[[server/服务器运维 Runbook|服务器运维 Runbook]]、[[reports/server-health-2026-06-30|服务器体检报告 - 2026-06-30]]

## 0. 总原则

### Q：服务器出问题时第一步做什么？

先判断影响范围，不要直接重启所有服务。

推荐顺序：

1. 确认用户访问的具体 URL；
2. 检查 DNS / HTTPS / 状态码；
3. 检查 Nginx；
4. 检查后端端口；
5. 检查 Docker / systemd；
6. 检查日志；
7. 检查 CPU、内存、磁盘、inode；
8. 最后再考虑单服务重启。

### Q：哪些事不能直接做？

- 不能无确认删除文件、volume、数据库、备份；
- 不能无确认重启 SSH、Nginx、数据库、整机；
- 不能无确认修改 UFW、证书、反代配置；
- 不能自动合并/删除 Hermes 或 OpenClaw 记忆；
- 不能执行 `docker system prune -a --volumes` 这类高风险清理。

## 1. 公网入口打不开

### Q：用户说网站打不开，怎么排查？

按“域名 → 公网 IP → 端口 → Nginx → 后端服务”的顺序。

```bash
dig +short A monitor.zmjjkkk.fun @1.1.1.1
curl -I --max-time 10 http://monitor.zmjjkkk.fun
curl -I --max-time 10 https://monitor.zmjjkkk.fun
systemctl status --no-pager nginx
sudo nginx -t
ss -tulpn | grep -E ':80|:443|19000|19100|19200|19300|19500|19600|19700'
sudo ufw status verbose
```

判断：

- DNS 不对：检查 A 记录；
- HTTP 不是 301：检查 Nginx server block；
- HTTPS 不是 200：检查证书、Nginx、upstream；
- 本机可访问公网不可访问：检查 UFW / 云安全组；
- 502：检查后端服务端口；
- 504：检查后端服务卡死或超时。

### Q：本机 curl 正常但公网打不开？

通常是防火墙、安全组、DNS、IPv4/IPv6、Nginx server_name 或公网链路问题。不要直接改服务代码。

## 2. HTTPS / 证书问题

### Q：证书过期或浏览器报错怎么办？

```bash
sudo certbot certificates
echo | openssl s_client -servername monitor.zmjjkkk.fun -connect monitor.zmjjkkk.fun:443 2>/dev/null | openssl x509 -noout -dates -issuer -subject
systemctl list-timers --all | grep -i certbot || true
```

常见原因：

- certbot timer 没跑；
- Nginx 没 reload，仍使用旧证书；
- DNS 指向旧服务器；
- ACME challenge 被 Nginx 配置或防火墙阻断；
- 多域名证书 SAN 不包含当前域名。

真实续期和改证书配置需确认。

## 3. Docker 容器异常

### Q：某服务不可用，Docker 怎么查？

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
docker ps -a
docker stats --no-stream
docker logs --tail 200 <container>
docker inspect <container> | grep -Ei 'RestartCount|OOMKilled|ExitCode' -A 3
```

常见原因：

- 容器退出或 Restarting；
- 环境变量缺失；
- 数据库/Redis 连接失败；
- volume 权限异常；
- 端口冲突；
- OOM；
- 镜像版本变化。

不要直接 prune、删 volume、重建数据库容器。

### Q：Nginx 502 但容器 running？

检查 upstream 路径：

```bash
sudo tail -n 100 /var/log/nginx/error.log
sudo nginx -T 2>&1 | grep -E 'server_name|proxy_pass'
ss -tulpn
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

常见原因：proxy_pass 端口写错、后端监听地址变了、容器健康检查失败、应用启动但内部 API 异常。

## 4. Atlas 入口问题

### Q：Atlas 某个入口不可用先查什么？

先对照域名和端口：

| 入口 | 端口 |
|---|---:|
| monitor | 19000 |
| agent | 19100 |
| notes | 19200 |
| files | 19300 |
| daily | 19500 |
| server | 19600 |
| collab | 19700 |

```bash
systemctl status --no-pager atlas-observe.service
systemctl status --no-pager atlas-agents.service
systemctl status --no-pager atlas-notes.service
systemctl status --no-pager atlas-files.service
systemctl status --no-pager atlas-daily.service
systemctl status --no-pager atlas-control.service
systemctl status --no-pager atlas-collab.service
ss -tulpn | grep -E '19000|19100|19200|19300|19500|19600|19700'
```

不要随意改端口；改端口必须同步 Nginx、systemd、文档和验证。

## 5. Shared Memory / Hermes / OpenClaw

### Q：Hermes 和 OpenClaw 的记忆能直接合并吗？

不能直接合并。记忆会影响未来行为，必须先做候选报告和人工确认。

安全流程：

1. 扫描现有记忆路径；
2. 复制到 Obsidian 分区；
3. 添加 source_path/frontmatter；
4. 生成 duplicates/source-map；
5. 人工确认 canonical；
6. 如需软链，再双向验证。

### Q：shared memory 异常怎么查？

```bash
curl -sS http://127.0.0.1:9400/health
systemctl status --no-pager shared-agent-memory.service
systemctl status --no-pager atlas-memory.service
systemctl status --no-pager atlas-recall.service
```

不要删除或覆盖 memory sqlite / markdown export。

## 6. SSH 扫描与安全

### Q：SSH 大量失败登录是否代表入侵？

不一定。公网 SSH 经常被扫描。重点是有没有未知成功登录。

```bash
journalctl -u ssh --since '24 hours ago' --no-pager | grep -Ei 'Failed password|Invalid user|Accepted|authentication failure' | tail -100
last -a | head -50
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication|pubkeyauthentication|port|maxauthtries|allowusers'
```

升级人工确认条件：

- 出现未知成功登录；
- 出现未知 SSH key；
- root 密码登录开启；
- 发现未知用户、未知 systemd 服务、未知 crontab；
- 出现疑似挖矿或异常外连。

## 7. systemd failed units

### Q：`systemd-networkd-wait-online.service` failed 严重吗？

不一定。它通常表示启动时等待网络在线超时，不等于当前网络不可用。若公网入口和 DNS 都正常，可先记录为低优先级。

```bash
systemctl status --no-pager systemd-networkd-wait-online.service
journalctl -u systemd-networkd-wait-online.service --since '24 hours ago' --no-pager
```

不要直接禁用网络相关服务。

### Q：`fwupd-refresh.service` failed 严重吗？

通常不影响业务，它与固件元数据刷新相关。除非确实要做固件维护，否则低优先级。

```bash
systemctl status --no-pager fwupd-refresh.service
journalctl -u fwupd-refresh.service --since '24 hours ago' --no-pager
```

不要在生产机器上无计划升级固件。

## 8. 系统资源异常

### Q：服务器变慢先看什么？

```bash
uptime
free -h
df -h
df -i
ps aux --sort=-%cpu | head -20
ps aux --sort=-%mem | head -20
docker stats --no-stream
journalctl --since '24 hours ago' --no-pager | grep -i 'out of memory\|oom' | tail -50
```

判断：

- load 高但 CPU 不高：可能是 IO wait；
- 内存低且 swap 高：内存压力；
- 磁盘满：优先查日志、Docker、数据库、上传目录；
- inode 满：大量小文件；
- Docker stats 高：定位容器而不是盲目 kill。

## 9. 磁盘满

### Q：能不能直接清理 Docker？

不能。先区分 images、containers、volumes、logs。

```bash
df -h
df -i
docker system df
du -sh /var/log/* 2>/dev/null | sort -h
```

危险操作需确认：

- `docker volume prune`
- `docker system prune -a --volumes`
- 删除数据库目录；
- 删除上传文件；
- 删除未知大文件。

## 10. 快速分流表

| 现象 | 首查 | 风险 |
|---|---|---|
| 域名打不开 | DNS / Nginx / UFW / HTTPS | 中 |
| HTTPS 报错 | 证书 / Nginx reload / DNS | 中高 |
| 502 | upstream / Docker / 端口 | 中 |
| 504 | 后端超时 / 资源 / DB | 中 |
| 容器 Restarting | logs / OOM / env / volume | 中高 |
| 磁盘满 | df / docker df / logs | 高 |
| SSH 大量失败 | auth 日志 | 中 |
| SSH 未知成功 | 登录日志 / key / 用户 | 高 |
| Hermes 记忆异常 | 路径 / 软链 / 覆盖 | 高 |
| Atlas 不可达 | systemd / 端口 / Nginx | 中高 |

## 11. 故障记录模板

```markdown
## 故障记录 - YYYY-MM-DD 服务名

### 现象
- 用户反馈：
- 影响范围：
- 首次发现：

### 快速判断
- DNS：
- HTTPS：
- Nginx：
- 后端端口：
- Docker/systemd：
- 资源：

### 排查过程
1. 
2. 
3. 

### 结论
- 根因：
- 临时修复：
- 长期修复：

### 是否需要人工确认
- [ ] 否
- [ ] 是，原因：
```
