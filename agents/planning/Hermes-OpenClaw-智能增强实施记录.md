---
title: Hermes / OpenClaw 智能增强实施记录
tags: [agents, hermes, openclaw, implementation, roadmap]
created: 2026-06-25
updated: 2026-06-25
---

# Hermes / OpenClaw 智能增强实施记录

关联：[[Hermes-OpenClaw-智能增强路线图]]、[[MOC]]、[[服务器架构]]、[[shared-memory/exports/shared-memory|共享记忆导出]]

## 2026-06-25 阶段 1 启动

目标：按路线图从“Obsidian + shared-memory + 自动召回”开始，逐步完成智能增强。

### 已完成：Obsidian Knowledge Web RAG 只读接口

服务：`obsidian-knowledge-web.service`，端口 `9200`。

新增接口：

- `GET /api/search?q=...`
  - 搜索 path/title/tags/frontmatter/body。
  - 返回 score、snippet、headings、wikilinks。
- `GET /api/context?q=...`
  - 返回适合 agent 注入上下文的 compact blocks 和 markdown。
- `POST /api/reindex`
  - 手动重建内存索引。

安全边界：

- 只读 vault。
- 不删除、不移动、不改写历史笔记。
- 保持原 `/api/status`、`/api/files`、`/api/file`、`/api/graph` 和前端兼容。

备份：

- `/opt/obsidian-knowledge-web/backups/main.py.pre-rag-20260625T154822Z`
- `/opt/obsidian-knowledge-web/backups/index.html.pre-rag-20260625T154822Z`

### 已完成：Agent Recall Gateway 统一召回网关

服务：`agent-recall-gateway.service`，本机端口 `127.0.0.1:19410`。

作用：只读聚合：

- `shared-agent-memory :9400`
- `obsidian-knowledge-web :9200`

接口：

- `GET /health`
- `GET /api/recall?q=...&mode=default|debug|planning|all`
- `GET /api/recall.md?q=...`

安全边界：

- 只读。
- 输出做基础 secret-like redaction。
- 不改变 Hermes/OpenClaw 主回复路由。

### 已完成：Provider Health Monitor 初版

服务：`provider-health-monitor.service`，本机端口 `127.0.0.1:19420`。

接口：

- `GET /health`
- `POST /api/check-now`
- `GET /api/latest`
- `GET /api/history`

探测范围：

- 只探测 `/v1/models`。
- 不做 `/v1/chat/completions` 压测。
- 读取本机既有配置中的 key 仅用于 Authorization，不输出、不入库。

初次结果：

- `hermes-local-proxy`：200，约 119ms，10 models。
- `subus-zmjjkkk`：200，约 123ms，10 models。
- `sub-zmjjkkk`：401 Unauthorized，约 775ms。

### 后续队列

1. 将 `agent-recall-gateway` 接入 Dashboard 检索面板。
2. 为 Provider Health Monitor 增加 QQ 告警策略：连续失败 N 次才提醒，不自动切路由。
3. 增加 Hermes / OpenClaw 自愈巡检服务：先只报告。
4. 再推进任务队列中心、日志索引、QQ 指令菜单等阶段。

## 2026-06-25 阶段 2：ServerHub Next 只读接入

### 已完成：统一控制台服务卡片

`/opt/serverhub-next/services.yml` 新增：

- `recall-gateway`：Agent 统一召回，端口 `19410`，unit `agent-recall-gateway.service`。
- `provider-health`：中转健康监控，端口 `19420`，unit `provider-health-monitor.service`。

### 已完成：ServerHub Next 只读 API 代理

服务：`serverhub-next.service`，端口 `9600`。

新增接口：

- `GET /api/recall?q=...`
  - 代理到 `127.0.0.1:19410/api/recall`。
  - 用于 Dashboard 页面测试统一召回。
- `GET /api/provider-health`
  - 代理到 `127.0.0.1:19420/api/summary`。
  - 展示各中转目标最近状态、成功率、连续失败次数和 report-only 告警。

### 已完成：Provider Health Monitor 告警摘要

`provider-health-monitor.service` 新增：

- `GET /api/summary?window=30&alert_after=3`

策略：

- 连续失败达到阈值只标记 alert。
- 不自动切换模型路由。
- 不压测 chat/completions。
- 仅 `/v1/models` 探测。

当前摘要：

- `hermes-local-proxy`：正常。
- `subus-zmjjkkk`：正常。
- `sub-zmjjkkk`：连续 8 次 401，alert=true。

### 验证

- `python3 -m py_compile /opt/serverhub-next/backend.py` passed。
- `node --check` ServerHub inline JS passed。
- `serverhub-next.service` active。
- `/api/state` 200，10 个服务在线。
- `/api/provider-health` 200。
- `/api/recall?q=Hermes OpenClaw 自动召回` 200，返回 shared-memory 5 条 + Obsidian 5 条。

### 备份

- `/opt/serverhub-next/backups/backend.py.pre-recall-health-20260625T160424Z`
- `/opt/serverhub-next/backups/index.html.pre-recall-health-20260625T160424Z`
- `/opt/serverhub-next/backups/services.yml.pre-recall-health-20260625T160424Z`
- `/opt/serverhub-next/backups/backend.py.pre-provider-summary-*`
- `/opt/serverhub-next/backups/index.html.pre-provider-summary-*`
- `/opt/provider-health-monitor/backups/main.py.pre-summary-*`

## 2026-06-25 阶段 2b：重构新控制台替代 ServerHub

用户反馈：`serverhub我不想要了，能否重构一个`。

处理策略：

- 不删除、不停止原 `serverhub-next.service`，避免破坏现有入口。
- 新建独立控制台 `hermes-control-center.service`，端口使用五位数 `19600`。
- 后续验证满意后，再决定是否切换域名/入口。

### 新服务

- 路径：`/opt/hermes-control-center`
- Unit：`hermes-control-center.service`
- 端口：`19600`
- 首页：`http://127.0.0.1:19600/`

### 新接口

- `GET /api/health`
- `GET /api/overview`
- `GET /api/provider-summary`
- `GET /api/recall?q=...`
- `GET /api/logs/{unit}`

### 新前端组织

- 总览：关键服务、CPU/内存/磁盘/在线数。
- 服务：服务全图和日志入口。
- 知识召回：调用 `19410` 统一召回网关。
- 中转健康：调用 `19420` provider summary，report-only。
- 日志：受限 unit 白名单，只读 journalctl。

### 验证

- `python3 -m py_compile /opt/hermes-control-center/backend.py` passed。
- inline JS `node --check` passed。
- `hermes-control-center.service` active。
- `0.0.0.0:19600` listening。
- `/api/health` 200。
- `/api/overview` 200。
- `/api/provider-summary` 200。
- `/api/recall?q=OpenClaw 自动召回` 200，返回 shared-memory 5 条 + Obsidian 5 条。
- 首页 HTML 200，包含 `Hermes Control Center` 与 `新的统一控制台`。

备注：浏览器自动打开因当前 browser tool policy 阻止，已用 curl + JS/HTML 检查替代。
