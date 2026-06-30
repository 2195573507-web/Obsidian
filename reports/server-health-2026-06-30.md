---
title: "服务器体检报告 - 2026-06-30"
date: 2026-06-30T06:52:58Z
tags: [server, health-check, atlas, hermes, report]
source: hermes-multi-agent
---

# 服务器体检报告 - 2026-06-30

> 关联：[[服务器架构]]、[[MOC|知识库导航]]  
> 采集时间：2026-06-30 06:49-06:53 UTC（北京时间 2026-06-30 14:49-14:53）  
> 方式：3 个 agent 并行只读体检：基础资源、服务/容器、网络/公网入口。  
> 说明：未修改系统、未重启服务、未打印密钥/Token/环境变量。

## 1. 总体结论

当前服务器整体健康状态：**良好，可继续运行**。

- **资源压力**：CPU、内存、磁盘、inode 均处于正常区间。
- **业务服务**：Atlas、Hermes gateway、shared memory、Nginx、Docker 等关键服务运行中。
- **Docker**：4 个容器均 running 且 healthy。
- **公网入口**：7 个主入口域名 DNS 正确，HTTP 自动跳 HTTPS，HTTPS 均返回 200。
- **证书**：Let's Encrypt 证书剩余约 82-86 天，未临近过期。
- **安全信号**：SSH 日志存在外部扫描/爆破尝试迹象，但防火墙仅开放 22/80/443，入口策略较收敛。
- **需关注项**：2 个 systemd failed unit、SSH 扫描日志、sub2api/Postgres 瞬时 CPU 较高、部分 Atlas 端口直接监听 `0.0.0.0`。

## 2. 快速评分

| 维度 | 状态 | 判断 |
|---|---|---|
| CPU / Load | ✅ 正常 | 8 核，Load `1.25 / 0.92 / 0.85`，低于核心数 |
| 内存 | ✅ 正常 | 7.8Gi 总量，可用约 5.0Gi，Swap 使用约 123Mi |
| 磁盘 | ✅ 正常 | `/` 使用 56%，`/www` 使用 33% |
| inode | ✅ 正常 | `/` 9%，`/www` 10% |
| 关键服务 | ✅ 正常 | Docker/Nginx/Hermes/Atlas/Shared Memory 多数 active |
| Docker | ✅ 正常 | 4/4 running + healthy |
| 公网入口 | ✅ 正常 | 所有目标域名 HTTPS 200 |
| 证书 | ✅ 正常 | 剩余 82-86 天 |
| 日志安全 | ⚠️ 关注 | SSH 扫描/认证失败较多 |
| systemd failed | ⚠️ 低优先级 | `fwupd-refresh`、`systemd-networkd-wait-online` failed |

## 3. 基础资源体检

### 3.1 系统信息

| 项目 | 值 |
|---|---|
| 主机名 | `ser347176424328` |
| OS | Ubuntu 22.04 LTS |
| 内核 | `5.15.0-30-generic` |
| 架构 | `x86_64` |
| 虚拟化 | KVM |
| 运行时间 | 约 11 天 21 小时 |
| 时区 | `Etc/UTC` |
| NTP | active / synchronized |

### 3.2 CPU 与负载

| 项目 | 值 |
|---|---|
| CPU | Intel Xeon E5-2699C v4 @ 2.20GHz |
| 核心 | 8 核 |
| 线程 | 8 线程 |
| Load Average | `1.25 / 0.92 / 0.85` |

判断：当前负载明显低于 8 核容量，未见 CPU 饱和。

### 3.3 内存与 Swap

| 项目 | 值 |
|---|---:|
| 内存总量 | 7.8Gi |
| 已用 | 2.4Gi |
| 可用 | 5.0Gi |
| Buff/Cache | 4.5Gi |
| Swap 总量 | 4.0Gi |
| Swap 已用 | 123Mi |

判断：内存余量充足，Swap 仅少量使用。

### 3.4 磁盘与 inode

| 挂载点 | 总量 | 已用 | 可用 | 使用率 | inode 使用率 |
|---|---:|---:|---:|---:|---:|
| `/` | 40G | 23G | 18G | 56% | 9% |
| `/www` | 49G | 16G | 32G | 33% | 10% |
| `/boot/efi` | 105M | 6.1M | 99M | 6% | - |

判断：空间与 inode 均无近期耗尽风险。Docker overlay 位于 `/www` 底层磁盘，当前也正常。

### 3.5 资源占用较高进程快照

> 仅记录进程名，不记录完整命令行参数，避免泄露密钥。

CPU 排名前列：

| 进程 | CPU | 内存 RSS |
|---|---:|---:|
| `systemd-timedat` | 29.0% | 6MiB |
| `sub2api` | 17.4% | 126MiB |
| `openclaw` | 11.0% | 762MiB |
| `python` | 6.4% | 831MiB |
| `dockerd` | 6.0% | 54MiB |

