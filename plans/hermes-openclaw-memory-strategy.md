---
title: Hermes/OpenClaw 记忆策略
updated: 2026-06-27
tags: [memory, hermes, openclaw, atlas]
---

# Hermes/OpenClaw 记忆策略

关联：[[agents/shared/TOOLS|共享工具规则]]、[[agents/shared/USER|共享用户画像]]、[[MOC|知识库导航]]

## 目标

让 Hermes 与 OpenClaw 使用一致的长期记忆策略：少写噪音、写入带结构、召回可验证、旧状态不误导当前判断。

## 分层

1. **会话短期记忆**：当前对话上下文，只服务本轮任务，不主动沉淀。
2. **日记/episodic 记忆**：`memory/YYYY-MM-DD.md`，记录“发生过什么”和可追溯的实施日志。
3. **共享 semantic 记忆**：`shared-agent-memory` / Atlas Memory，存放可跨 Hermes/OpenClaw 复用的偏好、决策、事实、服务状态、待办。
4. **程序/procedural 记忆**：Skills、TOOLS、经验卡，记录可重复流程，不塞进普通事实记忆。
5. **Compiled wiki / Obsidian**：稳定知识、架构摘要、报告和长期文档。

## 写入策略

- 只写会影响未来行为的信息：用户偏好、长期决策、当前架构摘要、重要服务状态、已验证故障/修复、明确待办。
- 不写：寒暄、短期中间状态、未验证猜测、重复日志、一次性临时输出。
- 所有 shared-memory 写入必须显式字段：`kind`、`importance`、`sensitivity`、`tags`、`subject`、`content`。
- “当前状态”必须写入 `checked_at` 或在内容中明确 live-check 时间；旧状态默认按历史看。
- Provider、端口、服务健康等 mutable facts 只作为快照，不得跳过 live check。

## 召回策略

- 回答 prior work / decisions / preferences / todos 前必须先查 memory/wiki。
- 执行服务、端口、配置、Provider、健康状态相关任务前，先读“当前事实 marker”，再 live check。
- 自动召回优先使用 Atlas Recall 模板：
  - `memory-policy`：记忆策略和偏好
  - `server-architecture`：当前服务/端口架构
  - `provider-health`：中转健康
  - `openclaw-no-reply`：OpenClaw 不回消息类问题
- 召回结果要带来源路径/行号或接口 source id，避免“凭记忆说”。

## 压缩与治理

- 日记超过长篇时，追加“当前事实摘要/历史标记”，不要把历史条目改成当前事实。
- temporary/todo 记忆到期或完成后，标记完成/过期，不直接删除。
- 每月 decay：低重要度、长期未访问进入 noise 候选。
- 每季度 audit：重复、矛盾、过期状态、缺证据项形成报告。
- 重要架构变更后，必须更新一条当前事实摘要。

## Hermes / OpenClaw 分工

- Hermes：可以继续使用 `/root/obsidian-vault/agents/hermes/MEMORY.md` 做自身工作记忆，但长期共享事实应沉淀到 shared-memory 或 Obsidian。
- OpenClaw：使用 workspace `memory/*.md` 作为日记/episodic 记录，使用 wiki/compiled memory 做长期知识。
- 两者共享：偏好、决策、当前架构、安全边界、Provider 策略、可复用经验。
- 不共享：敏感 token、一次性会话细节、未验证猜测。

## 当前未完成/后续建议

- 把 Hermes 的旧 MEMORY.md 再做一次去重/历史化审计。
- 给 Atlas Recall 前端增加 memory-policy 模板快捷入口。
- 定期核验 embeddings 数量与 nomic-embed-text 可用性。
