---
title: "2026-07-11-Hermes每周skills记忆趋势"
type: note
category: "shared-memory"
updated: 2026-07-11
managed_by: Hermes
tags: [vault, shared-memory]
---

# Hermes 每周 skills/记忆增长趋势报告（2026-07-11）

> 运行日期：2026-07-11（北京时间，来自 `TZ=Asia/Shanghai date +%F`）  
> 任务边界：本次仅统计和报告，未清理、未移动、未删除任何文件；未恢复旧 BJT 03:00-05:00 自动学习任务；未修改 Hermes/OpenClaw 主模型配置。

## 1. 数据来源

- 北京时间日期：`TZ=Asia/Shanghai date +%F` -> `2026-07-11`
- shared-memory 健康检查：`shared-memory-client health`
- SQLite 只读查询：`/opt/shared-agent-memory/data/memory.sqlite`
- skills 目录统计：只读递归统计
- 历史基线：`/root/obsidian-vault/shared-memory/reports/2026-07-05-Hermes每周skills记忆趋势.md`

## 2. Skills 数量与目录规模

| 路径 | 顶层目录数 | `SKILL.md` 数 | 文件数 | 递归目录数（含根） | 字节数 | 状态 |
|---|---:|---:|---:|---:|---:|---|
| `/root/.hermes/skills` | 20 | 56 | 335 | 126 | 1,540,073 | 存在 |
| `/opt/hermes-agent/skills` | 17 | 73 | 448 | 157 | 5,676,014 | 存在 |
| **合计** | **37** | **129** | **783** | **283** | **7,216,087** | - |

### `/root/.hermes/skills` 顶层目录清单

`.curator_backups`、`autonomous-ai-agents`、`creative`、`data-science`、`devops`、`dogfood`、`github`、`hermes-dashboard`、`intelligence-cron-pipeline`、`media`、`note-taking`、`research`、`shared-agent-memory`、`software-development`、`yuanbao`、`autonomous-ai-agents`、`creative`、`devops`、`github`、`software-development`

### `/opt/hermes-agent/skills` 顶层目录清单

`apple`、`autonomous-ai-agents`、`creative`、`data-science`、`devops`、`dogfood`、`email`、`github`、`media`、`mlops`、`note-taking`、`productivity`、`research`、`smart-home`、`social-media`、`software-development`、`yuanbao`

## 3. Shared-memory 当前规模

`shared-memory-client health` 返回：`ok=true`，版本 `1.3.0`，数据库 `/opt/shared-agent-memory/data/memory.sqlite`，vault `/root/obsidian-vault/shared-memory`，embedding model `nomic-embed-text`。

| 指标 | 数量 |
|---|---:|
| memories | 100 |
| memory_embeddings | 97 |
| local_tasks | 39 |
| events | 0 |

## 4. 记忆分布

### 按 kind 分布

| kind | 数量 |
|---|---:|
| success | 45 |
| service_state | 15 |
| fact | 12 |
| decision | 11 |
| temporary | 8 |
| failure | 3 |
| preference | 3 |
| noise | 2 |
| todo | 1 |

### 按状态字段说明

`memories` 表当前字段为：`id, created_at, updated_at, scope, kind, source_agent, subject, content, confidence, importance, tags_json, metadata_json, disabled, expires_at`。  
因此本报告以 `disabled` 作为记忆状态口径。

| disabled | 数量 | 占比 |
|---:|---:|---:|
| 0 | 74 | 74.0% |
| 1 | 26 | 26.0% |

### 按 kind/disabled 分布

| kind | disabled | 数量 |
|---|---:|---:|
| decision | 0 | 5 |
| decision | 1 | 6 |
| fact | 0 | 11 |
| fact | 1 | 1 |
| failure | 0 | 2 |
| failure | 1 | 1 |
| noise | 1 | 2 |
| preference | 0 | 1 |
| preference | 1 | 2 |
| service_state | 0 | 14 |
| service_state | 1 | 1 |
| success | 0 | 36 |
| success | 1 | 9 |
| temporary | 1 | 8 |
| todo | 0 | 1 |

## 5. local_tasks 分布

### 按 status 分布

| status | 数量 | 占比 |
|---|---:|---:|
| done | 14 | 35.9% |
| escalate | 14 | 35.9% |
| fallback | 10 | 25.6% |
| failed | 1 | 2.6% |

非 `done` 合计 25，占 64.1%。

### 按 task_type 分布

| task_type | 数量 |
|---|---:|
| summary | 13 |
| compaction-summary | 7 |
| report-draft | 7 |
| completion-postprocess | 5 |
| short-ack | 3 |
| cron-summary | 1 |
| log-summary | 1 |
| polish | 1 |
| status-summary | 1 |

