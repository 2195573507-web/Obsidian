---
title: 技术决策 ADR 与架构评审
type: wiki
category: software
updated: 2026-06-30
tags: [adr, architecture-review, decision]
source: hermes-multi-agent-deep-expansion-wave2
---

# 技术决策 ADR 与架构评审

> 关联：[[软件工程/现代软件工程实战总纲|现代软件工程实战总纲]]

## 1. 为什么需要 ADR

很多架构问题不是“对错”，而是权衡。ADR 记录当时背景、选择、理由和后果，避免未来重复争论。

## 2. ADR 模板

```markdown
# ADR-001：选择 FastAPI + 原生前端

## 状态
Accepted

## 背景
团队小，服务以内部门户为主，需要快速交付。

## 决策
使用 FastAPI 提供 API 和静态页面，前端使用原生 HTML/CSS/JS。

## 备选方案
- React SPA
- Vue
- Django Admin

## 理由
部署简单、依赖少、符合现有服务栈。

## 后果
正面：开发快、运维简单。
负面：大型复杂交互需要自建组件规范。
```

## 3. 架构评审清单

- 需求和非目标是否明确；
- 数据模型是否稳定；
- 权限和安全是否前置；
- 性能瓶颈是否可预期；
- 可观测性是否设计；
- 部署、回滚、迁移是否明确；
- 是否有测试策略；
- 是否过度设计。

## 4. 决策生命周期

Proposed → Accepted → Deprecated → Superseded。旧 ADR 不删除，新 ADR 标注替代关系。
