---
title: OpenClaw Session Memory Compact Report 2026-07-12
type: memory-compact-report
agent: openclaw
updated: 2026-07-12
tags: [openclaw, memory, session-compaction]
---

# OpenClaw Session Memory Compact Report - 2026-07-12 11:00 BJT

- 工作记忆：[[agents/openclaw/MEMORY|OpenClaw Working Memory]]
- 模式：`apply`
- 截止日期：`2026-06-28`（含当天及更早日记）
- 报告文件：`/root/obsidian-vault/记忆治理/openclaw-session-memory-compact-20260712T030000Z.md`

## 2026-06-23.md

- SHA256：`4ae3de7521cf01526e2ffd46d4572aa8c8ab2e72eeee2e83fe088402f8454784`
- 原始 bullet 数：`5`
- 备份目标：`/root/.openclaw/workspace/memory/backups/openclaw-session-memory-compact-20260712T030000Z/2026-06-23.md`
- 归档目标：`/root/.openclaw/workspace/memory/archive/2026-06-23.md`

### 关键决策

- 用户要求删除“晚上不允许做任务 / 避免北京时间 00:00-07:00 创建任务”的旧规则；该规则现在废弃。
- 以后如果用户指定凌晨或夜间执行任务，应按用户指定执行，不再自动避开夜间。
- OpenClaw + Hermes skills 每日整理任务已从北京时间 09:00 改为北京时间 03:00。

### 完成工作

- 无明显条目

### 发现的问题

- 无明显条目

### 待跟进

- 无明显条目

## 2026-06-24.md

- SHA256：`af4ea93828053bf711777eda6e9a5221272b07da0d82d67799cde2de237fb058`
- 原始 bullet 数：`39`
- 备份目标：`/root/.openclaw/workspace/memory/backups/openclaw-session-memory-compact-20260712T030000Z/2026-06-24.md`
- 归档目标：`/root/.openclaw/workspace/memory/archive/2026-06-24.md`

### 关键决策

- 依赖默认自动压缩（200K tokens 窗口），未配置主动修剪
- 用户确认：不调整
- 发现问题：矛盾记忆（夜间规则vs废弃决策）、千问相关记忆与"不再使用千问"冲突
- SOUL.md → symlink到shared（原614B默认模板→1,871B定制版）
- MEMORY.md/SHARED_MEMORY.md → 保持独立
- 用户确认：允许0-7工作，不要千问
- shared-agent-memory main.py：CLASSIFIER_MODEL改为None，classify跳过返回默认值，local-task全部走GPT fallback
- 规则写入shared/TOOLS.md记忆小节：必须显式提供kind/importance/sensitivity/tags/subject
- 新记忆不再依赖自动分类
- fact a53ceef2（禁止BJT 00:00-07:00）→ 已更新为"允许任意时段"
- decision 87728452/8348467d/995e0b1a（qwen路由）→ 标记为历史/已废弃
- Hermes MEMORY.md：已更新时间规则

### 完成工作

- 创建 `/root/obsidian-vault/agents/shared/` 目录
- IDENTITY.md → 新增symlink
- TOOLS.md → 新增symlink
- OpenClaw侧暂未接入（用户说不着急）
- OpenClaw MEMORY.md缺失待创建

### 发现的问题

- 无明显条目

### 待跟进

- 无明显条目

## 2026-06-25.md

- SHA256：`7f3fcd67108363aa0e8a683105b0d14e6a7742b1ddd1dce00075076fd4e35681`
- 原始 bullet 数：`32`
- 备份目标：`/root/.openclaw/workspace/memory/backups/openclaw-session-memory-compact-20260712T030000Z/2026-06-25.md`
- 归档目标：`/root/.openclaw/workspace/memory/archive/2026-06-25.md`

### 关键决策

- 无明显条目

### 完成工作

