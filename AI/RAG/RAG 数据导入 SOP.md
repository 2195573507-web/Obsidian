---
title: RAG 数据导入 SOP
type: wiki
category: ai-rag
updated: 2026-06-30
tags: [rag, data-ingestion, sop]
source: hermes-multi-agent-broad-expansion-wave4
---

# RAG 数据导入 SOP

> 关联：[[AI/RAG 调优案例库|RAG 调优案例库]]

## 流程

1. 盘点来源；2. 清洗格式；3. 添加 metadata；4. chunk；5. embedding；6. 写入向量库；7. 建索引；8. 跑黄金问题；9. 发布。

## Metadata

```yaml
source_path:
title:
section:
updated_at:
permission:
tags:
checksum:
```

## 校验

- [ ] 文档数量符合预期；
- [ ] chunk 有标题路径；
- [ ] 权限字段存在；
- [ ] 检索能命中已知问题；
- [ ] 引用链接可打开；
- [ ] 旧版本可回滚。