内存排名前列：

| 进程 | RSS |
|---|---:|
| `python` | 831MiB |
| `openclaw` | 762MiB |
| `hermes` | 288MiB |
| `postgres` | 136MiB / 122MiB |
| `sub2api` | 126MiB |

判断：单进程内存占用未超过危险范围，整体内存仍充足。

## 4. 服务与容器体检

### 4.1 systemd failed units

当前发现 2 个 failed unit：

| Unit | 状态 | 影响判断 |
|---|---|---|
| `fwupd-refresh.service` | failed | 固件元数据/MOTD 刷新失败，通常不影响当前业务 |
| `systemd-networkd-wait-online.service` | failed | 等待网络在线超时，可能与实际网络管理方式不匹配；业务入口当前正常 |

建议：低优先级处理。可后续单独判断是否 reset failed、禁用 wait-online 或修复 fwupd metadata。

### 4.2 关键服务状态

运行中关键服务包括：

- `docker.service`
- `containerd.service`
- `nginx.service`
- `ssh.service`
- `hermes-gateway.service`
- `ollama.service`
- `shared-agent-memory.service`
- `atlas-agents.service`
- `atlas-collab.service`
- `atlas-control.service`
- `atlas-daily.service`
- `atlas-files.service`
- `atlas-intelligence.service`
- `atlas-memory.service`
- `atlas-notes.service`
- `atlas-observe.service`
- `atlas-provider-watch.service`
- `atlas-recall.service`

判断：核心服务链路完整，未见关键服务 down。

### 4.3 监听端口摘要

对外监听端口：

| 端口 | 进程/用途线索 |
|---:|---|
| 22 | SSH |
| 80 / 443 | Nginx |
| 8080 | Docker `sub2api` |
| 8420 | Docker `upstream-hub-standalone` |
| 19000-19700 | Atlas 系列服务 |
| 9400 | shared-agent-memory |

本地监听端口包括：`127.0.0.1:11434`、`127.0.0.1:18789`、`127.0.0.1:18888`、`127.0.0.1:19410`、`127.0.0.1:19420`、`127.0.0.1:19800`、`127.0.0.1:9501` 等。

关注点：UFW 当前只放行 22/80/443，虽然多个服务监听 `0.0.0.0`，公网侧理论上仍被 UFW 拦截；但从最小暴露原则看，后续可评估是否将内部服务绑定到 `127.0.0.1`。

### 4.4 Docker 状态

Docker 概览：

| 项目 | 值 |
|---|---:|
| 镜像 | 4 |
| 容器 | 4 |
| 运行中容器 | 4 |
| 本地卷 | 3 |
| Images 总大小 | 777.1MB |
| Containers 总大小 | 4.461MB |

容器状态：

| 容器 | 镜像 | 状态 | 健康 | 端口 |
|---|---|---|---|---|
| `sub2api` | `sub2api:local-upstream-monitor` | running | healthy | `0.0.0.0:8080->8080` |
| `upstream-hub-standalone` | `upstream-hub:local-clickfix` | running | healthy | `0.0.0.0:8420->8418` |
| `sub2api-postgres` | `postgres:18-alpine` | running | healthy | internal 5432 |
| `sub2api-redis` | `redis:8-alpine` | running | healthy | internal 6379 |

资源快照：

| 容器 | CPU | 内存 | 网络 I/O | 块 I/O |
|---|---:|---:|---:|---:|
| `sub2api` | 24.28% | 179.9MiB | 25.6GB / 21.6GB | 329MB / 331MB |
| `upstream-hub-standalone` | 0.00% | 34.77MiB | 78.8MB / 166MB | 113MB / 497MB |
| `sub2api-postgres` | 27.21% | 261.7MiB | 12.4GB / 9.88GB | 827MB / 17.4GB |
| `sub2api-redis` | 2.72% | 11MiB | 1.48GB / 4.02GB | 23.7MB / 1GB |

判断：容器健康；`sub2api` 与 Postgres 瞬时 CPU 偏高但未造成系统负载异常，建议观察趋势而不是立即处理。

## 5. 网络与公网入口体检

### 5.1 公网 IP 与 DNS

| 项目 | 值 |
|---|---|
| 当前出口公网 IPv4 | `38.55.146.137` |
| DNS A 记录 | 全部指向 `38.55.146.137` |
| AAAA | 未配置 |

检查域名：

- `monitor.zmjjkkk.fun`
- `agent.zmjjkkk.fun`
- `notes.zmjjkkk.fun`
- `server.zmjjkkk.fun`
- `files.zmjjkkk.fun`
- `daily.zmjjkkk.fun`
- `collab.zmjjkkk.fun`

### 5.2 Nginx 反代摘要

