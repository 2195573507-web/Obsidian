---
title: RAG、知识库与记忆系统工程
type: deep-wiki
category: ai
updated: 2026-06-30
tags: [rag, knowledge-base, memory, embedding]
source: hermes-deep-expansion
---

# RAG、知识库与记忆系统工程

> 关联：[[AI/AI Agent 与 LLM 工程总览|AI Agent 与 LLM 工程总览]]、[[AI/RAG/RAG 调优案例库|RAG 调优案例库]]、[[AI/RAG/RAG 数据导入 SOP|RAG 数据导入 SOP]]、[[shared-memory/README|共享记忆说明]]

## 1. RAG 的本质

RAG（Retrieval-Augmented Generation）是把“知识获取”和“语言生成”拆开的架构。模型不需要把所有知识记在参数里，而是在回答前从外部知识库检索相关资料，再基于资料生成答案。

```text
用户问题 → 查询理解 → 检索 → 重排序 → 上下文构造 → LLM 生成 → 引用与校验
```

RAG 的优势是知识可更新、可溯源、可权限控制；缺点是系统复杂度从“一个模型调用”变成“搜索系统 + 数据治理 + 生成系统 + 评测系统”。

## 2. RAG 不是什么

RAG 不是万能记忆，不是数据库，不是搜索框套壳，也不是微调替代品。它无法自动修复混乱文档，无法保证检索一定正确，也不能在权限设计错误时保护敏感数据。

| 误解 | 正确认知 |
|---|---|
| 长上下文能替代 RAG | 长上下文成本高且噪声多，RAG 仍负责选择信息 |
| 换 embedding 就能提升质量 | 文档清洗和 chunk 往往更重要 |
| 检索到了就不会幻觉 | 模型可能误读、遗漏或编造 |
| RAG 等于向量库 | RAG 包含数据、检索、生成、引用、评测和权限 |

## 3. 数据源治理

常见数据源包括 Markdown、PDF、网页、代码仓库、工单、数据库、聊天记录、日志、表格和图片 OCR。每种来源都要清洗。

文档进入知识库前至少要做：

- 去除页眉页脚、重复目录、广告导航；
- 保留标题层级；
- 表格转 Markdown；
- 代码块保留语言；
- 添加 source_path、更新时间、权限、tags；
- 计算 checksum；
- 去重；
- 标记敏感等级。

## 4. Chunk 设计

Chunk 是 RAG 的检索单元。切分策略决定召回质量。

### 4.1 固定长度

简单，但可能切断语义。适合结构差、内容均匀的文本。

### 4.2 标题切分

适合 Markdown、技术文档、手册。chunk 应带完整标题路径，例如：`服务器运维 > Nginx > 502 排查`。

### 4.3 Parent-child chunk

小 chunk 用于精准召回，大 parent 用于生成上下文。适合长报告和复杂手册。

### 4.4 代码切分

按文件、类、函数、符号切分，并保留 import、调用关系和路径。

## 5. 检索策略

### 5.1 向量检索

适合语义相似查询，但对错误码、端口、函数名、命令、版本号不稳定。

### 5.2 BM25/关键词

适合精确匹配。技术文档、运维日志、代码问答必须保留关键词检索能力。

### 5.3 Hybrid Search

生产默认推荐 hybrid：向量召回语义，BM25 召回精确词，再融合排序。

### 5.4 Query Rewrite

用户问题常常不适合直接检索。需要结合上下文补全：

```text
“这个 502 怎么办？” → “Nginx 502 Bad Gateway upstream 端口 Docker systemd 排查”
```

### 5.5 Rerank

先召回 top50，再用 reranker 选 top5/top10。rerank 成本高，但对企业知识库质量提升明显。

## 6. 上下文构造

检索结果不能粗暴拼接。应考虑：相关性、多样性、来源可信度、时间新旧、权限、重复、上下文窗口和引用格式。

推荐格式：

```text
<doc id="1" source="运维/Nginx 502 504 专项排查.md" updated="2026-06-30" score="0.82">
内容...
</doc>
```

要求模型回答时引用 doc id，并在资料不足时拒答。

## 7. Memory 与 RAG 的区别

| 维度 | RAG | Memory |
|---|---|---|
| 内容 | 文档知识 | 用户偏好、项目状态、经验 |
| 写入 | 批量导入 | 对话中沉淀 |
| 更新 | 文档版本 | 持续变化 |
| 风险 | 检索错 | 记忆污染 |
| 用途 | 回答知识问题 | 个性化和连续任务 |

RAG 适合“知识”，Memory 适合“上下文”。例如 Nginx 502 排查手册属于 RAG；用户偏好“危险操作先确认”属于 Memory。

## 8. 记忆治理

记忆必须分类：preference、fact、service_state、decision、lesson、todo、temporary。每条记忆需要 importance、sensitivity、source、updated_at、expires_at。mutable facts 必须带 live-check 时间。

不应写入：密钥、token、一次性日志、未验证猜测、短期任务进度、会很快过期的状态。

## 9. RAG 评测

分层评测：

- 检索：Recall@K、MRR、nDCG、Hit Rate；
- 生成：faithfulness、answer relevance、citation accuracy；
- 安全：权限过滤、注入防护、敏感泄露；
- 端到端：用户任务成功率。

黄金问题示例：

```yaml
question: "Nginx 502 如何排查？"
expected_sources:
  - "运维/Nginx 502 504 专项排查.md"
expected_points:
  - "查看 error.log"
  - "检查 proxy_pass"
  - "检查后端端口"
forbidden:
  - "直接重启全部服务"
```

## 10. 常见事故

### 10.1 检索不到

可能是 chunk 太大、metadata 过滤错、query 太口语、embedding 不适合中文、关键词缺失。

### 10.2 答案看似有来源但引用错

可能是拼接时丢失 doc id，或模型引用了相邻片段。需要引用校验。

### 10.3 敏感文档泄露

通常是检索后过滤，而不是检索前过滤。权限必须在召回前参与。

### 10.4 知识过期

需要 updated_at、checksum、重建索引、过期提示和来源优先级。

## 11. 生产清单

- [ ] 数据源清单完整；
- [ ] 文档有 metadata；
- [ ] chunk 策略有评测；
- [ ] hybrid search；
- [ ] rerank 可开关；
- [ ] 权限检索前过滤；
- [ ] 引用可追踪；
- [ ] 资料不足可拒答；
- [ ] 有黄金问题集；
- [ ] 有索引重建和回滚流程。
