---
title: 服务器架构 Wiki
type: deep-wiki
category: server
updated: 2026-06-30
tags: [server, architecture, atlas, hermes, nginx, docker]
source: hermes-deep-expansion
---

# 服务器架构 Wiki

> 关联：[[MOC|知识库导航]]、[[server/Atlas Hermes OpenClaw Shared Memory 专属架构|Atlas Hermes OpenClaw Shared Memory 专属架构]]、[[server/服务器运维 Runbook|服务器运维 Runbook]]、[[reports/server-health-2026-06-30|服务器体检报告 - 2026-06-30]]

## 1. 服务器定位

当前服务器是用户个人 Agent 基础设施和知识中枢的主节点，承载 Atlas Web 入口、Hermes 支持服务、OpenClaw 状态入口、Shared Memory、Obsidian vault、Docker 服务和 Nginx HTTPS 入口。

它不是单一 Web 服务器，而是一个多层系统：

```text
公网域名层 → Nginx/TLS → Atlas/Hermes/Docker 服务 → 记忆/知识/数据层 → Agent 自动化层
```

因此维护时要同时关注：入口可达性、内部端口、systemd 状态、Docker 健康、证书、记忆一致性、知识库同步、Agent 工具权限。

## 2. 公网入口

当前主入口域名：

| 域名 | 后端 | 用途 | 风险等级 |
|---|---|---|---|
| `monitor.zmjjkkk.fun` | `127.0.0.1:19000` | Atlas Observe / 监控 | 中 |
| `agent.zmjjkkk.fun` | `127.0.0.1:19100` | Agent 管理 | 高 |
| `notes.zmjjkkk.fun` | `127.0.0.1:19200` | 知识库 / Notes | 中高 |
| `files.zmjjkkk.fun` | `127.0.0.1:19300` | 文件入口 | 高 |
| `daily.zmjjkkk.fun` | `127.0.0.1:19500` | 日报/情报 | 中 |
| `server.zmjjkkk.fun` | `127.0.0.1:19600` | 服务器控制 | 高 |
| `collab.zmjjkkk.fun` | `127.0.0.1:19700` | 协作入口 | 中高 |

风险等级高的原因不是一定存在漏洞，而是这些入口如果无鉴权或权限过宽，可能暴露 Agent 操作、文件、服务器状态或控制能力。

## 3. 网络分层

```text
Internet
  ↓ DNS A: 38.55.146.137
UFW: 22/80/443
  ↓
Nginx: 80/443
  ↓ proxy_pass
127.0.0.1:19000-19700 / Docker published ports
  ↓
应用服务 / 数据 / 记忆 / 文件
```

UFW 当前只放行 22、80、443，这是正确的外部暴露策略。即使内部服务监听 `0.0.0.0`，公网侧仍受 UFW 限制；但从最小暴露面看，内部服务最好绑定 `127.0.0.1`。

## 4. 服务分层

### 4.1 Nginx 层

负责 TLS、HTTP→HTTPS、server_name 路由、反代、日志。任何公网入口异常，第一排查点是 Nginx 配置、证书、error.log 和 upstream。

### 4.2 Atlas 层

Atlas 是当前主要 Web 入口矩阵。各服务以 systemd 运行，端口集中在 19000-19700。新增、变更或下线 Atlas 服务时必须同步：systemd、端口、Nginx、证书、MOC、服务清单。

### 4.3 Hermes / OpenClaw 层

Hermes 负责多工具编排、cron、长任务；OpenClaw 负责多通道、浏览器自动化等能力。两者 provider 和记忆策略独立，不应自动同步。

### 4.4 Shared Memory 层

`shared-agent-memory:9400` 是跨 Agent 长期事实服务。`atlas-memory:19400` 和 `atlas-recall:19410` 是 Atlas 记忆/召回链路。稳定事实进入 shared memory 或 Obsidian，Hermes 工作记忆只保留当前有效状态。

### 4.5 Docker 层

Docker 当前承载 sub2api、upstream-hub、Postgres、Redis 等服务。数据库容器和 volume 是高风险对象，不得无确认 prune 或 down -v。

## 5. 数据路径

| 路径 | 用途 | 风险 |
|---|---|---|
| `/opt/hermes-agent/` | Hermes Agent | 高 |
| `/opt/shared-agent-memory/` | 共享记忆服务 | 高 |
| `/root/obsidian-vault/` | Obsidian 知识库 | 中高 |
| `/opt/atlas-*` | Atlas 应用 | 中高 |
| `/var/lib/docker` | Docker 根目录 | 高 |
| `/www` | 数据盘/服务数据 | 中高 |

路径风险主要来自误删、错误迁移、权限扩大、备份缺失和服务停机。

## 6. 运行健康基线

来自 2026-06-30 体检：CPU/内存/磁盘/inode 健康；Docker 4 个容器 healthy；HTTPS 主入口 200；证书剩余 82-86 天；存在 SSH 扫描；`fwupd-refresh` 与 `systemd-networkd-wait-online` failed 低优先级。

这些是历史快照。执行运维动作前必须 live check。

## 7. 变更原则

- 修改 Nginx 前备份配置并 `nginx -t`；
- 修改 SSH/UFW 前确认不会锁死；
- Docker volume 操作前确认备份；
- Agent 记忆合并前生成候选报告；
- 新域名必须同步 DNS、证书、Nginx、systemd、文档；
- 高风险入口必须有鉴权和日志；
- 任何“清理”先出报告，不自动删。

## 8. 建议后续补全

- Atlas 每个服务的 `/health` 接口；
- 每个服务的数据路径；
- 鉴权方式；
- Docker compose 实际目录；
- 证书 timer 状态；
- 备份恢复演练记录；
- 当前服务依赖图；
- Agent 工具权限矩阵。
