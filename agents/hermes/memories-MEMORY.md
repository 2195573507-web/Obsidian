---
source_path: /root/.hermes/memories/MEMORY.md
imported_at: 2026-06-19T13:19:46
category: agents/hermes
---

Server public IP: 38.55.146.137. Server monitoring dashboard running on port 9000 at /opt/hermes-system-monitor/ using FastAPI + psutil + SQLite + ECharts. UFW active with port 9000 open. Python venv at /opt/hermes-agent/.venv/.
§
创建 cron 定时任务时，所有时间应以北京时间（UTC+8）为准，cron 表达式需转换为 UTC（BJT - 8h）。禁止安排北京时间 00:00-07:00 之间的任务。所有任务输出使用中文。多任务流水线应清晰标注 BJT 和 UTC 双时区。
§
Hermes management dashboard architecture: backend at /opt/hermes-dashboard/backend.py (FastAPI, port 9100), frontend at /opt/hermes-dashboard/static/index.html. Reads state.db (~/.hermes/state.db) and config.yaml (~/.hermes/config.yaml). System monitor panel is separate at /opt/hermes-system-monitor/ (port 9000). Both use uvicorn + StaticFiles pattern. Config.yaml model section: model.default, model.provider, model.base_url. Server restart uses: kill existing PIDs on the port, then python3 backend.py as background process.
§
Patch tool pitfall: when inserting a new code block before an existing decorated function, be careful with `old_string` — if you match the decorator line (e.g. @app.get("/api/status")), it will be replaced and the function below loses its decorator. Either include the decorator in new_string or use a broader old_string that includes the function signature, then add the decorator explicitly in new_string. Follow-up patches to restore decorators work fine.
§
用户已放开所有权限：终端命令无需确认直接执行。approvals: mode=auto, timeout=300, cron_mode=allow, 所有confirm=false。项目：系统监控 /opt/hermes-system-monitor/(:9000) + Dashboard /opt/hermes-dashboard/(:9100)。配置 /root/.hermes/config.yaml，数据库 ~/.hermes/state.db。公网IP 38.55.146.137，BJT时区，venv /opt/hermes-agent/.venv/bin/python3。
§
每日3:00-5:00 BJT自动学习任务已创建(cron d4d3343581aa)，输出到 /opt/hermes-learning/YYYY-MM-DD/。学习范围：App/Web/代码/AI开源/嵌入式。输出projects.md+summary.md+archive.json。
§
用户使用中转站 pool.gptstore.club/v1（非直连 OpenAI/DeepSeek）：OpenClaw 和 Hermes 均通过此中转站。Hermes config: provider=openai, model=gpt-5.5。Dashboard 切换 provider 后须验证 model 段写入成功 — Hermes 可能覆写 model.provider。