- Hermes monitor optimization completed in two rounds. Round 1: removed FastAPI GZipMiddleware, cached compact serialized JSON bytes for `/latest` and `/history`, fixed rollup fields (`sub2api_rx_kbps`, `sub2api_tx_kbps`, `pg_active_connections`, `pg_max_connect
- Continued roadmap implementation after user said “继续按照计划完成”. Updated new service ports to five digits per user request: `agent-recall-gateway` now `127.0.0.1:19410`; `provider-health-monitor` now `127.0.0.1:19420`. Old 9410/9420 no longer listening.
- Continued Atlas frontend feature completion. Added functional frontends: Observe now shows SVG history charts from `/api/system/metrics/history`; Agents has provider management cards and agent log/restart entry; Notes has graph/context/Markdown rendering; File
- 2026-06-26 roadmap continuation: Atlas Recall now has common recall templates (`/api/templates`, `/api/template/{name}`, `/api/template/{name}.md`) for openclaw-no-reply, provider-health, server-architecture, memory-policy, atlas-ui-archive. Fixed route orderi

### 发现的问题

- User asked for deep server benchmark excluding `/v1/chat/completions` pressure test (`3.3不做`). Final benchmark report saved at `/root/.openclaw/workspace/reports/server_deep_bench_20260625_145811.md`; raw log at `/root/.openclaw/workspace/reports/server_deep_b
- Server benchmark highlights: host `ser347176424328`, 8 vCPU Xeon E5-2699C v4, ~7.8 GiB RAM; network ~41–46 Mbps up/down; `/v1/models` all 200 with ~25 RPS at c1, ~79–83 RPS c5–20, degrading to ~60 RPS p95 ~3.9s at c200; no 429/502/503/504 in `/models` test.
- Added `agent-recall-gateway.service` at `http://127.0.0.1:9410`, aggregating `shared-agent-memory :9400` and `obsidian-knowledge-web :9200`. Interfaces: `/health`, `/api/recall?q=...&mode=default|debug|planning|all`, `/api/recall.md?q=...`. Purpose: one read-o
- Added `provider-health-monitor.service` at `http://127.0.0.1:9420`. Interfaces: `/health`, `POST /api/check-now`, `/api/latest`, `/api/history`. Safety: only probes `/v1/models`, no chat/completions pressure, reads local keys only for Authorization, never outp
- Provider Health Monitor now has `/api/summary?window=30&alert_after=3`, computing success_rate, consecutive_failures, alert flag. Policy remains report-only, no automatic route switch, `/v1/models` only. Current state: `sub-zmjjkkk` alert=true due to repeated 

### 待跟进

- 无明显条目

## 2026-06-26.md

- SHA256：`e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`
- 原始 bullet 数：`0`
- 备份目标：`/root/.openclaw/workspace/memory/backups/openclaw-session-memory-compact-20260712T030000Z/2026-06-26.md`
- 归档目标：`/root/.openclaw/workspace/memory/archive/2026-06-26.md`

### 关键决策

- 无明显条目

### 完成工作

- 无明显条目

### 发现的问题

- 无明显条目

### 待跟进

- 无明显条目

## 2026-06-27.md

- SHA256：`8c0d946328fc5d5807df984b68ab8ba46c6be20df591ff945122bf1d49280109`
- 原始 bullet 数：`34`
- 备份目标：`/root/.openclaw/workspace/memory/backups/openclaw-session-memory-compact-20260712T030000Z/2026-06-27.md`
- 归档目标：`/root/.openclaw/workspace/memory/archive/2026-06-27.md`

### 关键决策

- 无明显条目

### 完成工作

- 无明显条目

### 发现的问题

- Atlas roadmap work completed so far includes: Atlas Recall unified memory+Obsidian recall with schema/probe/redaction/timeouts; Atlas Control self-check, recall probe, QQ command API, remediation UI; Atlas Provider Watch alert throttle/ack/silence/notify previ
- QQ command coverage now includes low-risk executable/read-only commands `/状态`, `/查中转`, `/服务器巡检`, `/查记忆`, `/排队记忆`, `/排队巡检`, `/生成日报`, `/最近错误`, `/今日任务`; and high-risk report-only operation-card commands `/重启openclaw`, `/压测中转`, `/同步github`.
- Recent verification claims before compaction: `python3 -m py_compile` passed for changed Control/Collab Python files; `atlas-control.service` and `atlas-collab.service` active; `/api/error-summary` and `/最近错误` ok; high-risk QQ commands return report-only cards
- Provider alert/status claims in older memories, e.g. `sub-zmjjkkk 401`, are snapshots only; check `atlas-provider-watch` live before relying on them.
- Data upload decision: avoid flaky WebService ingest (`502 Bad Gateway` and SFT-only partial duplication observed). `configs/factory.yaml` has `upload.enabled: false`. Use SSH/scp sync via `scripts/maintain_training_link.sh` and `scripts/sync_to_training_server
- Important bug found around 16:35-17:11 UTC: remote was not effectively training because newly generated JSONL had inconsistent nested metadata types. Teacher output sometimes emitted `metadata.rubric`/`metadata.tags` as arrays or objects and sometimes strings.

### 待跟进

- Subagent roadmap gap audit concluded remaining safe next priorities are: unified log index/recent-errors enrichment, web search + research report queue job, task completion notification dry-run/opt-in, browser smoke tests, route/model recommendation dry-run. I
- Current durable preference: dangerous operations remain report-only/no automatic route switch/no chat-completions stress/no gateway restarts/config/systemd/nginx/cron writes/GitHub pushes without explicit confirmation. Hermes Provider and OpenClaw Provider are
- User goal: keep a continuous local code-distillation pipeline running for Hermes/sub teacher-generated SFT/DPO/Eval data, sync data to remote training server, and keep remote LLaMA-Factory LoRA SFT training looping continuously. Reports should be concise, abou
