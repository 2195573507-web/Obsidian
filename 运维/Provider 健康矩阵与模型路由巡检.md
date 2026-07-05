---
title: "Provider 健康矩阵与模型路由巡检"
type: wiki
category: "运维"
updated: 2026-07-05
managed_by: Hermes
tags: [llm, gateway, monitoring, routing]
---

# Provider 健康矩阵与模型路由巡检

> 上级入口：[[运维/运维 MOC|运维 MOC]]、[[AI/模型路由、成本控制与降级|模型路由、成本控制与降级]]、[[运维/LLM 上游故障处置手册|LLM 上游故障处置手册]]。

## 目标

Provider 健康矩阵用于回答：哪个 provider/model 正在失败、慢在哪里、fallback 是否生效、是否应该暂停批量任务。它是 LLM Gateway 治理面的核心观测视图。

## 指标维度

| 维度 | 指标 | 用途 |
|---|---|---|
| 可用性 | success rate、error rate、timeout rate | 判断是否可用 |
| 延迟 | P50/P95/P99、first token latency | 判断是否变慢 |
| 流式 | chunk gap、active streams、client cancel | 判断 streaming 是否卡住 |
| 限流 | 429、quota、retry-after | 判断是否需要降并发 |
| 成本 | input/output token、单次成本 | 判断是否需要降级 |
| fallback | fallback count、fallback success rate | 判断路由是否有效 |

## 巡检步骤

1. 先按 provider 聚合成功率和错误率。
2. 再按 model 细分，避免一个 provider 下只有部分模型异常。
3. 区分普通请求和 streaming 请求。
4. 对 cron/report 任务单独查看 last success 与 failure reason。
5. 连续失败时先暂停补跑，再验证轻量模型调用。

## 告警建议

- 5 分钟成功率低于 80%：黄色。
- 连续 3 次 cron/report 任务因同一 provider 失败：橙色。
- 所有 fallback 均失败：红色。
- active streams 异常累积且无 chunk：红色。

## 关联入口

- [[运维/可观测性与日志体系|可观测性与日志体系]]
- [[运维/Cron 定时任务补跑 Runbook|Cron 定时任务补跑 Runbook]]
- [[数据/知识库事件与变更日志模型|知识库事件与变更日志模型]]
