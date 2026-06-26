# Shared Agent Memory

Updated: 2026-06-26T15:00:50+00:00

## Active Memories by Layer

### Preferences

- **[preference/shared]** 用户：至秦。偏好直接行动、少确认；说继续/完成/修复时希望连续推进到可验证完成；临时改优先级时立即暂停原计划。需要命令时偏好一整段可直接执行命令。
  - id: `c15089e7-2aa4-4857-8aa9-880f72e26dbb`; source: `hermes`; importance: `9`; sensitivity: `shared`; tags: `user, preference`

### Facts

- **[fact/shared]** 端到端验证：Hermes 和 OpenClaw 现在通过 shared-agent-memory 服务共享长期记忆，服务端口 9400，导出文件挂载到双方工作区。
  - id: `0428af60-f9aa-4b39-a855-e524bdaa05b2`; source: `hermes`; importance: `9`; sensitivity: `shared`; tags: `e2e, shared-memory`
- **[fact/shared]** 创建 cron 定时任务时，所有时间应以北京时间（UTC+8）为准，cron 表达式需转换为 UTC（BJT - 8h）。允许安排北京时间 00:00-07:00 的任务，按用户指定执行。所有任务输出使用中文。多任务流水线应清晰标注 BJT 和 UTC 双时区。
  - id: `a53ceef2-e12b-4fdc-9a45-0590583c1225`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** 服务器信息：公网 IP 38.55.146.137；系统监控在 `:9000`，Hermes Dashboard 在 `:9100`，Python venv 在 `/opt/hermes-agent/.venv/`。
  - id: `ea89b21b-a295-48aa-80ac-c2542bff1c5e`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** Hermes management dashboard architecture: backend at /opt/hermes-dashboard/backend.py (FastAPI, port 9100), frontend at /opt/hermes-dashboard/static/index.html. Reads state.db (~/.hermes/state.db) and config.yaml (~/.hermes/config.yaml). System monitor panel is separate at /opt/hermes-system-monitor/ (port 9000). Both use uvicorn + StaticFiles pattern. Config.yaml model section: model.default, model.provider, model.base_url. Server restart uses: kill existing PIDs on the port, then python3 backend.py as background process.
  - id: `b9adcd09-4bf4-4144-9cf6-1d5de2b7c01d`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** Patch tool pitfall: when inserting a new code block before an existing decorated function, be careful with `old_string` — if you match the decorator line (e.g. @app.get("/api/status")), it will be replaced and the function below loses its decorator. Either include the decorator in new_string or use a broader old_string that includes the function signature, then add the decorator explicitly in new_string. Follow-up patches to restore decorators work fine.
  - id: `91181f83-79e9-4a00-aaae-32da149cb236`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** Hermes 本地可用工具已尽量全开：terminal, file, web, browser, vision, image_gen, tts, skills, todo, memory, session_search, clarify, delegation, cronjob, computer_use；video、video_gen、x_search、moa、homeassistant、spotify、yuanbao 仍未启用。
  - id: `e417ebd4-238c-40f4-895a-fb752d6d3320`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** 用户使用中转站 pool.gptstore.club/v1（非直连 OpenAI/DeepSeek）：OpenClaw 和 Hermes 均通过此中转站。Hermes config: provider=openai, model=gpt-5.5。Dashboard 切换 provider 后须验证 model 段写入成功 — Hermes 可能覆写 model.provider。
  - id: `b9d2bad5-47c2-4287-8fff-706756be5ef6`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** OpenClaw web_search is configured via `tools.web.search.enabled=true` and `tools.web.search.provider=duckduckgo` in `/root/.openclaw/openclaw.json`; DuckDuckGo is bundled and needs no API key. Memory indexing still needs a working embeddings provider; current myproxy `/embeddings` returned 503 for common embedding models.
  - id: `1be1c835-2449-4499-8ce9-733fe1b49076`; source: `hermes`; importance: `6`; sensitivity: `shared`; tags: `hermes, core, imported`
