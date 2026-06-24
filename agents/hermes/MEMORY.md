---
title: "MEMORY"
tags: ['agent', 'config']
updated: 2026-06-24
---

Server `38.55.146.137`: system monitor `:9000`, Hermes Dashboard `:9100`, OpenClaw `127.0.0.1:18789`, shared memory `:9400` at `/opt/shared-agent-memory`; shared-memory CLI supports layers/expire/context debug/search.
§
创建 cron 定时任务默认以北京时间（UTC+8）为准，cron 表达式需转换为 UTC（BJT - 8h）。允许安排任意时段任务（包括 BJT 00:00-07:00），按用户指定执行。所有任务输出使用中文，多任务流水线标注 BJT/UTC。
§
Dashboards are systemd-managed: system-monitor `/opt/hermes-system-monitor` on `:9000`, management `/opt/hermes-dashboard` on `:9100`; restart by killing PID and letting systemd auto-restart; Hermes config at `~/.hermes/config.yaml`.
§
用户常用中转站：Hermes/OpenClaw 主要走 `sub.zmjjkkk.fun/v1`；OpenClaw `myproxy` 默认 `gpt-5.5`。备用 `pool.gptstore.club/v1` key 不匹配。OpenClaw 本机有 curl chat-completions 兜底补丁，`openclaw-gateway.service` 已持久设置。
§
中转站诊断：排查 OpenClaw/Hermes 连接问题时优先测当前中转站 `/models` 和逐模型探活，区分 401/502/503；不要主动建议换 `baseUrl`，除非当前中转站整体不可用。
§
长时间研究任务(7h+)：用多轨道 cron job（每10分钟一波、3轨道并行）比 delegate_task 更稳定，后者会被中断。用户明确要求学习任务只生成报告、不改任何代码。服务器项目全部是 FastAPI + 原生HTML/JS/CSS，无现代前端框架。
§
新服务器 `192.204.35.79`：OpenResty (Nginx) + MySQL:3306 + SSH:22，静态首页130字节。压测可达 7000并发/420 req/s/100%成功率。用户无SSH访问，仅远程操作。