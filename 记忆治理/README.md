---
type: index
category: memory-governance
status: active
created_at: 2026-06-24T01:56:00Z
updated_at: 2026-06-24T01:56:00Z
tags:
  - memory-governance
  - openclaw
  - hermes
---

# 记忆治理入口

> 用于汇总 OpenClaw、Hermes、shared-memory、Obsidian vault 的记忆质量巡检、重复内容、过期信息、矛盾项和合并建议。

## 入口

- [[_meta/obsidian-operating-logic|Obsidian 运行逻辑]]
- [[_meta/duplicates|重复内容报告]]
- [[_meta/needs-review|待整理清单]]
- [[shared-memory/README|Shared Memory 导出]]
- [[agents/openclaw/index|OpenClaw 记忆入口]]
- [[agents/hermes/index|Hermes 记忆入口]]

## 处理原则

1. 不自动删除记忆。
2. 不覆盖用户手写内容。
3. 对重复内容优先“合并引用”，而不是复制全文。
4. 对矛盾内容标记 `needs-review`，等待确认。
5. 对敏感内容只记录“存在风险/已脱敏”，不记录原文。

## 推荐状态

| 状态 | 含义 |
|---|---|
| `keep` | 保留 |
| `merge` | 可合并 |
| `expire` | 可能过期，待确认 |
| `rewrite` | 需要改写为更短、更准 |
| `quarantine` | 需要隔离，不能直接使用 |
| `investigate` | 信息不足，需要继续查证 |

## 现有巡检记录

```dataview
TABLE file.mtime AS 修改时间
FROM "记忆治理"
WHERE file.name != "README"
SORT file.mtime DESC
```

> 如果 Dataview 不可用，可直接浏览本目录下按日期命名的 Markdown 文件。
