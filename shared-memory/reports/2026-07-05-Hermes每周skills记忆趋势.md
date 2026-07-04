# Hermes 每周 skills/记忆增长趋势报告（2026-07-05）

> 运行日期：2026-07-05（北京时间，来自 `TZ=Asia/Shanghai date +%F`）  
> 任务边界：本次仅统计和报告，未清理、未移动、未删除任何文件；未恢复旧 BJT 03:00-05:00 自动学习任务；未修改 Hermes/OpenClaw 主模型配置。

## 1. 数据来源

- 北京时间日期：`TZ=Asia/Shanghai date +%F` → `2026-07-05`
- shared-memory 健康检查：`shared-memory-client health`
- SQLite 只读查询：`/opt/shared-agent-memory/data/memory.sqlite`，使用 `file:...?mode=ro`
- skills 目录统计：`find`、`du -sh`、`du -sb` 只读统计
- 历史基线：`/root/obsidian-vault/shared-memory/reports/2026-06-27-Hermes每周skills记忆趋势.md`

## 2. Skills 数量与目录规模

| 路径 | 顶层目录数 | `SKILL.md` 数 | 文件数 | 递归目录数（含根） | `du -sh` | `du -sb` 字节数 | 状态 |
|---|---:|---:|---:|---:|---:|---:|---|
| `/root/.hermes/skills` | 15 | 56 | 314 | 118 | 3.5M | 2,948,189 | 存在 |
| `/opt/hermes-agent/skills` | 17 | 73 | 448 | 157 | 7.0M | 6,319,086 | 存在 |
| **合计** | **32** | **129** | **762** | **275** | - | **9,267,275** | - |

### `/root/.hermes/skills` 顶层目录清单

`.curator_backups`、`autonomous-ai-agents`、`creative`、`data-science`、`devops`、`dogfood`、`github`、`hermes-dashboard`、`intelligence-cron-pipeline`、`media`、`note-taking`、`research`、`shared-agent-memory`、`software-development`、`yuanbao`

### `/opt/hermes-agent/skills` 顶层目录清单

`apple`、`autonomous-ai-agents`、`creative`、`data-science`、`devops`、`dogfood`、`email`、`github`、`media`、`mlops`、`note-taking`、`productivity`、`research`、`smart-home`、`social-media`、`software-development`、`yuanbao`

## 3. Shared-memory 当前规模

`shared-memory-client health` 返回：`ok=true`，版本 `1.3.0`，数据库 `/opt/shared-agent-memory/data/memory.sqlite`，vault `/root/obsidian-vault/shared-memory`，embedding model `nomic-embed-text`。

| 指标 | 数量 |
|---|---:|
| memories | 82 |
| memory_embeddings / embeddings | 79 |
| local_tasks | 39 |
| events | 0 |
| SQLite 文件大小 | 1,880,064 bytes |

## 4. 记忆分布

### 按 kind 分布

| kind | 数量 |
|---|---:|
| success | 31 |
| service_state | 12 |
| decision | 11 |
| fact | 11 |
| temporary | 8 |
| failure | 3 |
| preference | 3 |
| noise | 2 |
| todo | 1 |

### 按 status/disabled 分布

当前 `memories` 表字段为：`id, created_at, updated_at, scope, kind, source_agent, subject, content, confidence, importance, tags_json, metadata_json, disabled, expires_at`，无 `status` 字段；因此本报告以 `disabled` 作为记忆状态分布。

| disabled | 数量 | 占比 |
|---:|---:|---:|
| 0 | 60 | 73.2% |
| 1 | 22 | 26.8% |

### 按 kind/disabled 分布

| kind | disabled | 数量 |
|---|---:|---:|
| decision | 0 | 5 |
| decision | 1 | 6 |
| fact | 0 | 10 |
| fact | 1 | 1 |
| failure | 0 | 2 |
| failure | 1 | 1 |
| noise | 1 | 2 |
| preference | 0 | 1 |
| preference | 1 | 2 |
| service_state | 0 | 11 |
| service_state | 1 | 1 |
| success | 0 | 30 |
| success | 1 | 1 |
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

历史基线文件：`/root/obsidian-vault/shared-memory/reports/2026-06-27-Hermes每周skills记忆趋势.md`。