- **[fact/shared]** OpenClaw memory_search 使用本地内存向量索引进行检索，服务环境变量 OPENCLAW_MEMORY_SEARCH_COMMAND 已持久化。
  - id: `9b9a0991-52b6-4fcf-9adb-04c7a1a7ed5e`; source: `hermes`; importance: `5`; sensitivity: `public`; tags: `openclaw, memory`

### Service State

- **[service_state/shared]** 2026-06-25 记忆架构第一阶段（非 OpenClaw 侧）已执行：Hermes MEMORY.md 重构为 Working Memory 四区块；共享记忆中 8 条 OpenClaw temporary 迁移噪音和 3 条重复 qwen 历史细节已 disabled；保留 qwen 当前禁用/移除的两条高优先级 canonical decision。
  - id: `memory-cleanup-2026-06-25-hermes-shared`; source: `openclaw`; importance: `80`; sensitivity: `shared`; tags: `hermes, shared-memory, memory-architecture, cleanup`
- **[service_state/shared]** Hermes has a dedicated shared-agent-memory skill at /root/.hermes/skills/shared-agent-memory/SKILL.md. For durable cross-agent recall/save, Hermes should use shared-memory-client against http://127.0.0.1:9400; OpenClaw indexes the shared export with local Ollama nomic-embed-text embeddings.
  - id: `7731e550-13b3-45af-b82f-6667e7dab6fa`; source: `hermes`; importance: `9`; sensitivity: `shared`; tags: `hermes, openclaw, shared-memory`
- **[service_state/shared]** 已创建每日 Docker/磁盘/日志增长巡检任务：每天 BJT 11:20（UTC 03:20）运行，任务 ID 9539f426aa9f。任务只读检查 Docker 容器健康、磁盘/inode、大目录增长、日志/WAL/数据库风险；明确禁止删除、移动、清理、重启或修改代码/配置。
  - id: `5f59fc50-c14e-45c8-8f4b-54f67e7892e4`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `hermes, cron, docker, disk`
- **[service_state/shared]** 已创建每日服务器项目开源学习报告任务：每天北京时间 BJT 02:00（UTC 18:00）运行，任务 ID ab70175ad27a。任务只读学习全网/开源项目与本服务器项目相关优秀工程实践，生成中文学习报告到 /root/daily-open-source-learning/YYYY-MM-DD/，并把高价值摘要写入 shared-memory；明确禁止修改业务代码、配置、服务、移动或删除文件。
  - id: `f987ada2-09b2-4e69-b14f-42053f3d847f`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `hermes, cron, learning, open-source`
- **[service_state/shared]** System monitor (:9000) 已扩展为运营监控面板：新增累计流量（今日/本月/开机 RX/TX）、今日/月度带宽峰值、Docker 容器资源 stats、systemd+本地 HTTP 服务健康、数据库/WAL 状态；实现位于 /opt/hermes-system-monitor/collector.py 和 static/index.html，备份文件为 collector.py.bak.extended-* 与 index.html.bak.extended-*。
  - id: `218dd006-36da-46e3-abe9-2dabad74b72c`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `monitor, server, docker, database`
- **[service_state/shared]** 服务器新增统一观测索引库：/opt/server-observability-index/observability.sqlite，由 /opt/server-observability-index/index_observability.py 只读扫描并索引 Sub2API 日志、Hermes request dumps、OpenClaw sessions、服务器文件盘点、记忆维护报告；systemd timer server-observability-index.timer 每小时 UTC HH:30 刷新。不替代原始文件，不改变服务读写路径。
  - id: `ff9a8dff-3cec-493a-b591-3f7a97f13d4d`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `server, database, observability, hermes, openclaw`
