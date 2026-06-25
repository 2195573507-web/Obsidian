---
title: OpenClaw Working Memory
type: working-memory
agent: openclaw
source: /root/.openclaw/workspace/memory/*.md + /root/obsidian-vault/agents/hermes/MEMORY.md + /root/obsidian-vault/shared-memory/exports/shared-memory.md
updated: 2026-06-25
tags: [openclaw, memory, working-memory]
---

# OpenClaw Working Memory

> Layer 1 工作记忆：只保留当前有效的偏好、关键决策、服务器状态和待办。上限 50 条，过时信息应合并或移入 [[shared-memory/exports/shared-memory|共享记忆导出]]。
> 来源：OpenClaw 日记、Hermes 工作记忆与共享记忆导出中提取的当前有效信息；历史/已废弃配置不再默认保留。

## 用户偏好

- [preference/shared] 用户至秦偏好直接行动、少确认；说“继续/完成/修复”时希望连续推进到可验证完成；需要命令时偏好一整段可直接执行命令。(updated: 2026-06-25)
- [preference/shared] 默认中文回复；涉及 cron/定时任务时使用北京时间意图并标注 UTC 换算。(updated: 2026-06-25)
- [preference/shared] 所有主请求和后台需要模型的任务默认走中转 GPT；本地模型仅用于明确授权的低风险辅助任务，不作为 OpenClaw 主回复路由。(updated: 2026-06-25)
- [preference/shared] 做管理后台二开/服务器改造时，倾向端到端完成并验证后再交付；只读学习任务则偏好输出报告，不主动改服务器代码。(updated: 2026-06-25)

## 关键决策

- [decision/shared] 不再使用千问/Qwen 作为当前运行态配置；相关 qwen/ollama 旧配置仅允许作为历史/已废弃记录保留。(updated: 2026-06-25)
- [decision/shared] 用户明确要求移除本地千问后的主路由保持中转 GPT，Ollama 仅保留 `nomic-embed-text` 用于 embeddings/索引。(updated: 2026-06-25)
- [decision/shared] 写入 shared-agent-memory 时必须自带 `kind`、`importance`、`sensitivity`、`tags`、`subject`，不依赖自动分类器。(updated: 2026-06-25)
- [decision/shared] OpenClaw / Hermes 的长期共享知识中心以 Obsidian vault 为中枢，重要文件需要 frontmatter、wikilink 和可追溯来源说明。(updated: 2026-06-25)
- [decision/shared] OpenClaw 侧记忆治理采用保守策略：压缩/归档前先备份，避免删除原始数据；只做可验证的整理与归档。(updated: 2026-06-25)

## 当前服务器状态

- [state/shared] 主服务器 `38.55.146.137`：system-monitor `:9000`，Hermes Dashboard `:9100`，Obsidian Knowledge Web `:9200`，shared-memory `:9400`，OpenClaw `127.0.0.1:18789`。(updated: 2026-06-25)
- [state/shared] OpenClaw workspace：`/root/.openclaw/workspace/`；OpenClaw vault 入口：`/root/obsidian-vault/agents/openclaw/`。(updated: 2026-06-25)
- [state/shared] OpenClaw 工作记忆 canonical 文件将位于 `[[agents/openclaw/MEMORY|OpenClaw Working Memory]]`，workspace 侧通过 symlink 读取。(updated: 2026-06-25)
- [state/shared] 现有 OpenClaw 维护 timer：`openclaw-memory-maintenance.timer`（BJT 偶数小时）仍保留；本次新增的是周日 BJT 03:00 的 session memory compact timer。(updated: 2026-06-25)
- [state/shared] OpenClaw 日记当前已有 2026-06-19、06-20、06-21、06-23、06-24 五个文件，压缩脚本需只读扫描 14 天前条目并归档，不直接删除原始内容。(updated: 2026-06-25)

## 待办/进行中的事

- [todo/shared] 运行 OpenClaw session memory 周压缩：扫描 `memory/YYYY-MM-DD.md` 中 14 天前的日记，生成摘要，合并高价值内容进 `MEMORY.md`，并把原文件移到 `memory/archive/`。(updated: 2026-06-25)
- [todo/shared] 维持 OpenClaw vault 的 `SOUL.md` / `USER.md` / `IDENTITY.md` / `TOOLS.md` symlink 到 shared 目录，`MEMORY.md` 作为 OpenClaw 独有工作记忆入口。(updated: 2026-06-25)
- [todo/shared] 把 OpenClaw memory 压缩 timer 接入 systemd 并验证 `list-timers`、dry-run 和备份安全性。(updated: 2026-06-25)

## 相关链接

- [[agents/openclaw/index|OpenClaw 详情]]
- [[agents/shared/TOOLS|共用工具与记忆规则]]
- [[shared-memory/exports/shared-memory|共享记忆导出]]
- [[MOC|知识库导航]]
