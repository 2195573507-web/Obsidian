---
title: "流式 API 可观测性指标"
type: wiki
category: "运维"
updated: 2026-07-05
managed_by: Hermes
tags: [streaming, observability, api]
---

# 流式 API 可观测性指标

> 上级入口：[[运维/运维 MOC|运维 MOC]]、[[软件工程/API 设计规范与错误码|API 设计规范与错误码]]。

## 背景

Streaming API 不能只看 HTTP 请求总耗时。一个请求可能连接成功但首 token 极慢，或中途 chunk 长时间断流，或客户端取消后上游仍在消耗资源。

## 核心指标

- `first_token_latency_ms`：从请求开始到首个 chunk。
- `chunk_gap_ms`：相邻 chunk 最大间隔。
- `active_streams`：当前活跃流数量。
- `client_cancel_total`：客户端主动取消次数。
- `upstream_timeout_total`：上游超时次数。
- `stream_duration_ms`：完整流生命周期。
- `stream_bytes`：输出字节数。

## 分层排查

1. 首 token 慢：模型排队、provider 慢、上下文过大。
2. chunk gap 大：上游卡顿、代理 buffering、网络抖动。
3. client cancel 高：前端超时、用户体验差、响应过慢。
4. active streams 不降：泄漏、连接未关闭、上游未中止。

## 标签基数原则

低基数字段可做 label：provider、model、route、status、error_type。高基数字段不得做 label：request_id、prompt、用户输入、token、文件路径、完整 URL。

## 关联入口

- [[运维/Provider 健康矩阵与模型路由巡检|Provider 健康矩阵与模型路由巡检]]
- [[软件工程/日志错误处理规范|日志错误处理规范]]
- [[安全/Web API 密钥与日志脱敏规范|Web API 密钥与日志脱敏规范]]