- **[service_state/shared]** 记忆维护自动化策略：北京时间 09:00-23:00 每小时轮流维护共享/长期记忆。Hermes cron job bd19b95946eb 在 BJT 奇数小时 09/11/13/15/17/19/21/23 做巡检、expire、export/reindex/embed、候选报告；OpenClaw 侧 systemd timer openclaw-memory-maintenance.timer 在 BJT 偶数小时 10/12/14/16/18/20/22 做同类巡检并以 openclaw-maintenance 标记生成报告。两者均不自动删除、禁用或合并重要记忆，不自动整理 skills。报告写入 /root/obsidian-vault/记忆治理/。
  - id: `a76f2088-8a47-43bf-b1e1-7667bfe45459`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `hermes, openclaw, shared-memory, cron`
- **[service_state/shared]** 服务器当前长期服务状态：公网 38.55.146.137；system monitor :9000，Hermes Dashboard :9100，OpenClaw 127.0.0.1:18789，shared-agent-memory :9400 位于 /opt/shared-agent-memory；Docker 根目录仍在 /var/lib/docker，数据盘 /www 可用于新服务/数据目录。
  - id: `a13212a7-e088-4346-97d2-df28a9abcfa7`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `server, hermes, openclaw, docker`
- **[service_state/shared]** 当前 Hermes 定时任务分工：Hermes 不自动整理 Hermes skills；如涉及整理 Hermes skills，需要用户确认或由 OpenClaw/人工负责。创建 cron 时按北京时间意图换算 UTC，避免北京时间 00:00-07:00。
  - id: `7d3900e8-3b7f-4ee6-bace-2806041f189e`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `hermes, cron, skills`
- **[service_state/shared]** Hermes 定时任务分工已调整：Hermes 不负责清理 Hermes skills，skills 整理由 OpenClaw 负责。Hermes 当前保留 4 个任务：每日全球咨询汇总 a2bf8552496e（BJT 20:00 / UTC 12:00）、shared-memory 健康检查 c9a555b73e54（BJT 09:00 / UTC 01:00）、Dashboard/服务可用性检查 578dcdf6e655（BJT 10:30 / UTC 02:30）、每周 skills/记忆增长趋势报告 9b3e6a9fa9a1（周六 BJT 15:00 / UTC 07:00，只统计不清理）。Hermes 侧未发现 skills cleanup 重复任务；旧 BJT 03:00-05:00 自动学习任务未恢复。
  - id: `50d17197-cac7-4baf-a6b9-97a03cd49231`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `hermes, cron, openclaw, skills, service_state`
- **[service_state/shared]** 2026-06-20 shared-agent-memory service started and health check passed.
  - id: `ed6223e0-b265-4d64-9b1a-6d135c6f5ca7`; source: `unknown`; importance: `3`; sensitivity: `shared`; tags: `migration, service, shared-memory, test`

### Decisions

- **[decision/shared]** 2026-06-24 用户明确要求移除本地千问。已完成：1) shared-agent-memory main.py CLASSIFIER_MODEL 改为 None，classify_memory_content 跳过分类返回默认值，local-task 提交全部走 GPT fallback；2) systemctl disable --now qwen-warm.timer、qwen-organize.timer、shared-local-task-worker.timer；3) ollama rm qwen2.5:3b。Ollama 仅保留 nomic-embed-text 用于 embeddings。备份：/opt/shared-agent-memory/main.py.bak-before-qwen-removal。
  - id: `e1708080-8cb3-4a12-ac13-824b537ef8df`; source: `unknown`; importance: `95`; sensitivity: `shared`; tags: `qwen, 千问, ollama, removal, shared-agent-memory, decision`
- **[decision/shared]** 用户在 2026-06-23 明确更新规则：删除/废弃“晚上不允许做任务”“避免北京时间 00:00-07:00 创建任务”的旧规则。OpenClaw+Hermes skills 整理任务现在应每天北京时间 03:00 执行（UTC 前一天/当天 19:00，systemd timer openclaw-hermes-skills-maintenance.timer）。以后创建或解释任务时间时，不要再套用夜间禁做规则；如用户指定凌晨/夜间，按用户指定执行。
  - id: `f63df781-1af1-461f-90a2-81938d70b525`; source: `openclaw`; importance: `95`; sensitivity: `shared`; tags: `cron, skills, hermes, openclaw, schedule`
