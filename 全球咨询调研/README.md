---
title: "全球咨询调研"
tags: ['research', 'daily-news']
updated: 2026-06-24
---

# 全球咨询调研

> 创建时间：2026-06-21 03:11 UTC
> 用途：统一存放 OpenClaw 与 Hermes 的每日全球调研、领域总结、周报和月报。

## 目录

- [[00-任务说明/任务分工]]：两个 Agent 的固定分工与时间
- [[01-每日汇总/日报模板]]：最终日报模板
- [[02-OpenClaw事实数据/OpenClaw每日任务]]：OpenClaw 每日事实/数据调研
- [[03-Hermes新闻解读/Hermes每日任务]]：Hermes 每日新闻/解读/汇总
- [[04-领域专题/领域清单]]：长期专题领域
- [[05-周报月报/周报模板]]：周报/月报模板
- [[_meta/obsidian-operating-logic|Obsidian 运行逻辑]]：文档分层、命名、去重、每日合并规则

## 每日节奏

| 时间 | Agent | 任务 |
|---|---|---|
| 08:00 | OpenClaw | 全球事实、数据、来源、领域素材 |
| 21:00 | Hermes | 汇总解读，生成《全球调研日报》 |
| 周日 21:30 | Hermes | 生成《全球趋势周报》 |

## 输出命名

- OpenClaw：`02-OpenClaw事实数据/YYYY-MM-DD-OpenClaw事实数据.md`
- Hermes：`03-Hermes新闻解读/YYYY-MM-DD-Hermes新闻解读.md`
- 日报：`01-每日汇总/YYYY-MM-DD-全球调研日报.md`
- 周报：`05-周报月报/YYYY-WW-全球趋势周报.md`

## 合并原则

- `02-OpenClaw事实数据/` 和 `03-Hermes新闻解读/` 是素材/分析层。
- `01-每日汇总/` 是当天唯一主总结，避免生成多个最终版。
- 每天 05:00 的 OpenClaw/Hermes 学习合并任务输出在 workspace，Obsidian 侧只保留摘要或索引，避免重复复制全文。
- 详细规则见 [[_meta/obsidian-operating-logic|Obsidian 运行逻辑]]。
