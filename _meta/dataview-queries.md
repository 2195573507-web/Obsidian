---
title: "Dataview 查询集合"
tags: ['meta', 'dataview']
updated: 2026-06-24
---

# Dataview 查询集合

本文件包含针对本知识库常用查询的 Dataview 代码块，可直接复制到其他页面使用。

---

## 1. 按标签查询所有文件

查询指定标签的文件并按更新时间倒序排列。修改 `#news` 为目标标签即可。

```dataview
TABLE file.tags AS "标签", file.mtime AS "更新时间", file.folder AS "目录"
FROM #news
SORT file.mtime DESC
LIMIT 50
```

查询所有带 `#agent` 标签的文件：

```dataview
TABLE file.tags AS "标签", file.mtime AS "更新时间"
FROM #agent
SORT file.mtime DESC
```

多标签过滤（含 `#news` 且含 `#daily`）：

```dataview
TABLE file.tags AS "标签", file.ctime AS "创建时间"
FROM #news AND #daily
SORT file.mtime DESC
LIMIT 30
```

---

## 2. 最近 7 天更新的文件

```dataview
TABLE file.tags AS "标签", file.mtime AS "最后修改", file.folder AS "目录"
FROM ""
WHERE file.mtime >= date(today) - dur(7 days)
SORT file.mtime DESC
LIMIT 100
```

按目录分组显示：

```dataview
LIST
FROM ""
WHERE file.mtime >= date(today) - dur(7 days)
GROUP BY file.folder
SORT file.mtime DESC
```

---

## 3. 缺少 Frontmatter 的文件

查找没有标准 `title` 或 `tags` 字段的文件：

```dataview
TABLE file.path AS "路径", file.size AS "大小", file.mtime AS "修改时间"
FROM ""
WHERE !title OR !tags
SORT file.mtime DESC
```

仅查找完全无 frontmatter 的文件：

```dataview
TABLE file.folder AS "目录", file.size AS "大小", file.mtime AS "修改时间"
FROM ""
WHERE length(file.frontmatter) = 0
SORT file.size DESC
LIMIT 50
```

---

## 4. 查询所有决策类记忆

查找标记为 `decision` 类型的记忆条目：

```dataview
TABLE title AS "标题", tags AS "标签", updated AS "更新日期"
FROM "agents" OR "memory" OR "shared-memory"
WHERE type = "decision" OR contains(tags, "decision")
SORT updated DESC
```

包含决策和偏好类记忆：

```dataview
TABLE title AS "标题", type AS "类型", tags AS "标签", updated AS "更新"
FROM "agents" OR "memory" OR "shared-memory"
WHERE type = "decision" OR type = "preference" OR contains(tags, "decision") OR contains(tags, "preference")
SORT updated DESC
```

---

## 5. 按计划状态查询

查找所有计划类文件并按状态分组：

```dataview
TABLE title AS "标题", status AS "状态", updated AS "更新", tags AS "标签"
FROM ""
WHERE type = "plan" OR contains(tags, "plan")
SORT updated DESC
```

按状态分组显示：

```dataview
LIST
FROM ""
WHERE type = "plan" OR contains(tags, "plan")
GROUP BY default(status, "未定义")
```

---

## 6. 按日期范围查询每日资讯

查询特定日期范围内的每日资讯文件：

```dataview
TABLE title AS "标题", file.tags AS "标签", file.mtime AS "更新时间"
FROM "资讯更新"
WHERE contains(tags, "daily") AND file.day >= date("2026-06-01") AND file.day <= date("2026-06-30")
SORT file.day DESC
```

最近 7 天的资讯汇总：

```dataview
TABLE title AS "标题", file.folder AS "分类", file.mtime AS "更新时间"
FROM "资讯更新"
WHERE contains(tags, "daily") AND file.mtime >= date(today) - dur(7 days)
SORT file.day DESC
```

按资讯分类统计文件数量：

```dataview
TABLE length(rows) AS "文章数", min(rows.file.day) AS "最早", max(rows.file.day) AS "最新"
FROM "资讯更新"
WHERE contains(tags, "daily")
GROUP BY file.folder
SORT length(rows) DESC
```

---

## 附加：常用 TASK 查询

查找所有未完成的任务：

```dataview
TASK
FROM ""
WHERE !completed
SORT file.mtime DESC
LIMIT 50
```

按文件列出待办事项：

```dataview
TASK
FROM "agents" OR "memory"
WHERE !completed
GROUP BY file.link
```
