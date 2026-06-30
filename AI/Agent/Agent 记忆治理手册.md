---
title: Agent 记忆治理手册
type: wiki
category: ai-agent
updated: 2026-06-30
tags: [memory, governance]
source: hermes-multi-agent-broad-expansion-wave4
---

# Agent 记忆治理手册

> 关联：[[AI/RAG、知识库与记忆系统工程|RAG、知识库与记忆系统工程]]、[[shared-memory/README|共享记忆说明]]

## 记忆类型

偏好、事实、服务状态、决策、经验、待办、临时状态。每条记忆应有来源、时间、敏感级别、重要性、过期策略。

## 写入原则

写长期稳定信息，不写密钥、不写一次性日志、不写未验证猜测、不写短期进度。mutable facts 必须带 checked_at。

## 审计

定期检查重复、冲突、过期、低价值噪音、敏感泄漏。重要删除/合并先生成候选报告。

## 冲突处理

新旧事实冲突时优先 live check；用户明确纠正高于模型总结；过期状态转历史。
