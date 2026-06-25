---
title: Hermes Working Memory
type: working-memory
agent: hermes
updated: 2026-06-25
tags: [hermes, memory, working-memory]
---

# Hermes Working Memory

> Layer 1 工作记忆：只保留当前有效的偏好、关键决策、服务器状态和待办。上限 50 条，过时信息应合并或移入 [[shared-memory/exports/shared-memory|共享记忆归档]]。

## 用户偏好

- [preference/shared] 用户至秦偏好直接行动、少确认；说“继续/完成/修复”时希望连续推进到可验证完成；需要命令时偏好一整段可直接执行命令。(updated: 2026-06-24)
- [preference/shared] 默认中文回复；涉及 cron/定时任务时使用北京时间意图并标注 UTC 换算。(updated: 2026-06-24)
- [preference/shared] 长研究任务优先用 cron 多轨道方式，学习任务只生成报告，不主动改代码或配置。(updated: 2026-06-24)

## 关键决策

- [decision/shared] 允许安排任意时段任务，包括 BJT 00:00-07:00；不要再套用夜间禁做规则。(updated: 2026-06-24)
- [decision/shared] 不使用本地千问/Qwen；所有主请求和后台需要模型的任务默认走中转 GPT。本地 Ollama 仅保留 `nomic-embed-text` 做 embeddings。(updated: 2026-06-24)
- [decision/shared] 写入 shared-agent-memory 时必须自带 `kind`、`importance`、`sensitivity`、`tags`、`subject`，不依赖自动分类器。(updated: 2026-06-24)
- [decision/shared] Obsidian vault 是 Hermes/OpenClaw 共享知识中枢，重要新文件需带 frontmatter、至少 1 个 wikilink，并在必要时更新 [[MOC|知识库导航]]。(updated: 2026-06-24)

## 当前服务器状态

- [state/shared] 主服务器 `38.55.146.137`：system-monitor `:9000`，Hermes Dashboard `:9100`，Obsidian Knowledge Web `:9200`，shared-memory `:9400`，OpenClaw `127.0.0.1:18789`。(updated: 2026-06-24)
- [state/shared] 关键路径：Hermes `/opt/hermes-agent/`，Dashboard `/opt/hermes-dashboard/`，System Monitor `/opt/hermes-system-monitor/`，Shared Memory `/opt/shared-agent-memory/`，Vault `/root/obsidian-vault/`。(updated: 2026-06-24)
- [state/shared] GitHub vault remote：`git@github.com:2195573507-web/Obsidian.git`，`obsidian-vault-autocommit.timer` 每 6 小时自动提交并推送。(updated: 2026-06-24)
- [state/shared] 用户常用中转站：Hermes/OpenClaw 主要走 `sub.zmjjkkk.fun/v1`；备用 `pool.gptstore.club/v1` key 不匹配。排障优先测当前中转站 `/models` 和逐模型探活。(updated: 2026-06-24)
- [state/private] 服务器 `192.204.35.79`：OpenResty/Nginx + MySQL:3306 + SSH:22，用户无 SSH，仅远程操作。(updated: 2026-06-24)
- [state/shared] GitHub `git@github.com:2195573507-web/usserver.git` 存放服务器 12 个自建项目 monorepo；第三方项目不上传：hermes-agent、openclaw、sub2api、upstream-hub-standalone。(updated: 2026-06-24)

## 待办/进行中的事

- [todo/shared] 继续保持 Obsidian vault 的 MOC、frontmatter、wikilinks、templates、Dataview 查询和 Knowledge Web 页面可用；重要结构变化后推送 GitHub。(updated: 2026-06-24)
- [todo/shared] 共享记忆维护策略：优先生成候选报告，除用户明确要求外不自动删除/合并重要记忆。(updated: 2026-06-24)

## 相关链接

- [[MOC|知识库导航]]
- [[服务器架构]]
- [[agents/shared/TOOLS|共用工具与记忆规则]]
- [[shared-memory/exports/shared-memory|共享记忆导出]]
