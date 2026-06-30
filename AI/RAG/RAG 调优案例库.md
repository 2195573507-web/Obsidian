---
title: RAG 调优案例库
type: wiki
category: ai-rag
updated: 2026-06-30
tags: [rag, retrieval, tuning]
source: hermes-multi-agent-broad-expansion-wave3
---

# RAG 调优案例库

> 关联：[[AI/RAG、知识库与记忆系统工程|RAG、知识库与记忆系统工程]]、[[AI/LLM 评测体系与黄金测试集|LLM 评测体系与黄金测试集]]

## 1. 调优顺序

不要一开始换 embedding。推荐顺序：文档质量 → chunk → metadata → query rewrite → hybrid search → rerank → context packing → 评测。

## 2. 常见问题

| 现象 | 可能原因 | 处理 |
|---|---|---|
| 检索不到精确错误码 | 纯向量不擅长精确词 | 加 BM25 / hybrid |
| 答案引用错 | chunk 无标题路径 | chunk 带 source/section |
| 长文档答偏 | chunk 太大或噪声多 | parent-child chunk |
| 中文召回差 | 分词/embedding 不适配 | 混合检索、多 query |
| 权限泄露 | 检索后过滤 | 检索前权限过滤 |

## 3. 调试模板

```markdown
## RAG 调试记录
- 问题：
- 期望文档：
- 实际 top_k：
- 是否命中：
- rerank 后：
- 最终答案问题：
- 修改动作：
- 回归结果：
```

## 4. 黄金问题集

每个重要文档至少配 3 类问题：精确事实、流程步骤、边界拒答。记录 expected_sources 和 forbidden hallucinations。

## 5. 上下文包装

每个片段格式：id、source、title_path、updated_at、score、content。模型回答必须引用 id。
