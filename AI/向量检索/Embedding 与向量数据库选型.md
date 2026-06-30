---
title: Embedding 与向量数据库选型
type: wiki
category: ai-rag
updated: 2026-06-30
tags: [embedding, vector-db, rag]
source: hermes-multi-agent-broad-expansion-wave3
---

# Embedding 与向量数据库选型

> 关联：[[AI/RAG、知识库与记忆系统工程|RAG、知识库与记忆系统工程]]

## 1. Embedding 选择维度

中文能力、多语言、代码能力、上下文长度、维度、成本、延迟、本地部署、批处理吞吐、与 reranker 搭配效果。

## 2. 向量库对比

| 类型 | 代表 | 适合 |
|---|---|---|
| 本地库 | FAISS | 单机、实验 |
| 轻量服务 | Chroma/Qdrant | 中小知识库 |
| 大规模 | Milvus/Weaviate | 高并发、多租户 |
| SQL 扩展 | pgvector | 已有 Postgres 团队 |
| 搜索引擎 | OpenSearch/ES | hybrid search |

## 3. 必备能力

metadata filter、增量更新、删除、备份、权限隔离、批量导入、相似度调试、监控、schema 迁移。

## 4. 选型建议

个人/小团队：pgvector 或 Qdrant。已有 Postgres：pgvector 优先。大规模多租户：Milvus/Qdrant。需要强关键词混合：OpenSearch/ES + vector。
