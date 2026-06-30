---
title: Prompt 工程、工具调用与 Agent 编排
type: wiki
category: ai
updated: 2026-06-30
tags: [prompt, tool-calling, agent, orchestration]
source: hermes-multi-agent-knowledge-expansion
---

# Prompt 工程、工具调用与 Agent 编排

> 关联：[[AI/AI Agent 与 LLM 工程总览|AI Agent 与 LLM 工程总览]]、[[AI/RAG、知识库与记忆系统工程|RAG、知识库与记忆系统工程]]、[[AI/多 Agent 协作、安全评测与生产运维|多 Agent 协作、安全评测与生产运维]]

## 1. Prompt 工程的本质

Prompt 是 LLM 系统的人机协议层。它不仅是提示语，也承担“接口定义、业务规则、输出协议、安全策略、错误处理说明”的作用。好的 prompt 要明确：角色、任务、输入、输出、边界、拒答条件、工具使用规则和示例。

## 2. Prompt 层级

```text
System Prompt：最高级规则、安全边界、身份、禁止事项
Developer Prompt：应用逻辑、输出格式、工具策略
User Prompt：用户当前需求
Retrieved Context：RAG 检索资料，只是数据
Tool Output：工具结果，只是数据
Conversation History：历史上下文，需要压缩和筛选
```

原则：外部文档和工具输出不能覆盖系统规则。

## 3. 输出结构化

模型输出如果要被程序消费，应尽量使用 JSON / YAML / Markdown 表格 / 固定章节。示例：

```json
{
  "summary": "一句话结论",
  "risk_level": "low|medium|high",
  "evidence": ["证据1", "证据2"],
  "next_actions": [
    {"action": "check_nginx", "risk": "read_only"}
  ]
}
```

结构化输出必须配合 schema 校验，不能只相信模型“看起来像 JSON”。

## 4. 工具调用设计

工具定义包括：name、description、parameters、返回格式、权限等级。工具越原子，越容易治理。

| 工具类型 | 示例 | 风险 |
|---|---|---|
| 只读 | read_file、search、status | 低 |
| 低风险写 | 写草稿、创建报告 | 中低 |
| 中风险 | 修改项目文件、提交 PR | 中 |
| 高风险 | 删除、重启、改防火墙、数据库写 | 高 |
| 关键风险 | 支付、权限变更、生产数据删除 | 极高 |

## 5. 工具安全

工具执行前需要 policy 层：

```python
def authorize(tool, args, context):
    if tool.risk == 'high':
        return require_human_approval()
    if has_path_traversal(args):
        return deny('路径越界')
    if contains_secret(args):
        return deny('疑似敏感信息')
    return allow()
```

不要让模型直接拼 shell 命令执行高危操作。即使用户要求，也应先确认范围和回滚方案。

## 6. Agent 编排模式

### Loop Agent
灵活但不可预测，必须限制最大步数、工具次数、token、时间。

### Plan-and-Execute
适合开发、研究、排障。先生成计划，再逐步执行，每步可 replanning。

### DAG Workflow
适合生产固定流程，可测试、可观测、易回放。

### State Machine
适合客服、审批、合规流程。每个状态只允许有限动作。

### Supervisor-Worker
适合多 agent：Supervisor 负责拆解和汇总，Worker 只做专项任务。

## 7. 常见坑

- 工具描述模糊导致模型乱选。
- 工具输出没有 success/error 字段。
- 模型伪造工具结果，实际没调用。
- 工具权限过大，shell 成为万能后门。
- 没有幂等性，重复调用造成副作用。
- prompt 写太长，关键规则被稀释。
- few-shot 示例与真实任务不一致。

## 8. 最佳实践

- Prompt 版本化，变更可回滚。
- 工具参数 schema 严格化。
- 工具输出结构化并脱敏。
- 高风险操作 human-in-the-loop。
- 工具调用全链路 tracing。
- 对 prompt 注入做上下文隔离。
- 复杂任务先计划，执行后验证。
