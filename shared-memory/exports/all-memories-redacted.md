---
title: "All Shared Memories Redacted"
tags: ['memory', 'shared']
updated: 2026-06-24
---

# All Shared Memories Redacted

Total: 30

## Counts

- decision: 1
- fact: 10
- failure: 2
- noise: 2
- preference: 2
- service_state: 2
- success: 3
- temporary: 8


## decision

### AGENTS.md
- id: `d92ddf2f-1757-45a6-b76c-2507f1b4239d`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `imported, instruction, migration, openclaw`
- disabled: `False`; expires: ``; sensitivity: ``

---
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


## fact

### shared-memory
- id: `0428af60-f9aa-4b39-a855-e524bdaa05b2`
- scope/source: `shared` / `hermes`
- importance/confidence: `9` / `0.8`
- tags: `e2e, shared-memory`
- disabled: `False`; expires: ``; sensitivity: ``

端到端验证：Hermes 和 OpenClaw 现在通过 shared-agent-memory 服务共享长期记忆，服务端口 9400，导出文件挂载到双方工作区。

### MEMORY.md
- id: `ea89b21b-a295-48aa-80ac-c2542bff1c5e`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

服务器信息：公网 IP 38.55.146.137；系统监控在 `:9000`，Hermes Dashboard 在 `:9100`，Python venv 在 `/opt/hermes-agent/.venv/`。

### MEMORY.md
- id: `a53ceef2-e12b-4fdc-9a45-0590583c1225`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

创建 cron 定时任务时，所有时间应以北京时间（UTC+8）为准，cron 表达式需转换为 UTC（BJT - 8h）。禁止安排北京时间 00:00-07:00 之间的任务。所有任务输出使用中文。多任务流水线应清晰标注 BJT 和 UTC 双时区。

### MEMORY.md
- id: `b9adcd09-4bf4-4144-9cf6-1d5de2b7c01d`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

Hermes management dashboard architecture: backend at /opt/hermes-dashboard/backend.py (FastAPI, port 9100), frontend at /opt/hermes-dashboard/static/index.html. Reads state.db (~/.hermes/state.db) and config.yaml (~/.hermes/config.yaml). System monitor panel is separate at /opt/hermes-system-monitor/ (port 9000). Both use uvicorn + StaticFiles pattern. Config.yaml model section: model.default, model.provider, model.base_url. Server restart uses: kill existing PIDs on the port, then python3 backend.py as background process.

### MEMORY.md
- id: `91181f83-79e9-4a00-aaae-32da149cb236`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

Patch tool pitfall: when inserting a new code block before an existing decorated function, be careful with `old_string` — if you match the decorator line (e.g. @app.get("/api/status")), it will be replaced and the function below loses its decorator. Either include the decorator in new_string or use a broader old_string that includes the function signature, then add the decorator explicitly in new_string. Follow-up patches to restore decorators work fine.

### MEMORY.md
- id: `e417ebd4-238c-40f4-895a-fb752d6d3320`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

Hermes 本地可用工具已尽量全开：terminal, file, web, browser, vision, image_gen, tts, skills, todo, memory, session_search, clarify, delegation, cronjob, computer_use；video、video_gen、x_search、moa、homeassistant、spotify、yuanbao 仍未启用。

### MEMORY.md
- id: `9b27cd5a-99b7-44ec-9006-39e2092b9a62`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

每日3:00-5:00 BJT自动学习任务已创建(cron d4d3343581aa)，输出到 /opt/hermes-learning/YYYY-MM-DD/。学习范围：App/Web/代码/AI开源/嵌入式。输出projects.md+summary.md+archive.json。

### MEMORY.md
- id: `b9d2bad5-47c2-4287-8fff-706756be5ef6`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

用户使用中转站 pool.gptstore.club/v1（非直连 OpenAI/DeepSeek）：OpenClaw 和 Hermes 均通过此中转站。Hermes config: provider=openai, model=gpt-5.5。Dashboard 切换 provider 后须验证 model 段写入成功 — Hermes 可能覆写 model.provider。

### MEMORY.md
- id: `1be1c835-2449-4499-8ce9-733fe1b49076`
- scope/source: `shared` / `hermes`
- importance/confidence: `6` / `0.8`
- tags: `hermes, core, imported`
- disabled: `False`; expires: ``; sensitivity: ``

OpenClaw web_search is configured via `tools.web.search.enabled=true` and `tools.web.search.provider=duckduckgo` in `/root/.openclaw/openclaw.json`; DuckDuckGo is bundled and needs no API key. Memory indexing still needs a working embeddings provider; current myproxy `/embeddings` returned 503 for common embedding models.

