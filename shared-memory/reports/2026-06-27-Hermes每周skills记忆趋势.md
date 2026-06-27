# Hermes 每周 skills/记忆增长趋势报告（2026-06-27）

> 运行日期：2026-06-27（北京时间，`TZ=Asia/Shanghai date +%F`）  
> 任务边界：本次仅统计和报告，未清理、未移动、未删除任何文件；未恢复旧 BJT 03:00-05:00 自动学习任务；未修改 Hermes/OpenClaw 主模型配置。

## 1. Skills 数量与目录规模

| 路径 | skills 目录数 | `du -sh` 规模 | `du -sb` 字节数 | 状态 |
|---|---:|---:|---:|---|
| `/root/.hermes/skills` | 21 | 2.7M | 2,180,457 | 存在 |
| `/opt/hermes-agent/skills` | 17 | 7.0M | 6,319,086 | 存在 |
| **合计** | **38** | - | **8,499,543** | - |

### `/root/.hermes/skills` 清单

`.curator_backups`、`apple`、`autonomous-ai-agents`、`creative`、`data-science`、`devops`、`dogfood`、`email`、`github`、`hermes-dashboard`、`intelligence-cron-pipeline`、`media`、`mlops`、`note-taking`、`productivity`、`research`、`shared-agent-memory`、`smart-home`、`social-media`、`software-development`、`yuanbao`

### `/opt/hermes-agent/skills` 清单

`apple`、`autonomous-ai-agents`、`creative`、`data-science`、`devops`、`dogfood`、`email`、`github`、`media`、`mlops`、`note-taking`、`productivity`、`research`、`smart-home`、`social-media`、`software-development`、`yuanbao`

## 2. Shared-memory 当前规模

来自 `shared-memory-client health` 与只读 SQLite 查询 `/opt/shared-agent-memory/data/memory.sqlite`。

| 指标 | 数量 |
|---|---:|
| memories | 77 |
| embeddings / memory_embeddings | 74 |
| local_tasks | 39 |
| events | 0 |

Health 返回：`ok=true`，版本 `1.3.0`，数据库 `/opt/shared-agent-memory/data/memory.sqlite`，vault `/root/obsidian-vault/shared-memory`，embedding model `nomic-embed-text`。

## 3. 记忆分布

### 按 kind 分布

| kind | 数量 |
|---|---:|
| success | 26 |
| service_state | 12 |
| decision | 11 |
| fact | 11 |
| temporary | 8 |
| failure | 3 |
| preference | 3 |
| noise | 2 |
| todo | 1 |

### 按 disabled 分布

| disabled | 数量 |
|---|---:|
| 0 | 56 |
| 1 | 21 |

> 注：当前 `memories` 表无 `status` 字段；本报告以 `disabled` 作为记忆启用/禁用状态分布。

## 4. local_tasks 分布

### 按 status 分布

| status | 数量 |
|---|---:|
| done | 14 |
| escalate | 14 |
| fallback | 10 |
| failed | 1 |

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

## 5. 与上周对比趋势

历史报告搜索路径：`/root/obsidian-vault/shared-memory/reports/*Hermes每周skills记忆趋势.md` 与 `/root/obsidian-vault/*Hermes每周skills记忆趋势.md`。

结果：未发现历史周报文件，因此本周为首次基线，标注为 **暂无历史基线**。

| 指标 | 本周 | 上周 | 趋势 |
|---|---:|---:|---|
| `/root/.hermes/skills` skills 数 | 21 | 无 | 暂无历史基线 |
| `/opt/hermes-agent/skills` skills 数 | 17 | 无 | 暂无历史基线 |
| skills 合计目录数 | 38 | 无 | 暂无历史基线 |
| memories | 77 | 无 | 暂无历史基线 |
| embeddings | 74 | 无 | 暂无历史基线 |
| local_tasks | 39 | 无 | 暂无历史基线 |
| local_tasks done | 14 | 无 | 暂无历史基线 |
| local_tasks escalate | 14 | 无 | 暂无历史基线 |
| local_tasks fallback | 10 | 无 | 暂无历史基线 |
| local_tasks failed | 1 | 无 | 暂无历史基线 |

## 6. 异常增长风险

- **历史趋势风险：无法判断**。没有上周基线，无法计算增量或环比。
- **skills 目录风险：低到中**。当前两个 skills 根目录合计 38 个目录，总字节数约 8.5MB，绝对规模不大；但 `/root/.hermes/skills` 比 `/opt/hermes-agent/skills` 多出 `.curator_backups`、`hermes-dashboard`、`intelligence-cron-pipeline`、`shared-agent-memory` 等目录，需要后续周报观察是否持续增加。
- **记忆 embedding 覆盖风险：低**。memories 77，embeddings 74，差值 3；当前未表现为明显异常，但建议后续持续观察差值是否扩大。
- **local_tasks 未正常完成占比偏高**。local_tasks 共 39，其中 `done=14`，`escalate=14`，`fallback=10`，`failed=1`；非 done 合计 25，占比约 64.1%。其中 `compaction-summary/escalate=7`、`summary/fallback=9` 是主要来源，建议关注本地 worker/模型路由是否经常无法完成摘要类任务。
- **disabled 记忆占比需观察**。memories 共 77，其中 disabled=21，占比约 27.3%。单周无法判断是否异常增长，但后续应纳入趋势跟踪。

## 7. 建议

1. 下周继续生成同名格式周报，以本报告作为历史基线，开始计算 skills、memories、embeddings、local_tasks 的周增量。
2. 不建议由 Hermes 自动清理 skills；skills 整理继续交由 OpenClaw 负责。
3. 重点观察 `local_tasks` 中 `escalate/fallback` 是否持续增加，尤其是 `compaction-summary` 与 `summary` 类任务。
4. 观察 memories 与 embeddings 差值是否扩大；若差值持续上升，再检查 embedding worker 或写入流程。
5. 观察 `/root/.hermes/skills` 中新增目录是否为主动安装或任务产生，避免误判为异常增长。