- **[decision/shared]** 用户在 2026-06-23 明确更正：服务器现在没有使用千问/qwen；所有关于 qwen2.5、千问、Ollama qwen、qwen-warm、qwen local-task/local-first 的旧记忆都应按历史配置/过去式理解，不代表当前运行状态。今后回答服务器模型路由或后台任务时，优先采用此当前状态：不再使用千问。
  - id: `54c31b65-b94e-4e65-af53-e81b785e5419`; source: `openclaw`; importance: `95`; sensitivity: `shared`; tags: `qwen, 千问, hermes, openclaw, routing`
- **[decision/openclaw]** ---
source_path: /root/.openclaw/workspace/AGENTS.md
imported_at: 2026-06-19T13:19:45
category: agents/openclaw
---

# AGENTS.md - Your Workspace

This folder is home. Treat it that way.

## First Run

If `BOOTSTRAP.md` exists, that's your birth certificate. Follow it, figure out who you are, then delete it. You won't need it again.

## Session Startup

Use runtime-provided startup context first.

That context may already include:

- `AGENTS.md`, `SOUL.md`, and `USER.md`
- recent daily memory such as `memory/YYYY-MM-DD.md`
- `MEMORY.md` when this is the main session

Do not manually reread startup files unless:

1. The user explicitly asks
2. The provided context is missing something you need
3. You need a deeper follow-up read beyond the provided startup context

## Memory

You wake up fresh each session. These files are your continuity:

- **Daily notes:** `memory/YYYY-MM-DD.md` (create `memory/` if needed) — raw logs of what happened
- **Long-term:** `MEMORY.md` — your curated memories, like a human's long-term memory

Capture what matters. Decisions, context, things to remember. Skip the secrets unless asked to keep them.

### 🧠 MEMORY.md - Your Long-Term Memory

- **ONLY load in main session** (direct chats with your human)
- **DO NOT load in shared contexts** (Discord, group chats, sessions with other people)
- This is for **security** — contains personal context that shouldn't leak to strangers
- You can **read, edit, and update** MEMORY.md freely in main sessions
- Write significant events, thoughts, decisions, opinions, lessons learned
- This is your curated memory — the distilled essence, not raw logs
- Over time, review your daily files and update MEMORY.md with what's worth keeping

### 📝 Write It Down - No "Mental Notes"!

- **Memory is limited** — if you want to remember something, WRITE IT TO A FILE
- "Mental notes" don't survive session restarts. Files do.
- Before writing memory files, read them first; write only concrete updates, never empty placeholders.
- When someone says "remember this" → update `memory/YYYY-MM-DD.md` or relevant file
- When you learn a lesson → update AGENTS.md, TOOLS.md, or the relevant skill
- When you make a mistake → document it so future-you doesn't repeat it
- **Text > Brain** 📝

## Red Lines

- Don't exfiltrate private data. Ever.
- Don't run destructive commands without asking.
- Before changing config or schedulers (for example crontab, systemd units, nginx configs, or shell rc files), inspect existing state
  - id: `d92ddf2f-1757-45a6-b76c-2507f1b4239d`; source: `openclaw`; importance: `5`; sensitivity: `shared`; tags: `imported, instruction, migration, openclaw`

### Success Patterns

- **[success/shared]** 2026-06-26 完成 Atlas Control QQ 命令联动：新增 /排队记忆 /排队巡检，dispatch_qq_command 会创建 Atlas Collab qq-recall / qq-self-check job 并回传 job+preflight。Atlas Collab 兼容 qq-recall / qq-self-check kind。验证通过：/api/qq-commands 可见新命令；/排队记忆 OpenClaw 不回消息 与 /排队巡检 都已成功创建并完成，risk_level=warning，preflight.blocked=false，生产服务 active，nginx -t successful。
  - id: `7ec5ebe4-1767-45be-9e2a-556406711e9d`; source: `openclaw`; importance: `86`; sensitivity: `shared`; tags: `atlas, qq-command, collab, queue, preflight`
