---
title: Hermes Working Memory
type: working-memory
agent: hermes
updated: 2026-06-27
tags: [hermes, memory, working-memory]
---

# Hermes Working Memory

> Layer 1 工作记忆：只保留 Hermes 当前有效的偏好、关键决策、服务器状态和待办。上限 50 条；历史状态放入日记/共享记忆/Obsidian 报告，不再当当前事实。

## 用户偏好

- [preference/shared] 用户至秦偏好直接行动、少确认；说“继续/完成/修复”时希望连续推进到可验证完成；需要命令时偏好一整段可直接执行命令。(updated: 2026-06-24)
- [preference/shared] 默认中文回复；涉及 cron/定时任务时使用北京时间意图并标注 UTC 换算。(updated: 2026-06-24)
- [preference/shared] 长研究任务优先生成报告/候选方案；涉及危险写操作、外部发布、网关/系统服务/证书/nginx/cron/GitHub push 前先确认。(updated: 2026-06-27)

## 关键决策

- [decision/shared] 允许安排任意时段任务，包括 BJT 00:00-07:00；不要再套用夜间禁做规则。(updated: 2026-06-24)
- [decision/shared] 不使用本地千问/Qwen；所有主请求和后台需要模型的任务默认走中转 GPT。本地 Ollama 仅保留 `nomic-embed-text` 做 embeddings。(updated: 2026-06-24)
- [decision/shared] 写入 shared-agent-memory 时必须自带 `kind`、`importance`、`sensitivity`、`tags`、`subject`，不依赖自动分类器。(updated: 2026-06-24)
- [decision/shared] Obsidian vault 是 Hermes/OpenClaw 共享知识中枢，重要新文件需带 frontmatter、至少 1 个 wikilink，并在必要时更新 [[MOC|知识库导航]]。(updated: 2026-06-24)
- [decision/shared] Hermes Provider 与 OpenClaw Provider 独立维护；不要把 Hermes provider 配置当成 OpenClaw 当前 provider，也不要自动互相同步。(updated: 2026-06-27)

## 当前 Hermes 侧服务器状态

- [state/shared] 当前公开 Atlas Web 入口：`monitor.zmjjkkk.fun -> atlas-observe:19000`，`agent.zmjjkkk.fun -> atlas-agents:19100`，`notes.zmjjkkk.fun -> atlas-notes:19200`，`server.zmjjkkk.fun -> atlas-control:19600`，`files.zmjjkkk.fun -> atlas-files:19300`，`daily.zmjjkkk.fun -> atlas-daily:19500`，`collab.zmjjkkk.fun -> atlas-collab:19700`。(checked_at: 2026-06-27 05:12 UTC)
- [state/shared] Hermes legacy/support services still active: `hermes-gateway.service`，`hermes-dashboard-web.service` on `:9100`，`hermes-system-monitor.service` on `:9000`。这些是 legacy/support，不代表当前主 Web 入口。(checked_at: 2026-06-27 05:12 UTC)
- [state/shared] Shared memory stack: `shared-agent-memory:9400`，`atlas-memory:19400`，`atlas-recall:19410` active/healthy；Hermes 长期共享事实应写入 shared-memory 或 Obsidian，Hermes `MEMORY.md` 只保留当前工作记忆。(checked_at: 2026-06-27 05:38 UTC)
- [state/shared] 关键路径：Hermes `/opt/hermes-agent/`，legacy Dashboard `/opt/hermes-dashboard/`，legacy System Monitor `/opt/hermes-system-monitor/`，Shared Memory `/opt/shared-agent-memory/`，Vault `/root/obsidian-vault/`，Atlas apps `/opt/atlas-*`。(updated: 2026-06-27)
- [state/shared] Provider/中转状态是 mutable fact。旧的 401/alert/provider health 记忆只当历史快照；排障前必须 live check Atlas Provider Watch / 当前 `/v1/models`，且默认 report-only，不自动切换路由。(updated: 2026-06-27)
- [state/shared] GitHub vault remote：`git@github.com:2195573507-web/Obsidian.git`，`obsidian-vault-autocommit.timer` 每 6 小时自动提交并推送；推送/同步前仍需确认当前需求和状态。(updated: 2026-06-27)
- [state/private] 服务器 `192.204.35.79`：OpenResty/Nginx + MySQL:3306 + SSH:22，用户无 SSH，仅远程操作；操作前重新确认授权和连通性。(updated: 2026-06-24)

## 待办/进行中的事

- [todo/shared] 继续保持 Obsidian vault 的 MOC、frontmatter、wikilinks、templates、Dataview 查询和 Knowledge Web 页面可用；重要结构变化后生成候选报告，需要推送时先确认。(updated: 2026-06-27)
- [todo/shared] 共享记忆维护策略：优先生成候选报告，除用户明确要求外不自动删除/合并重要记忆；temporary/todo 完成后标记完成/过期，不直接删除。(updated: 2026-06-27)
- [todo/hermes] Hermes 旧状态已从工作记忆中历史化；后续如需要，可单独审计 `/root/obsidian-vault/agents/hermes/USER.md` 与 legacy dashboard 文档，但不要动 OpenClaw 侧记忆。(updated: 2026-06-27)

## 相关链接

- [[MOC|知识库导航]]
- [[服务器架构]]
- [[agents/shared/TOOLS|共用工具与记忆规则]]
- [[plans/hermes-openclaw-memory-strategy|Hermes/OpenClaw 记忆策略]]
- [[shared-memory/exports/shared-memory|共享记忆导出]]
