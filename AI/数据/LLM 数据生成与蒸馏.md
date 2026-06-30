---
title: LLM 数据生成与蒸馏
type: wiki
category: ai-data
updated: 2026-06-30
tags: [dataset, distillation, sft, dpo]
source: hermes-multi-agent-broad-expansion-wave3
---

# LLM 数据生成与蒸馏

> 关联：[[AI/AI Agent 与 LLM 工程总览|AI Agent 与 LLM 工程总览]]、[[软件工程/数据科学基础与工程衔接|数据科学基础与工程衔接]]

## 1. 数据类型

SFT：指令到理想回答。DPO：chosen/rejected 偏好对。Eval：评测题、标准答案、rubric。Agent Trace：任务、工具调用、观察、最终答案。

## 2. JSONL 规范

```json
{"id":"task_001","instruction":"...","response":"...","metadata":{"source":"synthetic","quality_score":0.92}}
```

DPO：

```json
{"prompt":"...","chosen":"...","rejected":"...","reason":"..."}
```

## 3. 质量门槛

- schema 校验；
- 去重；
- 长度范围；
- 禁止空回答；
- 安全过滤；
- 质量评分；
- 抽样人工复核。

## 4. 生成流水线

任务池 → 并发 teacher model → 解析 → 质量评分 → 失败隔离 → JSONL → eval 抽检 → 版本化。

## 5. 常见坑

字段类型不一致导致 pyarrow 失败；坏任务重复消耗并发；上传失败阻塞本地生成；数据没有版本；eval 泄漏到训练集。