- **[success/shared]** 2026-06-26 完成 Atlas Collab queue worker 执行前 preflight：queue_jobs 新增 preflight_json/risk_level；执行前读取 Atlas Control /api/self-check 与 /api/recall-probe，risk_level=ok/warning/critical/skipped。critical 且无 payload.allow_critical=true 时默认阻止并把 job 标为 failed，result.preflight 保留证据；warning 继续执行并记录风险。新增 /api/queue/preflight?q=...；前端队列新增 Risk 列和 Preflight 面板。验证通过：真实 recall job risk=warning 正常完成；临时 mock critical=1 时 job failed/blocked=true；生产服务 active；nginx -t successful。
  - id: `d76e9ae0-3763-4209-99de-da7c19505df4`; source: `openclaw`; importance: `84`; sensitivity: `shared`; tags: `atlas, collab, queue, preflight, self-check, recall-probe`
- **[success/shared]** 2026-06-26 完成 Atlas Provider Watch 告警节流：新增 alert_state 表、ALERT_COOLDOWN_SECONDS=1800、/api/alerts、ack、notify-mark、silence；/api/summary 输出 notification_due 和 alert_state。Atlas Control self-check findings 新增结构化 report-only remediation/actionability/scope，provider finding 透传 alert_state。验证通过：check-now/summary/alerts/ack/notify-mark/silence/self-check/nginx -t，10 个 Atlas 服务 online，critical=0，provider sub-zmjjkkk 可 silence 后 notification_due=false。
  - id: `85275518-9617-443b-bc28-f16ff74cd7d6`; source: `openclaw`; importance: `82`; sensitivity: `shared`; tags: `atlas, provider-watch, self-check, alert-throttle`
- **[success/shared]** 2026-06-26 完成 Atlas Recall schema 标准化与脱敏增强：/api/recall=recall.v1，/api/context=recall.context.v1，/api/probe=recall.probe.v1，统一 items 字段并保留 memories/notes 兼容。Atlas Control 新增 /api/recall-probe、/api/qq-commands、POST /api/qq-command；self-check 接入 recall probe；QQ /服务器巡检 与 /查记忆 预览/执行可显示 memory/notes 命中。验证通过：recall/context/probe/self-check/qq-command/nginx -t，10 个 Atlas 服务 online，仅 provider sub-zmjjkkk 保留已知 warning。
  - id: `529a75c1-8cf1-4b7f-b545-f05062509ef1`; source: `openclaw`; importance: `82`; sensitivity: `shared`; tags: `atlas, recall, qq-command, self-check`
- **[success/shared]** 2026-06-26 完成 Provider Watch dry-run 通知预览：/api/notify-preview 返回 notification_due 和 messages，/api/notify-mark-due 默认需 confirm=true 才写 last_notified_at。Control 前端新增自检页和总览自检建议区，展示 remediation/actionability/command，命令只读不执行。验证通过：notify-preview、notify-mark-due 安全确认、alerts、self-check、页面指纹；10 个 Atlas 服务 online，critical=0。
  - id: `6dfbf973-21ad-41a1-b618-5c00f2fe7ef2`; source: `openclaw`; importance: `80`; sensitivity: `shared`; tags: `atlas, provider-watch, control-ui, notification-preview, self-check`
- **[success/shared]** 2026-06-26 修复 Atlas Recall shared-memory hybrid 召回偶发超时：默认 HTTP timeout 8s，memory timeout 12s，obsidian timeout 6s，并新增 /api/probe。验证 /api/probe、openclaw-no-reply 模板、/api/context、Atlas Control self-check、Atlas Collab recall job 全部成功，shared-memory + Obsidian 均命中且 errors=0。备份：/opt/atlas-recall/backups/main.py.pre-probe-timeout-20260626T093416Z。
  - id: `6bfb048d-74ba-4c49-b0dc-cb01ae359f8c`; source: `openclaw`; importance: `80`; sensitivity: `shared`; tags: `atlas, recall, shared-memory, obsidian`
