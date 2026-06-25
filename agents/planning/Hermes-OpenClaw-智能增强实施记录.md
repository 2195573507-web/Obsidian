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

## 2026-06-25 阶段 2c：全 Web 重命名与 UI 重构

用户要求：服务器已有 Web 除 upstream/sub2api 外全部重新命名，不再以 Hermes/OpenClaw 命名；重新配置端口；删除旧前端/UI 并使用不同风格重构。

### 安全边界

未改动：

- `upstream.zmjjkkk.fun` → `127.0.0.1:8420`
- `subus.zmjjkkk.fun` / `apius.zmjjkkk.fun` → sub2api / upstream API 相关配置
- 本地中转 `127.0.0.1:18888`
- Docker 暴露 `8080/8420`
- OpenClaw gateway 运行时服务本身

### 新命名与端口

| 新服务 | 端口 | 说明 |
|---|---:|---|
| `atlas-control.service` | 19600 | 新统一控制台 |
| `atlas-observe.service` | 19000 | 系统观测/原监控 |
| `atlas-agents.service` | 19100 | Agent 状态面板 |
| `atlas-notes.service` | 19200 | Vault/RAG 知识库 |
| `atlas-files.service` | 19300 | 文件浏览 |
| `atlas-memory.service` | 19400 | shared-memory API |
| `atlas-recall.service` | 19410 | 统一召回 API |
| `atlas-provider-watch.service` | 19420 | 中转健康观察 |
| `atlas-daily.service` | 19500 | 每日总结 |
| `atlas-collab.service` | 19700 | 协作板 |

### 停用旧 Web 服务

已 disable + inactive：

- `hermes-control-center.service`
- `hermes-system-monitor.service`
- `hermes-dashboard-web.service`
- `obsidian-knowledge-web.service`
- `server-file-web.service`
- `shared-agent-memory.service`
- `agent-recall-gateway.service`
- `provider-health-monitor.service`
- `hermes-daily-summary-web.service`
- `agent-collab-web.service`
- `serverhub-next.service`
- `server-home.service`

### UI 重构

旧前端/UI 已归档备份，新前端统一改为 Atlas 风格：

- 明亮纸面背景
- 粗线框
- 硬阴影
- 高对比色块
- 不再使用旧 dashboard 暗色玻璃风格

已验证首页指纹：

- `19000` → `Atlas Observe`
- `19100` → `Atlas Agents`
- `19200` → `Atlas Notes`
- `19300` → `Atlas Files`
- `19500` → `Atlas Daily`
- `19600` → `Atlas Control`
- `19700` → `Atlas Collab`

### Nginx 入口迁移

非中转 Web 入口更新：

- `monitor.zmjjkkk.fun` → `127.0.0.1:19000`
- `agent.zmjjkkk.fun` → `127.0.0.1:19100`
- `notes.zmjjkkk.fun` → `127.0.0.1:19200`
- `server.zmjjkkk.fun` → `127.0.0.1:19600`

中转相关入口保持不变：

- `upstream.zmjjkkk.fun` → `127.0.0.1:8420`
- `subus.zmjjkkk.fun` → `127.0.0.1:8080`
- `apius.zmjjkkk.fun` 保持原配置

### 验证

全量验证日志：

- `/root/.openclaw/workspace/rebrand-web-final-20260625T164303Z.log`

结果：

- 10 个 `atlas-*` 服务全部 active。
- 旧 Web 服务全部 inactive/disabled。
- 旧端口 `7000/8000/9000/9100/9200/9300/9400/9500/9600` 已释放。
- 新端口 `19000/19100/19200/19300/19400/19410/19420/19500/19600/19700` 正常监听。
- 10 个本地 API 全部 HTTP 200。
- `nginx -t` passed。
- Nginx 本地 Host 复测：`monitor/agent/notes/server/upstream/subus/apius` 全部 HTTP 200。
- `atlas-recall` 召回测试 HTTP 200，返回 shared-memory 5 条 + Obsidian 5 条，errors=0。

