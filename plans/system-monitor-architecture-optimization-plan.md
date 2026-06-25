---
title: System Monitor 架构优化计划
type: plan
status: draft
created: 2026-06-25
updated: 2026-06-25
tags: [monitor, server, observability, dashboard, hermes, openclaw]
related:
  - [[MOC|知识库导航]]
  - [[dashboards/system-monitor-dashboard-analysis|System Monitor 分析]]
  - [[agents/planning/Hermes-OpenClaw-智能增强路线图|Hermes / OpenClaw 智能增强路线图]]
---

# System Monitor 架构优化计划

## 1. 目标

把当前 System Monitor 从“单机资源看板”升级为“AI 可读、可告警、可追溯的服务器运维中枢”。

核心目标：

1. 监控系统资源：CPU、内存、磁盘、网络、负载。
2. 监控关键 AI 服务：Hermes、OpenClaw、QQBot、shared-memory、Dashboard。
3. 监控中转站健康：连通性、延迟、错误码、模型可用性。
4. 提供给 Hermes / OpenClaw 查询的健康 API。
5. 异常时通过 QQ 告警，并生成可归档的诊断报告。

---

## 2. 当前架构假设

当前 monitor 大概率是轻量架构：

```text
FastAPI 后端
  ├─ psutil 采集系统指标
  ├─ SQLite 保存历史数据
  ├─ API 输出 latest/history/status
  └─ static/index.html + ECharts 展示图表
```

优点：

- 部署简单。
- 资源占用低。
- 单机足够稳定。
- 不依赖外部服务。

主要短板：

- 采集、API、前端职责耦合。
- 只偏系统资源，不够贴合 Hermes / OpenClaw。
- 告警能力弱或缺失。
- 历史数据缺少分层聚合。
- Agent 无法方便地读取结构化健康状态。

---

## 3. 目标架构

建议目标架构：

```text
collector 采集器
  ↓
metrics storage 指标库
  ↓
health API 状态 API
  ↓
Dashboard 前端
  ↓
alert manager 告警器
  ↓
QQ / Hermes / OpenClaw / Obsidian
```

更具体：

```text
systemd service: system-monitor

├─ collector
│  ├─ system metrics
│  ├─ service health checks
│  ├─ docker metrics
│  ├─ nginx status/log stats
│  └─ provider/API health checks
│
├─ storage
│  ├─ raw metrics, high precision
│  ├─ minute aggregates
│  ├─ hourly aggregates
│  └─ alert events
│
├─ API
│  ├─ /api/system/latest
│  ├─ /api/system/history
│  ├─ /api/services/health
│  ├─ /api/providers/health
│  ├─ /api/alerts/recent
│  └─ /api/agent/summary
│
├─ Dashboard
│  ├─ 系统资源
│  ├─ AI 服务状态
│  ├─ 中转站健康
│  ├─ Docker / systemd 状态
│  ├─ 告警历史
│  └─ 诊断报告入口
│
└─ alert manager
   ├─ rule engine
   ├─ cooldown / dedupe
   ├─ QQ notify
   └─ Obsidian report writer
```

---

## 4. 模块优化计划

### 4.1 采集层独立

把采集逻辑从 Web API 中拆出来，形成独立 collector。

采集内容：

- CPU 使用率、核心数、load average。
- 内存、swap。
- 根分区与数据盘使用率。
- 网络上下行速率、累计流量、峰值。
- 磁盘 IO。
- 关键端口存活。
- systemd 服务状态。
- Docker 容器状态。

建议采集间隔：

| 类型 | 间隔 |
|---|---:|
| 系统资源 | 2-5 秒 |
| 服务健康 | 10-30 秒 |
| 中转站健康 | 1-5 分钟 |
| 日志聚合 | 1 分钟 |
| 长期清理 | 1 天 |

---

### 4.2 存储层分层

SQLite 可以继续保留，但需要分层：

```text
raw_metrics          近 24 小时，高精度
metrics_1m           近 7 天，分钟聚合
metrics_1h           近 30/90 天，小时聚合
service_health       服务健康状态
provider_health      中转站健康状态
alert_events         告警事件
```

优点：

- 页面加载更快。
- 数据库不会无限增长。
- 压测、故障、趋势分析都能保留。

SQLite 配置建议：

```sql
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA busy_timeout=5000;
```

---

### 4.3 服务健康监控

新增服务健康矩阵：

| 服务 | 检查方式 | 异常判断 |
|---|---|---|
| System Monitor | HTTP `/api/health` | 非 200 |
| Hermes Dashboard | HTTP `:9100` | 非 200 / 超时 |
| Obsidian Knowledge Web | HTTP `:9200` | 非 200 / 超时 |
| shared-memory | HTTP `:9400` | 非 200 / 超时 |
| OpenClaw | HTTP 本地端口或进程 | 无响应 / 进程不存在 |
| QQBot | 进程 / 日志 / websocket 状态 | 断连 / 报错 |
| Nginx | systemd + 端口 80/443 | inactive / 端口不监听 |

前端展示建议：

```text
AI 服务状态
├─ Hermes Dashboard: 正常
├─ OpenClaw: 正常
├─ QQBot: 正常
├─ shared-memory: 正常
├─ Obsidian Web: 正常
└─ Nginx: 正常
```

---

### 4.4 中转站健康监控

新增 provider health check。

检查项：

- `/v1/models` 是否可用。
- 低 token 测试请求是否成功。
- HTTP 状态码：401、429、500、502、503。
- 响应延迟。
- 连续失败次数。
- 最近一次成功时间。

