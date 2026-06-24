---
type: operating-logic
category: meta
status: active
created_at: 2026-06-24T01:56:00Z
updated_at: 2026-06-24T01:56:00Z
tags:
  - obsidian
  - openclaw
  - hermes
  - knowledge-governance
---

# Obsidian 运行逻辑：OpenClaw + Hermes 统一知识库

> 目标：让 OpenClaw、Hermes、每日学习报告、全球咨询调研、记忆治理和经验卡都进入一个可读、可查、可长期沉淀的 Obsidian 结构。  
> 原则：先新增索引和规则，不移动/删除历史文件；旧目录逐步纳入索引。

## 1. 总入口

- [[README|Vault 根入口]]
- [[_meta/00-vault-dashboard|知识库总控台]]
- [[_meta/catalog|全量目录]]
- [[_meta/obsidian-operating-logic|Obsidian 运行逻辑]]
- [[全球咨询调研/README|全球咨询调研]]
- [[agents/index|Agent 资料入口]]
- [[plans/index|计划入口]]
- [[记忆治理/README|记忆治理入口]]（如不存在，后续生成）

## 2. 目录职责

| 目录 | 职责 | 写入者 | 备注 |
|---|---|---|---|
| `agents/openclaw/` | OpenClaw 身份、规则、工具、用户偏好镜像 | OpenClaw | 避免直接塞入每日流水 |
| `agents/hermes/` | Hermes 身份、记忆、用户偏好镜像 | Hermes | 避免重复维护多个 MEMORY 副本 |
| `全球咨询调研/` | 全球新闻/事实/专题/日报/周报 | OpenClaw + Hermes | 已有固定分工 |
| `资讯更新/` | 按领域保存资讯类日报 | 定时任务 | 作为素材层，不直接当最终总结 |
| `记忆治理/` | 记忆质量巡检、重复/过期/矛盾项 | OpenClaw + Hermes | 不自动删除，只提出建议 |
| `技能治理/` | Skill/能力治理、维护报告 | OpenClaw + Hermes | skill apply 状态要明确 pending/live |
| `dashboards/` | dashboard/monitor 设计与调研 | OpenClaw | 偏项目资料 |
| `plans/` | 中长期计划和方案 | 用户 + Agent | 不能替代执行记录 |
| `_meta/` | 索引、目录、规则、重复/孤立报告 | 自动/半自动 | 不放业务正文 |
| `shared-memory/` | shared-memory 导出和脱敏版本 | 自动任务 | 禁止写入原始密钥 |

## 3. 三层知识模型

### 3.1 素材层

只负责保存原始或半原始信息，允许较长、较散。

典型目录：

- `资讯更新/`
- `全球咨询调研/02-OpenClaw事实数据/`
- `全球咨询调研/03-Hermes新闻解读/`
- `shared-memory/exports/`

要求：

- 文件名包含日期和主题。
- 保留来源路径/链接。
- 不要求写最终结论。

### 3.2 总结层

把同一天/同一主题的素材压缩成可阅读结论。

典型目录：

- `全球咨询调研/01-每日汇总/`
- `全球咨询调研/05-周报月报/`
- `hermes-openclaw-learning/daily-merged/`（workspace 内生成，必要时可同步摘要到 vault）
- `记忆治理/`

要求：

- 一天一个主总结，避免多个“最终版”。
- 明确“采用 / 暂不采用 / 需验证”。
- 结尾附来源清单。

### 3.3 决策层

保存长期有效的偏好、原则、经验、约束。

典型目录：

- `plans/`
- `技能治理/`
- `agents/*/USER.md`
- `agents/*/MEMORY.md`
- OpenClaw wiki syntheses（独立于 vault，但可以在此处放摘要链接）

要求：

- 不放大段日志。
- 每条决策尽量有日期和来源。
- 矛盾项标记 `needs-review`，不要静默覆盖。

## 4. 每日生成文档流转规则

### 4.1 OpenClaw 每日学习/事实任务

优先写入：

```text
/root/.openclaw/workspace/full-project-learning-report/daily-YYYY-MM-DD.md
/root/obsidian-vault/全球咨询调研/02-OpenClaw事实数据/YYYY-MM-DD-OpenClaw事实数据.md
```

同时要求：

- 只写事实、数据、来源和可落地线索。
- 不在素材文档里直接覆盖长期计划。

### 4.2 Hermes 每日解读/总结任务

优先写入：

```text
/root/obsidian-vault/全球咨询调研/03-Hermes新闻解读/YYYY-MM-DD-Hermes新闻解读.md
/root/obsidian-vault/全球咨询调研/01-每日汇总/YYYY-MM-DD-全球调研日报.md
```

