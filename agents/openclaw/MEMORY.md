---
title: "MEMORY"
type: note
category: "agents"
updated: 2026-07-05
managed_by: Hermes
tags: [vault, agents]
---

# OpenClaw Working Memory

> 上限 50 条 | 超限由 Agent 主动压缩合并
> 最后维护：2026-06-27

## 用户偏好
- [preference] 默认中文回复，计划类回答尽量简洁，报告类可以详细 (updated: 2026-06-27)
- [preference] 用户偏好直接行动、少确认；说“继续/完成/修复”时应连续推进到可验证完成 (updated: 2026-06-27)
- [preference] 需要命令时，给一整段可直接执行的命令 (updated: 2026-06-27)

## 关键决策
- [decision] OpenClaw 主模型当前使用 `myproxy/gpt-5.5` (updated: 2026-06-27)
- [decision] 复杂、工具、联网、排障、配置、安全、关键决策默认走 GPT/中转模型 (updated: 2026-06-27)
- [decision] 不擅自修改 OpenClaw / Hermes / Codex 配置 (updated: 2026-06-27)
- [decision] 不自动删除或合并重要记忆，先产出候选报告再确认 (updated: 2026-06-27)

## 当前状态
- [state] OpenClaw 工作区位于 `/root/.openclaw/workspace` (updated: 2026-06-27)
- [state] 记忆学习目录为 `hermes-openclaw-learning/`，包含项目地图、用户偏好、失败经验、成功模式、经验卡和评测模板 (updated: 2026-06-27)
- [state] 本地记忆治理优先做审计、压缩建议和归档候选，不直接改共享记忆服务 (updated: 2026-06-27)
- [service_state] ESP 服务器（关键词：`esp服务器`、`ESP服务器`、`ESP-server`、`esp-server`）：IP `124.221.162.188`；SSH 旧操作使用 `ubuntu@124.221.162.188` 与 key `/root/.ssh/openclaw_124_221_162_188`；GitHub 仓库 `git@github.com:2195573507-web/ESP-server.git`；服务器侧曾见 clone 路径 `/home/ubuntu/ESP-server` 与 `/opt/ESP-server`，二者可能不一致；`/opt/ESP-server` 旧记录曾有脏状态 `M db/database.db` 和未跟踪 `cache/`，不要未确认覆盖；旧 smoke regression 在 `scripts/smoke-regression.js` 出现 `502 != 200`；做 `main/api/ui` 分支合并、push 或部署前先检查两个 clone、GitHub remote、脏文件和分支策略。详见 OpenClaw wiki `syntheses/esp-server-memory-archive.md` 与共享记忆 `e23814ff-9835-4cb2-9653-a97a855cc3b2`。(updated: 2026-07-09)

## 待办/进行中的事
- [todo] 维持 OpenClaw 工作记忆和学习目录的可检索性，定期补充经验卡、失败库和成功模式 (updated: 2026-06-27)
- [todo] 当记忆超过上限时，先合并相似条目或移除过时条目，再考虑归档 (updated: 2026-06-27)

---

## 关联入口

- [[agents/index|Agent 总览]]
- [[MOC|知识库导航]]

## 会话压缩摘要

- [summary/shared] 压缩 `2026-06-19.md`：关键决策0条，完成工作2条，问题0条，待跟进0条；高价值摘要：Recent web-health task: checked `:9000`, `:9100`, `:9200`; all services/APIs healthy. Dashboard CSS bug `--btn-hover` undefined was fixed. Token/cost trend API issue was a test-key mistake; data was normal.；详见 [[记忆治理/openclaw-session-memory-compact-20260705T030002Z|压缩报告]]。(updated: 2026-07-05) <!-- compact:2026-06-19.md:ff28c4bc65beeb27 -->
- [summary/shared] 压缩 `2026-06-20.md`：关键决策0条，完成工作2条，问题2条，待跟进1条；高价值摘要：Next continuation should run `systemctl daemon-reload && systemctl enable --now ollama`, verify `curl http://127.0.0.1:11434/api/tags`, then `ollama pull nomic-embed-text`, test `/api/embeddings`, configure OpenClaw embedding provider, rebuild memory index, an；详见 [[记忆治理/openclaw-session-memory-compact-20260705T030002Z|压缩报告]]。(updated: 2026-07-05) <!-- compact:2026-06-20.md:e9eee5438dca8c69 -->
- [summary/shared] 压缩 `2026-06-21.md`：关键决策0条，完成工作3条，问题0条，待跟进0条；高价值摘要：User reported Hermes/OpenClaw config UI available-model display was wrong. Fixed Hermes Dashboard backend/frontend in `/opt/hermes-dashboard`.；详见 [[记忆治理/openclaw-session-memory-compact-20260705T030002Z|压缩报告]]。(updated: 2026-07-05) <!-- compact:2026-06-21.md:f87aebbe825b9393 -->