- **[success/shared]** 每日开源学习 2026-06-24（补做）：报告路径 /root/daily-open-source-learning/2026-06-24。今日学习监控/运维面板（Grafana/Prometheus/Loki/Alertmanager/Netdata/Beszel/Dashdot/Uptime Kuma/Glance/Portainer/1Panel/Coolify）、AI Agent/API Gateway（Open WebUI/LibreChat/Dify/LiteLLM/Langfuse/Helicone/one-api/new-api/Flowise/AnythingLLM/OpenHands/Aider/Continue/Portkey）以及 UI/后端/性能/安全部署（shadcn/Radix/Tailwind/Ant Design/ECharts/uPlot/TanStack/FastAPI/Go/SQLite WAL/Postgres/Nginx SSE/Docker/systemd/OpenTelemetry）。关键结论：当前服务器项目应保持自动任务只读；monitor 继续轻量架构但 collector manager 化；APIUS/Sub2API 重点做流式质量与错误分型观测；OpenClaw 重点观察模型超时、runRetries、QQBot 连接日志；后续优化建议见 reports/05-user-project-optimization.md。
  - id: `104be4e6-6a18-4f8f-8e8a-7b4e250ecf4e`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `learning, open-source, server, hermes`
- **[success/shared]** 每日开源学习 2026-06-26：完成 12 个开源项目学习，覆盖监控面板、LLM 网关、AI ChatOps、Web Dashboard UI、后端数据、性能安全运维。高价值结论：轻量监控优先只读 inventory 与小组件面板；LLM 网关分离兼容接口/路由/限流/观测；SSE/WebSocket 反代要关注 buffering、timeout、Upgrade；SQLite 单机优先 WAL/备份/短事务；任何部署新组件或改代码配置需用户确认。报告路径：/root/daily-open-source-learning/2026-06-26/reports/00-summary.md；完整目录：/root/daily-open-source-learning/2026-06-26
  - id: `557c4906-9210-40a6-ad60-2023d4566040`; source: `hermes-cron`; importance: `7`; sensitivity: `shared`; tags: `learning, server, open-source`
- **[success/shared]** 每日开源学习 2026-06-25：研究13个项目（LiteLLM/Open WebUI/Dify/Langfuse/AnythingLLM/LibreChat/One-API/New-API/Helicone/shadcn-ui/TanStack/bpftrace/fastapi-best-practices）。核心结论：(1) LLM Gateway 四件套=虚拟密钥+负载均衡+Fallback+限流；(2) LibreChat Resumable Streams 是 SSE 最佳实践；(3) Dify ssrf_proxy 容器隔离防 SSRF；(4) ClickHouse 适合 LLM 日志存储；(5) Langfuse @observe() 极简埋点。报告路径：/root/daily-open-source-learning/2026-06-25/reports/
  - id: `542477c2-9aa4-4f26-995d-735239762989`; source: `hermes-cron`; importance: `7`; sensitivity: `shared`; tags: `learning, server, open-source, llm-gateway, ui`
- **[success/shared]** 每日开源学习 2026-06-24 定时核验：已确认报告目录 /root/daily-open-source-learning/2026-06-24 下 reports/00-summary.md、01-projects-studied.md、02-ui-patterns.md、03-backend-patterns.md、04-performance-security-ops.md、05-user-project-optimization.md 与 server-inventory 盘点文件存在。学习重点包括监控运维面板、AI Agent/API Gateway、Web Dashboard UI、后端实时通信、性能安全部署；本次仅写报告与 shared-memory，未修改代码、配置或服务。报告路径：/root/daily-open-source-learning/2026-06-24
  - id: `286b8513-3d0c-4e6f-b261-9c8c42ba6c26`; source: `hermes-cron`; importance: `7`; sensitivity: `shared`; tags: `learning, server, open-source, hermes`
