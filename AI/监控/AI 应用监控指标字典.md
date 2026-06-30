---
title: AI 应用监控指标字典
type: wiki
category: ai-ops
updated: 2026-06-30
tags: [llmops, metrics, monitoring]
source: hermes-multi-agent-broad-expansion-wave4
---

# AI 应用监控指标字典

> 关联：[[AI/模型路由、成本控制与降级|模型路由、成本控制与降级]]

## 指标

| 指标 | 含义 |
|---|---|
| requests_total | 请求量 |
| success_rate | 成功率 |
| latency_p95 | P95 延迟 |
| input_tokens/output_tokens | token 消耗 |
| cost_per_task | 单任务成本 |
| tool_call_count | 工具调用数 |
| tool_error_rate | 工具失败率 |
| retrieval_hit_rate | RAG 命中率 |
| schema_pass_rate | 结构化输出通过率 |
| safety_block_count | 安全拦截次数 |
| human_approval_rate | 人工确认率 |
| hallucination_reported | 用户报告幻觉数 |

## 告警

成本突增、错误率升高、schema 失败、工具超时、RAG 召回为空、安全拦截异常增长。