同时要求：

- 合并 OpenClaw 素材后再总结。
- 标出“明确事实”和“模型推断”。

### 4.3 OpenClaw + Hermes 学习文档合并任务

每天 05:00 的合并任务输出：

```text
/root/.openclaw/workspace/hermes-openclaw-learning/daily-merged/YYYY-MM-DD-openclaw-hermes-learning-summary.md
```

Obsidian 侧只需要记录摘要或索引，避免重复复制全文。建议后续建立：

```text
/root/obsidian-vault/记忆治理/YYYY-MM-DD-OpenClaw-Hermes学习合并摘要.md
```

内容包括：

- 来源文档数。
- 最重要 5 条结论。
- 需要进入经验卡/失败库/成功模式/skill 候选的项目。
- 链接到 workspace 原文路径。

## 5. 命名规范

### 5.1 日报

```text
YYYY-MM-DD-主题.md
```

例：

```text
2026-06-24-OpenClaw事实数据.md
2026-06-24-Hermes新闻解读.md
2026-06-24-全球调研日报.md
```

### 5.2 周报

```text
YYYY-WW-主题周报.md
```

例：

```text
2026-W26-全球趋势周报.md
```

### 5.3 运维/故障/经验

```text
YYYY-MM-DD-系统-事件-类型.md
```

例：

```text
2026-06-24-Hermes-Gateway-systemd接管-经验.md
```

## 6. Frontmatter 规范

建议所有新生成总结类文档包含：

```yaml
---
type: daily-summary | weekly-summary | source-note | operation-note | memory-governance | skill-governance
agent: openclaw | hermes | joint
created_at: YYYY-MM-DDTHH:mm:ssZ
updated_at: YYYY-MM-DDTHH:mm:ssZ
date: YYYY-MM-DD
status: draft | active | needs-review | archived
source_paths:
  - /path/to/source.md
tags:
  - openclaw
  - hermes
---
```

规则：

- `type` 用于 Dataview/搜索聚合。
- `source_paths` 只写路径/链接，不贴敏感原文。
- 不确定内容加 `status: needs-review`。

## 7. 链接规则

优先使用 Obsidian wikilink：

```markdown
[[全球咨询调研/README|全球咨询调研]]
[[agents/openclaw/index|OpenClaw]]
[[agents/hermes/index|Hermes]]
```

外部或 workspace 文件用普通路径代码块，不强行 wikilink：

```text
/root/.openclaw/workspace/hermes-openclaw-learning/reports/2026-06-24-eval-report.md
```

原因：workspace 不一定在 Obsidian vault 内，强行 wikilink 会形成坏链接。

## 8. 去重与合并规则

当同一天同主题出现多份文件：

1. 不删除旧文件。
2. 选择一个“主总结”。
3. 在主总结的“来源清单”里链接其它文件。
4. 在 `_meta/duplicates.md` 或 `needs-review` 记录重复原因。
5. 下次生成时只更新主总结，不继续生成“最终版2/修正版/新版”。

优先级：

```text
用户明确指定 > 总结层文档 > 素材层文档 > 自动导出/缓存
```

## 9. 安全规则

- 不写入 API key、token、cookie、完整 Authorization header。
- 日志中如必须引用请求，只保留状态码、路径、非敏感 IP、时间和错误类型。
- Git 同步前确认仓库是私有；不确定时不要自动 push。
- 对 `MEMORY.md`、`USER.md`、`SOUL.md` 的修改必须保守，避免覆盖手写内容。

## 10. 推荐每日节奏

| 时间 | 任务 | 输出 |
|---|---|---|
| 02:00 | 项目/开源学习 | `full-project-learning-report/` |
| 05:00 | OpenClaw + Hermes 学习合并 | `hermes-openclaw-learning/daily-merged/` |
| 08:00 | OpenClaw 全球事实数据 | `全球咨询调研/02-OpenClaw事实数据/` |
| 21:00 | Hermes 新闻解读/日报 | `全球咨询调研/03-Hermes新闻解读/` + `01-每日汇总/` |
| 周日 | 周报/月报/经验沉淀 | `05-周报月报/` + `技能治理/` + `记忆治理/` |

## 11. 后续优化待办

- [ ] 给 `记忆治理/` 建 README 和索引。
- [ ] 给 `技能治理/` 建 live/pending/blocked 状态表。
- [ ] 为每日 05:00 合并任务同步一份 Obsidian 摘要。
- [ ] 定期刷新 `_meta/catalog.md`、`_meta/orphans.md`、`_meta/duplicates.md`。
- [ ] 把 workspace 内重要经验卡摘要链接进 Obsidian，而不是复制多份全文。
