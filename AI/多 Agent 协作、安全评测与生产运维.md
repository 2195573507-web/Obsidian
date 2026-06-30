---
title: 多 Agent 协作、安全评测与生产运维
type: wiki
category: ai
updated: 2026-06-30
tags: [multi-agent, security, evaluation, llmops]
source: hermes-multi-agent-knowledge-expansion
---

# 多 Agent 协作、安全评测与生产运维

> 关联：[[AI/AI Agent 与 LLM 工程总览|AI Agent 与 LLM 工程总览]]、[[AI/Prompt 工程、工具调用与 Agent 编排|Prompt 工程、工具调用与 Agent 编排]]、[[AI/RAG、知识库与记忆系统工程|RAG、知识库与记忆系统工程]]

## 1. 多 Agent 为什么有用

单 Agent 容易受上下文限制、角色冲突、思路单一、自评不可靠影响。多 Agent 适合复杂研究、代码审查、长文档生成、系统排障、产品方案、数据分析。

## 2. 协作模式

### Supervisor-Worker

```text
Supervisor
  ├── Research Worker
  ├── Coding Worker
  ├── Review Worker
  └── Writer Worker
```

Supervisor 拆任务、控范围、汇总、去重、裁决。Worker 只做专项任务，不擅自扩大范围。

### Critic 模式

Writer 输出初稿，Critic 查事实、逻辑、安全、遗漏，Revision 合并修改。

### Pipeline 模式

Research → Outline → Draft → Review → Final。适合报告和知识库。

### Debate 模式

不同 Agent 提出对立方案，裁判汇总。适合架构选型，但成本高。

## 3. 任务拆分原则

- 子任务边界清晰；
- 输入输出明确；
- 尽量无共享写状态；
- 可并行；
- 可验证；
- 汇总层统一术语和格式。

给 Worker 的任务包应包含：goal、scope、exclude、output_format、constraints、facts、risk boundary。

## 4. 多 Agent 常见坑

- 只是换角色名，没有真实任务差异；
- 没有最终裁决者；
- 多个 Agent 同时写同一文件；
- 输出重复严重；
- 成本乘法爆炸；
- 子 Agent 自称完成但没有可验证证据；
- 错误信息被多个 Agent 放大。

## 5. Agent 安全分层

```text
输入安全：注入检测、认证、限流
上下文安全：来源标注、权限过滤、敏感脱敏
模型安全：拒答策略、结构化输出
工具安全：参数校验、沙箱、审批
输出安全：PII 检测、事实校验
运维安全：日志、告警、回滚
```

## 6. 高风险工具治理

高风险包括：删除文件、修改系统服务、开放端口、数据库写入、发送外部消息、支付、权限变更。策略：dry-run、差异报告、人工确认、幂等 key、审计日志、回滚路径。

## 7. LLM / Agent 评测

评测维度：正确性、完整性、工具调用准确率、安全违规率、幻觉率、成本、延迟、人工介入率、用户满意度。

测试集应覆盖：正常问题、边界问题、恶意注入、缺失资料、工具失败、权限不足、多轮上下文、长文档。

## 8. LLMOps 指标

- 请求量；
- 成功率；
- 错误率；
- P95/P99 延迟；
- input/output token；
- 成本；
- 工具失败率；
- 检索命中率；
- schema 失败率；
- 安全策略触发次数。

## 9. 降级策略

- 主模型不可用 → 备用模型；
- RAG 不可用 → 明确说明无法访问知识库；
- 工具失败 → 给手动操作建议；
- 高风险不确定 → 转人工；
- 成本超限 → 暂停多 Agent 或切小模型。

## 10. 生产事故例子

| 事故 | 原因 | 防护 |
|---|---|---|
| 输出格式变化 | 模型升级 | schema 校验 + 回归测试 |
| Agent 循环 | 无最大步数 | step limit + 重复动作检测 |
| 工具误删 | 权限过大 | human approval + dry-run |
| RAG 答错 | 检索错 | 检索评测 + 引用校验 |
| 记忆污染 | 错误写入长期记忆 | 写入分类 + 审计 |