### 备份

主要备份位置：

- `/root/.openclaw/workspace/backups/web-rebrand-20260625T162445Z`
- `/etc/nginx/sites-available/zmjjkkk.fun.pre-atlas-*`
- 各 `/opt/atlas-*/backups/` 中保存旧 UI 文件。

## 2026-06-25 阶段 2d：Atlas 前端功能补全

用户要求继续完成前端功能，并指出应学习 upstream UI 风格。

### 完成内容

- Atlas UI 继续保持 upstream 风格方向：顶部栏、居中内容、shadcn-like card/table/form/badge。
- `Atlas Observe` 补回历史曲线：轻量 SVG 折线图，使用 `/api/system/metrics/history?range=1h`。
- `Atlas Agents` 补回 Provider 管理卡片：baseUrl/models/enabled/API key 空值保留；保留 agent 重启/日志入口。
- `Atlas Notes` 补回 graph、context、Markdown 渲染和文件打开。
- `Atlas Files` 新增后端 `/api/preview` 和 `/api/raw`，前端补回路径摘要、quick paths、文本/图片预览。
- `Atlas Daily` 补回 Markdown 渲染和分类/文档列表阅读。
- `Atlas Control` 保留服务地图、中转、召回、日志入口。

### 安全修复

`Atlas Agents` 原 `/api/config/providers` 和 `/api/config` 会返回真实 API key。已修复：

- `/api/config/providers` 中 `apiKey` 永远为空，仅返回 `apiKeyMasked`。
- `/api/config` 的 `raw` 字段递归脱敏 key/token/secret/password/cookie/authorization。
- Provider 保存时 API key 留空不会覆盖原 key。

### 验证

- 7 个前端 inline JS `node --check` passed。
- `atlas-agents.service` 和 `atlas-files.service` 重启后 active。
- Provider/config 脱敏验证：无真实 `sk-*` key 泄露，`apiKey` 为空。
- 公网入口 `monitor/agent/notes/server` 均 HTTP 200。
- 功能接口验证：Observe history、Agents providers、Notes graph/context、Files preview、Daily summaries、Collab tasks 均 200。

## 2026-06-25 阶段 2e：Atlas 前端功能第二轮补齐

用户继续要求完成前端功能。本轮重点补旧前端的可操作细节。

### 完成内容

- `Atlas Observe`
  - 增加 `/api/system/metrics/detailed` 接入。
  - 增加 API / HTTP / Docker / DB / TCP 等诊断曲线卡片：`api_local_latency`、`api_apius_latency`、`http_5xx`、`docker_total_cpu`、`pg_active_connections`、`tcp_retrans_ratio`。
  - 增加 CPU per-core 卡片。
- `Atlas Agents`
  - Provider 管理补齐新增、保存、测试并拉取模型、设为活跃、启停、删除。
  - 增加最近 sessions 表。
  - 继续保持密钥脱敏和空 key 不覆盖。
- `Atlas Files`
  - `/api/raw?download=1` 增加 `Content-Disposition: attachment` 下载头。
  - 前端补充面包屑、文件类型图标、预览/下载/原始打开动作。
- `Atlas Daily`
  - 增加分类筛选、过滤计数和 Markdown 阅读布局。

### 验证

- 7 个 Atlas 前端 inline JS `node --check` 通过。
- `atlas-files`、`atlas-agents`、`atlas-observe`、`atlas-daily` 重启后 active；7 个 Atlas 服务均 active。
- 功能接口验证 200：Observe detailed/history、Agents providers、Files raw download、Daily summaries。
- Files raw download header 验证：`Content-Disposition: attachment`。
- 页面指纹验证：Observe `API / HTTP / Docker`、`CPU Cores`；Agents `新增 Provider`、`测试并拉取模型`；Files `Path Summary`/`下载`；Daily `分类筛选`/`Filtered`。
- `nginx -t` successful。
