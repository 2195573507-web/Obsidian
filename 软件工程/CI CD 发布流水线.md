---
title: CI CD 发布流水线
type: wiki
category: software
updated: 2026-06-30
tags: [ci-cd, deployment]
source: hermes-multi-agent-broad-expansion-wave3
---

# CI CD 发布流水线

> 关联：[[软件工程/测试与质量保障体系|测试与质量保障体系]]、[[运维/发布变更回滚与容量规划|发布变更、回滚与容量规划]]

## 1. CI

安装依赖、格式化检查、lint、类型检查、单元测试、集成测试、安全扫描、构建产物。

## 2. CD

部署 staging、冒烟测试、人工确认、生产部署、健康检查、监控观察、失败回滚。

## 3. 发布策略

小步发布、蓝绿/灰度、feature flag、数据库迁移分阶段、配置备份、回滚脚本预备。

## 4. 失败处理

自动停止流水线，保留日志和产物，通知负责人，回滚到上一个健康版本。
