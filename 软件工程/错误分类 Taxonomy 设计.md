---
title: "错误分类 Taxonomy 设计"
type: wiki
category: "软件工程"
updated: 2026-07-05
managed_by: Hermes
tags: [errors, taxonomy, observability]
---

# 错误分类 Taxonomy 设计

> 上级入口：[[软件工程/软件工程 MOC|软件工程 MOC]]、[[运维/Provider 健康矩阵与模型路由巡检|Provider 健康矩阵与模型路由巡检]]。

## 目标

统一错误分类，让日志、指标、告警、用户提示和复盘使用同一套语言。没有 taxonomy 时，所有失败都会退化成“请求失败”，难以定位和自动处理。

## 推荐分类

| error_type | 含义 | 示例 |
|---|---|---|
| upstream_unavailable | 上游不可用 | 502、503 |
| upstream_timeout | 上游超时 | request timeout |
| rate_limited | 限流 | 429、quota |
| auth_failed | 鉴权失败 | 401、403 |
| validation_failed | 参数或结果校验失败 | schema error |
| tool_failed | 工具执行失败 | file not found |
| delivery_failed | 投递失败 | message not delivered |
| policy_blocked | 策略阻断 | approval required |
| unknown | 未分类 | 兜底 |

## 设计原则

- 分类数量控制在可理解范围。
- 用户提示可读，内部字段稳定。
- error_type 可作为低基数 metrics label。
- 原始错误保留到日志，不直接暴露给用户。
- 分类变化要有迁移说明。

## 关联入口

- [[软件工程/API 错误模型与响应规范|API 错误模型与响应规范]]
- [[运维/LLM 上游故障处置手册|LLM 上游故障处置手册]]
- [[产品/Agent 产品失败兜底体验|Agent 产品失败兜底体验]]
