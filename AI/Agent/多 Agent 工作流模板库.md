---
title: 多 Agent 工作流模板库
type: wiki
category: ai-agent
updated: 2026-06-30
tags: [multi-agent, workflow, templates]
source: hermes-multi-agent-broad-expansion-wave3
---

# 多 Agent 工作流模板库

> 关联：[[AI/多 Agent 协作、安全评测与生产运维|多 Agent 协作、安全评测与生产运维]]

## 1. 研究报告模板

```text
Supervisor：定义主题、范围、输出结构
Agent A：官方文档与基础概念
Agent B：开源实现与案例
Agent C：风险、反例、最佳实践
Supervisor：去重、合并、生成最终报告
```

## 2. 代码审查模板

- Reviewer A：安全；
- Reviewer B：可维护性；
- Reviewer C：测试与边界；
- Supervisor：按严重级别合并。

## 3. 运维排障模板

- Agent A：系统资源；
- Agent B：服务/容器；
- Agent C：网络/入口；
- Supervisor：判断 P0/P1/P2 并生成只读结论。

## 4. 知识库扩写模板

- Agent A：AI/理论；
- Agent B：运维/SOP；
- Agent C：软件/产品/数据；
- Parent：写入 Obsidian、更新 MOC、验证。

## 5. 约束

子 Agent 默认只产草稿，不直接写共享文件；有副作用的外部操作要返回可验证证据；最终由父 Agent 统一写入。