| 域名 | 后端 |
|---|---|
| `monitor.zmjjkkk.fun` | `http://127.0.0.1:19000` |
| `agent.zmjjkkk.fun` | `http://127.0.0.1:19100` |
| `notes.zmjjkkk.fun` | `http://127.0.0.1:19200` |
| `server.zmjjkkk.fun` | `http://127.0.0.1:19600` |
| `files.zmjjkkk.fun` | `http://127.0.0.1:19300` |
| `daily.zmjjkkk.fun` | `http://127.0.0.1:19500` |
| `collab.zmjjkkk.fun` | `http://127.0.0.1:19700` |

### 5.3 HTTP/HTTPS 连通性

| 域名 | HTTP | HTTPS |
|---|---:|---:|
| `monitor.zmjjkkk.fun` | 301 | 200 |
| `agent.zmjjkkk.fun` | 301 | 200 |
| `notes.zmjjkkk.fun` | 301 | 200 |
| `server.zmjjkkk.fun` | 301 | 200 |
| `files.zmjjkkk.fun` | 301 | 200 |
| `daily.zmjjkkk.fun` | 301 | 200 |
| `collab.zmjjkkk.fun` | 301 | 200 |

判断：公网入口正常，HTTP 到 HTTPS 跳转正常，HTTPS 后端可达。

### 5.4 证书

| 域名 | 到期时间 UTC | 剩余 |
|---|---:|---:|
| `monitor.zmjjkkk.fun` | 2026-09-20 16:17:12 | 约 82 天 |
| `agent.zmjjkkk.fun` | 2026-09-20 16:17:16 | 约 82 天 |
| `notes.zmjjkkk.fun` | 2026-09-20 16:17:19 | 约 82 天 |
| `server.zmjjkkk.fun` | 2026-09-20 16:17:23 | 约 82 天 |
| `files.zmjjkkk.fun` | 2026-09-25 04:05:33 | 约 86 天 |
| `daily.zmjjkkk.fun` | 2026-09-25 04:05:33 | 约 86 天 |
| `collab.zmjjkkk.fun` | 2026-09-25 04:05:33 | 约 86 天 |

判断：证书均有效，续期不紧急。

### 5.5 防火墙

UFW 状态：active。

| 默认策略 | 值 |
|---|---|
| 入站 | deny |
| 出站 | allow |
| 转发 | deny |
| 日志 | on / low |

放行端口：

- 22/tcp
- 80/tcp
- 443/tcp
- IPv6 同样放行 22/80/443

判断：公网入口策略收敛，符合 Web 服务最小开放要求。

## 6. 日志与安全线索

最近 24 小时高严重日志主要集中在 SSH：

| 次数 | 线索 |
|---:|---|
| 35 | `kex_exchange_identification: Connection closed by remote host` |
| 23 | `maximum authentication attempts exceeded` |
| 3 | `systemd-networkd-wait-online` 等待网络超时 |
| 2 | SSH 客户端非法协议标识 |
| 2 | `MaxStartups throttling` |
| 2 | `fwupd-refresh` 启动失败 |

判断：SSH 暴露在公网，存在常见扫描/暴力尝试。这是公网服务器常见现象，但应持续关注。

## 7. 风险分级与建议

### P0 / 立即处理

无。

### P1 / 建议近期处理

1. **SSH 暴力尝试防护**  
   当前日志显示扫描/认证失败较多。建议后续确认现有 SSH 配置后，考虑：
   - 禁止密码登录，仅允许密钥；
   - 禁止 root 直接登录；
   - 配置 fail2ban 或等价限速；
   - 如有固定管理 IP，可限制 SSH 来源。

2. **内部服务监听面收敛评估**  
   Atlas 和 shared memory 等服务当前部分监听 `0.0.0.0`，虽然 UFW 未放行其端口，但仍建议后续评估改为 `127.0.0.1`，由 Nginx 统一反代。

### P2 / 低优先级维护

1. **systemd failed units 清理/修复**
   - `fwupd-refresh.service`
   - `systemd-networkd-wait-online.service`

2. **sub2api / Postgres CPU 趋势观察**
   单次快照偏高不代表异常，建议结合业务量或监控趋势判断。

3. **证书续期巡检**
   当前剩余 82-86 天，不急；建议在剩余 30 天内自动/人工巡检一次。

## 8. 本次体检边界

本次只读体检没有做以下操作：

- 未修改 Nginx / systemd / Docker / UFW 配置；
- 未重启任何服务；
- 未查看或输出 `.env`、API key、token、cookie、证书私钥；
- 未执行 Docker prune、日志清理、文件整理等清理操作；
- 未进行入侵溯源，只做运行状态和入口健康检查。

## 9. 后续可选动作

如果需要继续，我建议按以下顺序做 **候选方案报告**，确认后再动手：

1. SSH 加固方案；
2. Atlas/Shared Memory 端口绑定收敛方案；
3. systemd failed units 修复方案；
4. Docker/sub2api 性能趋势观察方案；
5. 证书续期与 Nginx 配置巡检自动化方案。
