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

## 待办/进行中的事
- [todo] 维持 OpenClaw 工作记忆和学习目录的可检索性，定期补充经验卡、失败库和成功模式 (updated: 2026-06-27)
- [todo] 当记忆超过上限时，先合并相似条目或移除过时条目，再考虑归档 (updated: 2026-06-27)
