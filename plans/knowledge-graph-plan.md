---
source_path: /root/.openclaw/workspace/knowledge-graph-plan.md
imported_at: 2026-06-19T13:19:45
category: plans
---

# 知识图谱 Web 页 — 实施计划

> 目标：在 Hermes Dashboard (:9100) 加一个 Tab，像 Obsidian Graph View 一样，
> 可视化展示服务器上所有 .md 文件之间的引用关系。

---

## 一、页面效果

```
┌─────────────────────────────────────────────────┐
│  📊 总览 │ ⚙️ 配置 │ 📈 用量 │ 🌐 网关 │       │
│  💬 会话 │ 🕸️ 知识图谱                          │  ← 新增 Tab
├─────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────────────────┐ │
│  │  力导向图      │  │  侧边栏                  │ │
│  │               │  │  📄 选中节点：MEMORY.md  │ │
│  │  ● 节点 = 文件 │  │  引用数：3              │ │
│  │  ─ 边 = 引用   │  │  被引用数：2            │ │
│  │  节点大小=引   │  │  [📂 打开文件]          │ │
│  │  用次数        │  │                        │ │
│  │               │  │  📂 目录过滤：          │ │
│  │  可拖拽/缩放   │  │  ☑ openclaw            │ │
│  │               │  │  ☑ hermes              │ │
│  │               │  │  ☐ .git                │ │
│  │               │  │                        │ │
│  │               │  │  🔗 连接列表：          │ │
│  │               │  │  MEMORY.md → SOUL.md   │ │
│  │               │  │  MEMORY.md → USER.md   │ │
│  └──────────────┘  └──────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

---

## 二、技术选型

| 层 | 选型 | 理由 |
|---|---|---|
| 后端 | FastAPI（复用 dashboard 的 backend.py） | 已有框架，零新增依赖 |
| 图谱渲染 | ECharts force graph | Dashboard 已加载 ECharts CDN，免额外引入 |
| 数据来源 | 扫描 `/root` 下的 `.md` 文件 | 有 30+ 个 .md 文件，后续可配路径 |

---

## 三、后端改动（backend.py）

### 新增端点：`GET /api/knowledge-graph`

**逻辑**：
```
1. 扫描指定目录下所有 .md 文件（排除 node_modules / .git / __pycache__）
2. 读取每个文件，正则提取引用：
   - [[wikilink]] 格式
   - [text](path.md) 格式
3. 构建节点列表 nodes + 边列表 links
4. 返回 JSON
```

**响应格式**：
```json
{
  "nodes": [
    {
      "id": "/root/obsidian-vault/MEMORY.md",
      "name": "MEMORY.md",
      "category": "memory",
      "symbolSize": 35,
      "path": "/root/obsidian-vault/MEMORY.md",
      "lines": 120
    }
  ],
  "links": [
    {
      "source": "/root/obsidian-vault/MEMORY.md",
      "target": "/root/obsidian-vault/SOUL.md",
      "label": "引用"
    }
  ],
  "stats": {
    "total_files": 32,
    "total_links": 47,
    "orphan_count": 5
  }
}
```

**改动量**：~60 行 Python

---

## 四、前端改动（index.html）

### 4.1 新增 Tab 按钮
在 tab-nav 里加一个：
```html
<button class="tab-btn" data-tab="knowledge">
  <span class="tab-icon">🕸️</span> 知识图谱
</button>
```

### 4.2 新增 Tab 面板
```html
<div class="tab-panel" id="panel-knowledge">
  <!-- 图谱区域 (左侧 70%) -->
  <!-- 侧边栏 (右侧 30%) -->
</div>
```

### 4.3 ECharts 力导向图配置
- 节点颜色按目录分类（openclaw=蓝, hermes=紫, 其他=绿）
- 节点大小 = 5 + 引用次数 × 3
- 边用箭头表示引用方向
- 支持拖拽 / 缩放 / hover 高亮
- 点击节点显示侧边栏详情

### 4.4 tabLoaders 注册
```js
knowledge: loadKnowledge
```

**改动量**：~120 行 HTML/CSS/JS

---

## 五、文件改动清单

| 文件 | 改动 | 行数 |
|---|---|---|
| `/opt/hermes-dashboard/backend.py` | 新增 `GET /api/knowledge-graph` 端点 | +60 |
| `/opt/hermes-dashboard/static/index.html` | 新增 Tab + 面板 + ECharts 绑定 | +120 |

---

## 六、预计效果

| 指标 | 预期 |
|---|---|
| 节点数 | 30~50（当前服务器 .md 文件） |
| 边数 | 20~80（取决于引用密度） |
| 渲染性能 | ECharts 力导向在 200 节点内流畅 |
| 加载时间 | < 500ms（30 个文件读取） |

---

## 七、Phase 2 扩展（先不做）

- 节点预览：悬停显示文件前 3 行
- 按标签过滤：日期 / 项目 / 类型
- 图谱导出 PNG
- 实时更新：文件变化时自动刷新图谱

---

## 八、风险和注意事项

| 风险 | 处理 |
|---|---|
| 大文件拖慢渲染 | 只取文件名和引用行，不传全文 |
| wikilink 匹配不精确 | 正则宽松匹配，宁可多连不错过 |
| 非 vault 文件干扰 | 提供目录过滤开关 |
| 并发文件读取 | 30 个文件一次性读完，耗时可控 |
