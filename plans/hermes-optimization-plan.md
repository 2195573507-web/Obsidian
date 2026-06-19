---
source_path: /root/.openclaw/workspace/hermes-optimization-plan.md
imported_at: 2026-06-19T13:19:45
category: plans
---

# Hermes Dashboard & System Monitor 优化计划

> 分析日期：2026-06-19
> 参考项目：Helicone / LiteLLM / Langfuse / Portkey / Open WebUI / One API / Grafana

---

## 总览

| 面板 | 端口 | 当前状态 | 可优化项 |
|---|---|---|---|
| Hermes Dashboard | 9100 | 7 个 Tab，有 KPI/图表/Provider管理 | 10 项 |
| System Monitor | 9000 | 单页实时监控，SSE + 历史折线 | 7 项 |

---

## 第一阶段：Quick Wins（投入 < 2h，效果显著）

### 1. Dashboard — 全局时间范围选择器
**参考**：Helicone / Langfuse 的统一时间选择器  
**现状**：所有图表硬编码 7 天，无切换  
**改法**：顶部加 `[1h] [24h] [7d] [30d]` 按钮，切换后所有图表 + KPI 同步刷新。后端 API 已有 `days` 参数，前端补 JS 即可  
**改动量**：前端 ~80 行 JS

### 2. System Monitor — 阈值告警通知
**参考**：Grafana AlertManager  
**现状**：卡片有 `warn/crit` 样式但无通知  
**改法**：CPU > 90%/内存 > 90%/磁盘 > 85% 时 toast 弹窗 + 状态栏闪烁 + 浏览器 title 变红  
**改动量**：纯前端 ~60 行 JS

### 3. System Monitor — 磁盘 IO 监控
**参考**：Grafana node_exporter  
**现状**：只有磁盘使用率，无读写速率  
**改法**：`collector.py` 加 `disk_io_counters()` 采集 read_bytes/write_bytes，前端加一个折线图  
**改动量**：后端 ~20 行 + 前端 ~30 行

### 4. Dashboard — 成本趋势图
**参考**：Helicone Cost 页面  
**现状**：数据已有 `estimated_cost_usd`，只显示了 KPI 总数  
**改法**：Overview 加一个每日成本折线图，Usage tab 补按模型成本分解表  
**改动量**：后端 1 个 SQL + 前端 ~50 行

---

## 第二阶段：信息密度提升（投入 ~4h）

### 5. Dashboard — 按模型缓存命中率对比
**参考**：Helicone Cache 分析页  
**现状**：有 overall gauge + trend，但 `by_model` 数据未展示  
**改法**：Usage tab 加按模型的缓存命中率柱状图/表格  
**改动量**：数据已有，前端 ~40 行

### 6. Dashboard — Top 工具 / Top 技能排行
**参考**：Langfuse / One API  
**现状**：Insights 引擎输出里有，前端未做  
**改法**：加两个横向条形图 Top 10，显示调用次数和占比  
**改动量**：后端 1 个 API + 前端 ~60 行

### 7. System Monitor — 启用进程列表
**现状**：代码和 HTML 框架在，但被注释掉了  
**改法**：取消注释，微调样式  
**改动量**：后端已有 `top_processes`，纯前端 ~10 行

### 8. Dashboard — 全局搜索/过滤栏
**参考**：Helicone 过滤器栏  
**现状**：会话列表有搜索，但模型/平台/用量表无过滤  
**改法**：顶部通用过滤器：模型下拉 + 平台下拉 + 关键词搜索  
**改动量**：前端 ~100 行

---

## 第三阶段：深度体验（投入 ~6h）

### 9. Dashboard — 活动热力图
**参考**：Langfuse / GitHub 贡献图  
**现状**：Insights 有 Activity Patterns 数据，前端未用  
**改法**：ECharts heatmap 日历图 + 24h 时段分布图  
**改动量**：后端 1 个 SQL + 前端 ECharts heatmap ~80 行

### 10. Dashboard — 会话详情展开
**参考**：Helicone 请求详情面板  
**现状**：会话列表只有基础字段  
**改法**：点击会话行展开详情弹窗：消息历史、token 分解、成本明细  
**改动量**：后端 1 个端点 + 前端弹窗 ~120 行

### 11. Dashboard — 会话搜索增强（FTS5）
**现状**：已有 `messages_fts` FTS5 表，前端只用简单 LIKE 搜索  
**改法**：接 FTS5 Boolean 全文搜索，支持短语匹配、前缀匹配  
**改动量**：后端改 SQL + 前端搜索栏升级

### 12. Dashboard — 数据导出
**参考**：Helicone / Langfuse  
**改法**：用量/会话数据加「导出 CSV」按钮  
**改动量**：后端 ~20 行 + 前端 ~20 行

---

## 第四阶段：锦上添花（按需做）

### 13. System Monitor — 网络连接数
`psutil.net_connections()` 加 TCP 连接数卡片

### 14. System Monitor — 暗色/亮色主题切换
参考 Open WebUI 主题系统

### 15. System Monitor — 历史数据智能降采样
24h/7d 范围做 avg/max rollup，避免折线图糊

### 16. Dashboard — 环境/Profile 多实例切换
参考 Langfuse 环境标签

### 17. Dashboard — 配置页交互升级
config.yaml 从伪 YAML 文本 → 表单式编辑（参考 One API / LiteLLM）

---

## 优先级矩阵

```
影响大 ┃  1.时间选择器    4.成本趋势图
       ┃  2.告警通知      8.全局搜索
       ┃  3.磁盘IO监控     
       ┃
       ┃  5.缓存命中对比  9.活动热力图
       ┃  6.Top排行       10.会话详情
       ┃  7.进程列表      
       ┃
影响小 ┃  16.Profile切换  13.TCP连接数
       ┃  17.配置交互升级  14.主题切换
       ┃  12.数据导出     15.降采样
       ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
           投入小              投入大
```

**推荐路径**：第一阶段 4 项 → 第二阶段 Top 排行 + 进程列表 → 看反馈再决定第三阶段。