### OpenClaw memory_search 已切到本地 memory-vector-index 检索，服务环境变量 OPENCLAW_MEMORY_SEARCH_COMMAND=1 已持久化。
- id: `9b9a0991-52b6-4fcf-9adb-04c7a1a7ed5e`
- scope/source: `shared` / `hermes`
- importance/confidence: `5` / `0.9`
- tags: `openclaw, memory`
- disabled: `False`; expires: ``; sensitivity: `public`

OpenClaw memory_search 使用本地内存向量索引进行检索，服务环境变量 OPENCLAW_MEMORY_SEARCH_COMMAND 已持久化。


## failure

### 2026-06-19.md
- id: `3b7ad1d4-ceec-415c-a36b-b2df4ed8f825`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: ``; sensitivity: ``

Recent JS bug: user reported `加载网关状态失败: Can't find variable: loadKnowledgeStatus`; fixed by adding missing `loadKnowledgeStatus` function to dashboard frontend to fetch/render `:9200` knowledge status. Dashboard did not require restart; refresh page.

### layer verification failure
- id: `f090472c-f251-4204-afed-d9c992bf2b5c`
- scope/source: `shared` / `hermes`
- importance/confidence: `1` / `0.8`
- tags: `layer-test`
- disabled: `True`; expires: ``; sensitivity: ``

分层记忆测试：failure entry should appear in debug retrieval.


## noise