注意：

- 不要高并发压测中转站。
- 默认只做低频健康检查。
- 不自动切换中转，除非用户明确授权。
- API key 不写入前端，不暴露日志。

---

### 4.5 告警模块

新增 alert manager，负责异常检测和 QQ 通知。

规则示例：

| 规则 | 级别 | 动作 |
|---|---|---|
| 磁盘 > 90% | warning | QQ 提醒 |
| 内存 > 90% 且持续 5 分钟 | warning | QQ 提醒 |
| OpenClaw 无响应 | critical | QQ 提醒 + 生成诊断 |
| Hermes Dashboard 无响应 | critical | QQ 提醒 |
| 中转连续失败 5 次 | warning | QQ 提醒 |
| Nginx inactive | critical | QQ 提醒 |

告警机制：

```text
触发 → 去重 → 冷却 → 通知 → 恢复通知 → 归档
```

冷却建议：

- warning：30 分钟内同类只发一次。
- critical：10 分钟内同类只发一次。
- recovery：恢复时发一次。

---

### 4.6 Agent 查询 API

为 Hermes / OpenClaw 增加专用接口：

```text
GET /api/agent/summary
GET /api/agent/services
GET /api/agent/providers
GET /api/agent/alerts/recent
GET /api/agent/diagnose
```

`/api/agent/summary` 示例：

```json
{
  "overall": "healthy",
  "cpu_percent": 12.3,
  "memory_percent": 48.1,
  "disk_percent": 41.7,
  "services": {
    "hermes_dashboard": "ok",
    "openclaw": "ok",
    "qqbot": "ok",
    "shared_memory": "ok"
  },
  "providers": {
    "sub_zmjjkkk": {
      "status": "ok",
      "latency_ms": 820,
      "last_error": null
    }
  },
  "recent_alerts": []
}
```

这样用户问“服务器现在正常吗”，Hermes / OpenClaw 可以直接读取结构化状态。

---

### 4.7 Dashboard 前端优化

页面建议重组为：

```text
顶部：总体健康状态 + 最后更新时间

第一屏：
- CPU
- 内存
- 磁盘
- 网络
- 当前告警数

第二屏：
- AI 服务健康矩阵
- 中转站健康
- Docker / systemd 状态

第三屏：
- 历史曲线
- 网络累计流量
- 资源峰值

第四屏：
- 最近错误
- 告警历史
- 诊断报告入口
```

移动端要求：

- 卡片自动换行。
- 操作按钮明确显示成功/失败。
- 图表不要挤压文字。
- 重要状态优先显示。

---

## 5. 分阶段实施

### 阶段一：低风险增强

目标：不大改架构，先补齐最有价值能力。

任务：

1. 增加服务健康检查 API。
2. 增加 Hermes / OpenClaw / QQBot / shared-memory 状态卡片。
3. 增加中转站低频健康检查。
4. 增加 QQ 告警的基础规则。
5. 增加 `/api/agent/summary`。

验收：

- 页面能看到所有关键服务状态。
- Hermes 能读取 `/api/agent/summary`。
- 模拟服务异常时能产生告警记录。
- 不暴露 API key。

---

### 阶段二：架构解耦

目标：拆出 collector 和 alert manager。

任务：

1. collector 独立循环采集。
2. API 只负责读取和输出。
3. alert manager 独立读取指标并判断告警。
4. SQLite 增加分层表。
5. 增加数据保留策略。

验收：

- 重启 API 不丢采集任务。
- 历史数据正常聚合。
- 告警不会重复刷屏。
- 7 天图表仍能快速加载。

---

### 阶段三：AI 运维中枢

目标：让 monitor 成为 Hermes / OpenClaw 的状态中心。

任务：

1. 增加诊断接口 `/api/agent/diagnose`。
2. 异常时生成 Markdown 诊断报告到 Obsidian。
3. 支持压测期间自动记录资源曲线。
4. 支持一键生成“服务器健康日报”。
5. Dashboard 增加报告入口。

验收：

- 用户问“服务器健康吗”，Agent 能基于 monitor 数据回答。
- 出现故障时能自动给出简短诊断。
- 压测后能生成资源变化摘要。
- 报告可在 Obsidian 中查看。

---

## 6. 安全边界

默认只读优先：

- 默认不自动重启服务。
- 默认不自动删除日志。
- 默认不自动清理 Docker。
- 默认不自动切换中转站。
- 默认不暴露 API key、环境变量、敏感日志。

可以保留人工确认动作：

```text
发现异常 → 生成候选修复方案 → 用户确认 → 执行修复
```

---

## 7. 推荐优先级

最建议先做这 5 个：

1. **服务健康监控**：Hermes、OpenClaw、QQBot、shared-memory。
2. **中转站健康监控**：低频、低 token、带错误码记录。
3. **QQ 告警**：带冷却和恢复通知。
4. **Agent 查询 API**：`/api/agent/summary`。
5. **历史数据分层**：避免 SQLite 长期膨胀。

---

## 8. 最终形态

最终 monitor 不只是资源看板，而是：

```text
服务器状态中心
+ AI 服务健康中心
+ 中转站可用性中心
+ QQ 告警中心
+ Hermes/OpenClaw 可查询的数据源
+ Obsidian 运维报告生成器
```

这套优化完成后，Hermes 和 OpenClaw 会更容易判断服务器状态，也更容易在故障时给出准确、可执行的诊断。