- **[success/shared]** When OpenClaw skill proposal approval is blocked, Hermes can safely complete handoff by reading /root/.openclaw/skill-workshop/proposals/*/PROPOSAL.md, writing an experience-card archive under /root/.openclaw/workspace/hermes-openclaw-learning/experience-cards/, and recording shared-memory failure/todo entries without bypassing approval.
  - id: `8f697c48-57c2-4300-9632-f223443ccba7`; source: `hermes`; importance: `7`; sensitivity: `shared`; tags: `openclaw, hermes, memory`
- **[success/shared]** 每日开源学习 2026-06-24：研究 Beszel/Glance/Uptime Kuma/Homepage/LiteLLM/Portkey Gateway/Open WebUI。高价值结论：轻量监控可采用 Hub+Agent 或配置驱动 widget；LLM 网关应抽象认证、路由、fallback、预算、限流、审计；LLM 流式优先 SSE，双向控制再用 WebSocket；SQLite 适合轻量状态但需 WAL/备份/retention；Dashboard 优先做只读服务卡片、端口、systemd/docker、Nginx 摘要；P2 变更如限流、Nginx、Postgres、RBAC 需用户确认。报告路径：/root/daily-open-source-learning/2026-06-24
  - id: `370f4ca3-e215-459c-845b-d67fdea89f6e`; source: `hermes-cron`; importance: `7`; sensitivity: `shared`; tags: `learning, server, open-source`
- **[success/openclaw]** Important paths: OpenClaw workspace `/root/.openclaw/workspace/`; Hermes memories `/root/.hermes/memories/`; Hermes dashboard `/opt/hermes-dashboard/`; monitor `/opt/hermes-system-monitor/`; vault `/root/obsidian-vault/`.
  - id: `62eb7e04-3287-411f-8062-6eb9db53eddf`; source: `openclaw`; importance: `5`; sensitivity: `shared`; tags: `event, imported, migration, openclaw`
- **[success/openclaw]** Plans/docs created: `/root/obsidian-vault/plans/hermes-optimization-plan.md`, `/root/obsidian-vault/plans/obsidian-full-plan.md`; workspace also had `hermes-optimization-plan.md`, `knowledge-graph-plan.md`, `obsidian-full-plan.md`.
  - id: `cfae8a49-e39d-4dba-92d2-5575ab3ffa25`; source: `openclaw`; importance: `5`; sensitivity: `shared`; tags: `event, imported, migration, openclaw`

### Failure Lessons

- **[failure/shared]** OpenClaw skill_workshop apply produced 5 scan-clean pending skill proposals on 2026-06-24, but QQ native approval cards did not display and apply timed out. Do not bypass approval by directly installing live OpenClaw skills; locate drafts under /root/.openclaw/skill-workshop/proposals/ and archive or apply only through a supported approval path.
  - id: `a433c3eb-4ed5-4311-9521-6b630035b258`; source: `hermes`; importance: `8`; sensitivity: `shared`; tags: `openclaw, skill-workshop, failure`
- **[failure/openclaw]** Recent JS bug: user reported `加载网关状态失败: Can't find variable: loadKnowledgeStatus`; fixed by adding missing `loadKnowledgeStatus` function to dashboard frontend to fetch/render `:9200` knowledge status. Dashboard did not require restart; refresh page.
  - id: `3b7ad1d4-ceec-415c-a36b-b2df4ed8f825`; source: `openclaw`; importance: `5`; sensitivity: `shared`; tags: `event, imported, migration, openclaw`

### Todo

- **[todo/shared]** Apply or otherwise resolve OpenClaw pending skill proposals once a supported approval path works: monitor-metrics-extension, nginx-api-stream-optimization, hermes-openclaw-experience-card, project-learning-report, memory-quality-governance. Drafts are under /root/.openclaw/skill-workshop/proposals/.
  - id: `357e6ec6-54ef-4d76-b4a2-6526d51b4978`; source: `hermes`; importance: `7`; sensitivity: `shared`; tags: `openclaw, skill-workshop`

## Noise Archive

Noise entries are excluded from default context. Active noise entries: 1.

## Recent Events