### 2026-06-19.md
- id: `726c415c-da0d-48a1-bc09-eea404a298ae`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-06-28T03:10:54+00:00`; sensitivity: ``

## 2026-06-19 16:58 UTC — Durable session memory

### layer verification noise
- id: `7c31c3e4-8440-44ef-b8f4-fd0c366993a5`
- scope/source: `shared` / `hermes`
- importance/confidence: `1` / `0.8`
- tags: `layer-test`
- disabled: `True`; expires: `2026-06-22T03:11:13+00:00`; sensitivity: ``

分层记忆测试：noise entry should be excluded by default.


## preference

### USER.md
- id: `8f67509d-312f-4a7d-9fcf-84e9da60a15c`
- scope/source: `user` / `hermes`
- importance/confidence: `8` / `0.8`
- tags: `hermes, imported, migration, user_profile`
- disabled: `False`; expires: ``; sensitivity: ``

---
source_path: /root/.openclaw/workspace/USER.md
imported_at: 2026-06-19T13:19:45
category: agents/openclaw
---

# USER.md - About Your Human

_Learn about the person you're helping. Update this as you go._

- **Name:** 至秦
- **What to call them:** 至秦
- **Pronouns:** _(optional)_
- **Timezone:** UTC
- **Notes:**

## Context

_(What do they care about? What projects are they working on? What annoys them? What makes them laugh? Build this over time.)_

---

The more you know, the better you can help. But remember — you're learning about a person, not building a dossier. Respect the difference.

## Related

- [Agent workspace](/concepts/agent-workspace)
§
User prefers direct action and wants me to proceed without repeated confirmation when they say to continue or fix something.
§
用户偏好：喜欢我直接继续做完，不要反复确认；也会要求把任务推进到可用状态后再停下。

### USER.md
- id: `554a360f-09b8-4886-b659-c95b2e523b8e`
- scope/source: `user` / `openclaw`
- importance/confidence: `8` / `0.8`
- tags: `imported, migration, openclaw, user_profile`
- disabled: `False`; expires: ``; sensitivity: ``

---
source_path: /root/.openclaw/workspace/USER.md
imported_at: 2026-06-19T13:19:45
category: agents/openclaw
---

# USER.md - About Your Human

_Learn about the person you're helping. Update this as you go._

- **Name:** 至秦
- **What to call them:** 至秦
- **Pronouns:** _(optional)_
- **Timezone:** UTC
- **Notes:**

## Context

_(What do they care about? What projects are they working on? What annoys them? What makes them laugh? Build this over time.)_

---

The more you know, the better you can help. But remember — you're learning about a person, not building a dossier. Respect the difference.

## Related

- [Agent workspace](/concepts/agent-workspace)
- test-symlink: 2026-06-19T14:17:46+00:00


## service_state

### Hermes/OpenClaw shared memory integration
- id: `7731e550-13b3-45af-b82f-6667e7dab6fa`
- scope/source: `shared` / `hermes`
- importance/confidence: `9` / `0.8`
- tags: `hermes, openclaw, shared-memory`
- disabled: `False`; expires: ``; sensitivity: ``

Hermes has a dedicated shared-agent-memory skill at /root/.hermes/skills/shared-agent-memory/SKILL.md. For durable cross-agent recall/save, Hermes should use shared-memory-client against http://127.0.0.1:9400; OpenClaw indexes the shared export with local Ollama nomic-embed-text embeddings.

### (no subject)
- id: `ed6223e0-b265-4d64-9b1a-6d135c6f5ca7`
- scope/source: `shared` / `unknown`
- importance/confidence: `3` / `0.8`
- tags: `migration, service, shared-memory, test`
- disabled: `False`; expires: ``; sensitivity: ``

2026-06-20 shared-agent-memory service started and health check passed.


## success

### 2026-06-19.md
- id: `62eb7e04-3287-411f-8062-6eb9db53eddf`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: ``; sensitivity: ``

Important paths: OpenClaw workspace `/root/.openclaw/workspace/`; Hermes memories `/root/.hermes/memories/`; Hermes dashboard `/opt/hermes-dashboard/`; monitor `/opt/hermes-system-monitor/`; vault `/root/obsidian-vault/`.

### 2026-06-19.md
- id: `cfae8a49-e39d-4dba-92d2-5575ab3ffa25`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: ``; sensitivity: ``

Plans/docs created: `/root/obsidian-vault/plans/hermes-optimization-plan.md`, `/root/obsidian-vault/plans/obsidian-full-plan.md`; workspace also had `hermes-optimization-plan.md`, `knowledge-graph-plan.md`, `obsidian-full-plan.md`.

### layer verification success
- id: `f785b7bd-8e4b-474d-9ab5-848092b0b38c`
- scope/source: `shared` / `hermes`
- importance/confidence: `1` / `0.8`
- tags: `layer-test`
- disabled: `True`; expires: ``; sensitivity: ``

分层记忆测试：success entry should appear in debug/default retrieval.


## temporary

### 2026-06-19.md
- id: `ea72c393-4ea8-4569-be76-193a03fc06d0`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

User is 至秦. They manage Linux server `38.55.146.137` running two AI agents: Hermes and OpenClaw.

### 2026-06-19.md
- id: `8a31484a-edd3-4c51-810e-cfb0b4e611d1`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

Web services on server: `:9000` Hermes System Monitor, `:9100` Hermes Dashboard / agent console, `:9200` Obsidian knowledge web / FileBrowser target.

### 2026-06-19.md
- id: `747eab00-8889-4b64-95c0-2ccd34538bf7`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

User preference: prioritizes full functionality over security; server is headless Linux; has domain `zmjjkkk.fun`.

### 2026-06-19.md
- id: `4711d304-0abe-4ad3-a98e-7bb2ede77d9c`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

SSH public key provided by user: `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDaQZWZozb0mCR+11ytvutnWaMc7s5OWlBCm1JwaiZGs esp-server`.

### 2026-06-19.md
- id: `878c4506-8c11-44c2-8be0-3dbb0ad5755d`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

Obsidian shared-memory plan: use `/root/obsidian-vault/` as source of truth, shared via Git, with OpenClaw/Hermes memory symlinked into vault; user local Obsidian opens cloned vault.

### 2026-06-19.md
- id: `d109cd36-3ef6-4f0b-ad94-fa9b52d636bb`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

Prior work completed: Docker `hermes-openai` removed; SSH PubkeyAuthentication enabled; UFW port 22 opened; Hermes skills cleaned from 57 to 37; vault cleaned from 1271 files/14MB to 38 files/700K; optimization docs created.

### 2026-06-19.md
- id: `f44040bb-ec47-4ac6-a3aa-2fee96eb7d45`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

User paused Hermes Dashboard OpenClaw config integration earlier and said not to start Obsidian plan (`不启动`).

### 2026-06-19.md
- id: `8ac40162-6f7f-4994-9a7c-3a4024b52e1c`
- scope/source: `openclaw` / `openclaw`
- importance/confidence: `5` / `0.8`
- tags: `event, imported, migration, openclaw`
- disabled: `False`; expires: `2026-07-21T03:10:54+00:00`; sensitivity: ``

Recent web-health task: checked `:9000`, `:9100`, `:9200`; all services/APIs healthy. Dashboard CSS bug `--btn-hover` undefined was fixed. Token/cost trend API issue was a test-key mistake; data was normal.
