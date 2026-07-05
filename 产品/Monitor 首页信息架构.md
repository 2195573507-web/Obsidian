---
title: "Monitor 首页信息架构"
type: wiki
category: "产品"
updated: 2026-07-05
managed_by: Hermes
tags: [monitor, dashboard, product]
---

# Monitor 首页信息架构

> 上级入口：[[产品/产品 MOC|产品 MOC]]、[[dashboards/system-monitor-dashboard-analysis|System Monitor 分析]]。

## 核心问题

Monitor 首页不应堆满指标，而要回答四个问题：服务是否可用、慢/错在哪层、哪个 provider/model 出问题、下一步点哪里排查。

## 推荐区块

1. 全局健康：RED/USE/Golden Signals 总览。
2. LLM Gateway：provider/model health matrix。
3. Streaming：first token、chunk gap、active streams、client cancel。
4. Cron/Reports：last success、runtime、failure reason、artifact path。
5. 变更与事故：最近部署、重启、告警、补跑。

## 交互原则

- 首页只放判断，详情页放原始曲线。
- 红黄绿状态必须可解释。
- 每个异常卡片给下一步操作。
- 高基数字段不放在聚合卡片。
- 报告产物路径可点击。

## 验收清单

- 30 秒内能判断是否需要人工介入。
- 能定位 provider/model 问题。
- 能看到 cron/report 最近成功状态。
- 能区分 API 和 streaming 问题。
- 能追踪到 run_id 或报告路径。

## 关联入口

- [[运维/Provider 健康矩阵与模型路由巡检|Provider 健康矩阵与模型路由巡检]]
- [[运维/流式 API 可观测性指标|流式 API 可观测性指标]]
- [[数据/指标标签基数控制清单|指标标签基数控制清单]]