### 按 task_type/status 分布

| task_type | status | 数量 |
|---|---|---:|
| compaction-summary | escalate | 7 |
| completion-postprocess | done | 2 |
| completion-postprocess | escalate | 3 |
| cron-summary | done | 1 |
| log-summary | done | 1 |
| polish | done | 1 |
| report-draft | done | 5 |
| report-draft | escalate | 1 |
| report-draft | failed | 1 |
| short-ack | done | 3 |
| status-summary | fallback | 1 |
| summary | done | 1 |
| summary | escalate | 3 |
| summary | fallback | 9 |

## 6. 与上周对比趋势

历史基线文件：`/root/obsidian-vault/shared-memory/reports/2026-07-05-Hermes每周skills记忆趋势.md`。

| 指标 | 本周 | 上周 | 增量 | 趋势 |
|---|---:|---:|---:|---|
| `/root/.hermes/skills` 顶层目录数 | 20 | 15 | +5 | 增长 |
| `/opt/hermes-agent/skills` 顶层目录数 | 17 | 17 | 0 | 持平 |
| skills 顶层目录合计 | 37 | 32 | +5 | 增长 |
| `/root/.hermes/skills` 字节数 | 1,540,073 | 2,948,189 | -1,408,116 | 下降 47.8% |
| `/opt/hermes-agent/skills` 字节数 | 5,676,014 | 6,319,086 | -643,072 | 下降 10.2% |
| skills 字节数合计 | 7,216,087 | 9,267,275 | -2,051,188 | 下降 22.1% |
| memories | 100 | 82 | +18 | 增长 22.0% |
| memory_embeddings | 97 | 79 | +18 | 增长 22.8% |
| memories - embeddings 差值 | 3 | 3 | 0 | 持平 |
| local_tasks | 39 | 39 | 0 | 持平 |
| local_tasks done | 14 | 14 | 0 | 持平 |
| local_tasks escalate | 14 | 14 | 0 | 持平 |
| local_tasks fallback | 10 | 10 | 0 | 持平 |
| local_tasks failed | 1 | 1 | 0 | 持平 |
| disabled 记忆 | 26 | 22 | +4 | 小幅增长 |

### 记忆 kind 趋势

| kind | 本周 | 上周 | 增量 | 趋势 |
|---|---:|---:|---:|---|
| success | 45 | 31 | +14 | 增长 |
| service_state | 15 | 12 | +3 | 增长 |
| fact | 12 | 11 | +1 | 增长 |
| decision | 11 | 11 | 0 | 持平 |
| temporary | 8 | 8 | 0 | 持平 |
| failure | 3 | 3 | 0 | 持平 |
| preference | 3 | 3 | 0 | 持平 |
| noise | 2 | 2 | 0 | 持平 |
| todo | 1 | 1 | 0 | 持平 |

## 7. 异常增长风险

- **skills 目录数量风险：低**。顶层目录合计从 32 增至 37，增长 5 个；`/opt/hermes-agent/skills` 持平，新增主要集中在 `/root/.hermes/skills`，属于结构性扩展，不是失控膨胀。
- **skills 体积风险：低**。总字节数从 9,267,275 降至 7,216,087，下降 22.1%，没有异常膨胀迹象。
- **记忆增长风险：低到中**。memories 从 82 增至 100，增长 18 条；`memory_embeddings` 同步从 79 增至 97，差值稳定为 3，没有 embedding 积压扩大。
- **local_tasks 风险：中**。总数持平，但非 `done` 仍为 25/39（64.1%），其中 `summary/fallback=9`、`compaction-summary/escalate=7` 仍是主要堆积点，属于持续存在的路由与收敛风险。
- **disabled 记忆风险：中**。disabled 记忆从 22 增至 26，占比 26.0%，单周增加 4 条，幅度不大，但值得继续跟踪。

## 8. 建议

1. 继续保持只统计不清理；Hermes 不处理 skills 整理，skills 整理继续交由 OpenClaw 负责。
2. 下周继续观察 `/root/.hermes/skills` 顶层目录是否继续增加；本周增长集中在 Hermes 本地技能树，需确认是否为预期扩展。
3. 继续观察 memories 与 `memory_embeddings` 的差值；当前稳定为 3，暂不需要干预。
4. 对 `local_tasks` 的 `summary` 与 `compaction-summary` 路径做只读诊断，重点看 `fallback/escalate` 的成因，但不要在本任务中做修复或清理。
5. 不恢复旧 BJT 03:00-05:00 自动学习任务，不修改 Hermes/OpenClaw 主模型配置。

---

## 关联入口

- [[shared-memory/README|共享记忆说明]]
- [[MOC|知识库导航]]