> 注：上周 skills 数口径为“顶层 skills 目录数”；本周新增补充 `SKILL.md` 文件数。与上周对比时，仅对相同口径（顶层目录数、总字节数、memory/local_tasks 等）计算趋势。

| 指标 | 本周 | 上周 | 增量 | 趋势 |
|---|---:|---:|---:|---|
| `/root/.hermes/skills` 顶层目录数 | 15 | 21 | -6 | 下降 |
| `/opt/hermes-agent/skills` 顶层目录数 | 17 | 17 | 0 | 持平 |
| skills 顶层目录合计 | 32 | 38 | -6 | 下降 |
| `/root/.hermes/skills` 字节数 | 2,948,189 | 2,180,457 | +767,732 | 增长 35.2% |
| `/opt/hermes-agent/skills` 字节数 | 6,319,086 | 6,319,086 | 0 | 持平 |
| skills 字节数合计 | 9,267,275 | 8,499,543 | +767,732 | 增长 9.0% |
| memories | 82 | 77 | +5 | 增长 6.5% |
| embeddings | 79 | 74 | +5 | 增长 6.8% |
| memories - embeddings 差值 | 3 | 3 | 0 | 持平 |
| local_tasks | 39 | 39 | 0 | 持平 |
| local_tasks done | 14 | 14 | 0 | 持平 |
| local_tasks escalate | 14 | 14 | 0 | 持平 |
| local_tasks fallback | 10 | 10 | 0 | 持平 |
| local_tasks failed | 1 | 1 | 0 | 持平 |
| disabled 记忆 | 22 | 21 | +1 | 小幅增长 |

### 记忆 kind 趋势

| kind | 本周 | 上周 | 增量 | 趋势 |
|---|---:|---:|---:|---|
| success | 31 | 26 | +5 | 增长 |
| service_state | 12 | 12 | 0 | 持平 |
| decision | 11 | 11 | 0 | 持平 |
| fact | 11 | 11 | 0 | 持平 |
| temporary | 8 | 8 | 0 | 持平 |
| failure | 3 | 3 | 0 | 持平 |
| preference | 3 | 3 | 0 | 持平 |
| noise | 2 | 2 | 0 | 持平 |
| todo | 1 | 1 | 0 | 持平 |

## 7. 异常增长风险

- **skills 目录数量风险：低**。按上周可比口径，顶层目录合计从 38 降至 32；`/opt/hermes-agent/skills` 持平，未见目录数量异常膨胀。
- **skills 体积风险：中低**。总字节数从 8,499,543 增至 9,267,275，增长 767,732 bytes（约 9.0%），增长全部来自 `/root/.hermes/skills`；绝对规模仍较小，但 `/root/.hermes/skills` 单周体积增长 35.2%，建议继续观察是否持续。
- **记忆增长风险：低**。memories 从 77 增至 82，新增 5 条，均体现为 `success` kind 增长；embeddings 同步从 74 增至 79，差值维持 3，未见 embedding 积压扩大。
- **local_tasks 风险：中**。总数和状态分布与上周完全持平，但非 `done` 仍为 25/39（64.1%），主要集中在 `compaction-summary/escalate=7` 与 `summary/fallback=9`。这不是新增风险，但仍是持续存在的质量/路由风险。
- **disabled 记忆风险：低到中**。disabled 记忆从 21 增至 22，占比 26.8%；单周仅 +1，不构成异常增长，但占比仍需持续跟踪。

## 8. 建议

1. 继续保持只统计不清理；Hermes 不处理 skills 整理，skills 整理继续交由 OpenClaw 负责。
2. 下周重点观察 `/root/.hermes/skills` 体积是否继续高比例增长；若连续增长，再定位新增文件类型和来源。
3. 继续观察 memories 与 embeddings 差值；目前差值稳定为 3，暂不需要干预。
4. local_tasks 的 `escalate/fallback` 虽未新增，但占比长期偏高，建议后续单独做只读诊断报告，重点看 `summary` 与 `compaction-summary` 的路由/模型可用性。
5. 不恢复旧 BJT 03:00-05:00 自动学习任务，不修改 Hermes/OpenClaw 主模型配置。
