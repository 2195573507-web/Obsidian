---
title: "OpenClaw + Hermes 记忆读取优化方案"
tags: ['plan', 'project']
updated: 2026-06-24
---

# OpenClaw + Hermes 记忆读取优化方案

目标：更快、更准、更省 token，同时让 Obsidian 统一管理长期知识。

## 当前状态

- 统一 vault：`/root/obsidian-vault/`
- 共享记忆服务：`shared-memory-client`，数据库在 `/opt/shared-agent-memory/data/memory.sqlite`
- 共享导出：`/root/obsidian-vault/shared-memory/exports/shared-memory.md`
- Hermes 固定记忆注入较短：`memory_char_limit=1200`，`user_char_limit=800`
- OpenClaw 已启用 memorySearch，并额外索引 shared-memory 与 Hermes/OpenClaw 用户文件

## 最优原则

1. **短核心常驻**：只有身份、路径、硬规则、常用服务放进常驻记忆。
2. **长知识按需检索**：项目记录、日报、调研、错误历史只进 Obsidian/wiki/shared-memory，不常驻注入。
3. **分层记忆**：preference/fact/service_state/decision/success/failure/temporary/todo/noise 分开。
4. **先搜后读**：先 `memory_search` 或 `shared-memory-client context/search`，再读取命中的小片段。
5. **日报类内容不进核心记忆**：只存 Obsidian 文件，并用索引页链接。

## 建议读取流程

### OpenClaw

- 问到历史、偏好、路径、决策：先 `memory_search(corpus=all)`。
- 命中后只读 5-20 行相关内容。
- 大型调研优先写 Obsidian 文件，不写入核心 memory。
- 每周把重复出现的事实沉淀为 1 条 shared-memory fact/decision。

### Hermes

- 常驻 MEMORY 只保留 10-20 条高价值事实。
- 执行复杂任务前调用：
  - `shared-memory-client context "任务关键词" --limit 8 --mode planning`
  - 调试问题调用：`shared-memory-client context "错误关键词" --limit 8 --mode debug --include-failure`
- 不把整份日报/报告放入 memory；只放文件路径和索引。

## 建议结构

- `/root/obsidian-vault/agents/openclaw/USER.md`：用户偏好、输出风格、常用约束
- `/root/obsidian-vault/agents/hermes/MEMORY.md`：Hermes 必须常驻的短事实
- `/root/obsidian-vault/shared-memory/exports/shared-memory.md`：跨 Agent 共享事实导出
- `/root/obsidian-vault/plans/`：方案、计划、架构
- `/root/obsidian-vault/全球咨询调研/`：每日调研与周报

## Token 控制标准

| 内容类型 | 存放位置 | 是否常驻 |
|---|---|---|
| 用户偏好 | agents/*/USER.md + shared-memory preference | 是，短 |
| 服务器路径/服务端口 | shared-memory fact/service_state | 是，短 |
| 项目方案 | Obsidian plans | 否，按需搜 |
| 日报/周报 | Obsidian 专题目录 | 否，按需搜 |
| 错误教训 | shared-memory failure/success | 否，debug 时读 |
| 临时状态 | shared-memory temporary/todo，带过期 | 否 |

## 立刻可做的优化

1. 清理 Hermes MEMORY：只保留最核心 10-20 条。
2. 为 OpenClaw 增加 Obsidian 关键索引路径，而不是索引整个 vault。
3. 每个专题建 README/索引页，Agent 先读索引再读具体文件。
4. 对 shared-memory 定期 expire temporary/noise。
5. 新任务统一写入 Obsidian 文件，聊天里只回摘要和路径。

## 推荐命令

```bash
shared-memory-client context "server paths services" --limit 8 --mode planning
shared-memory-client context "dashboard error" --limit 8 --mode debug --include-failure
shared-memory-client search "用户偏好" --limit 5 --kinds preference,fact,decision
shared-memory-client expire
shared-memory-client export
```
