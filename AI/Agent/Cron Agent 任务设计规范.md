---
title: "Cron Agent 任务设计规范"
type: wiki
category: "AI/Agent"
updated: 2026-07-05
managed_by: Hermes
tags: [cron, agent]
---

# Cron Agent 任务设计规范

> 全量扩写知识页。上级入口：[[AI/Agent/Agent 工程生产化手册|Agent 工程生产化手册]]、[[MOC|知识库导航]]。

Cron Agent 适合日报、巡检、索引重建、异常监测。prompt 必须自包含，写清北京时间/UTC、输出语言、数据路径、失败时行为和交付目的地。no-agent 脚本适合阈值告警，LLM cron 适合摘要、分析和报告。

## 标准执行模板

1. 明确目标、范围和不做事项。
2. 收集当前证据：文件、服务、日志、配置、健康接口或历史报告。
3. 给出最小可执行方案，并保留回滚路径。
4. 执行后验证：读回文件、运行测试、访问接口或检查 Git diff。
5. 将稳定经验沉淀到知识库，临时状态写入报告而不是长期记忆。

## 检查清单

- 是否有 frontmatter、更新时间和分类。
- 是否至少链接一个上级 MOC。
- 是否区分稳定事实与实时状态。
- 是否避免泄露密钥、token、隐私和内部未授权信息。

## 关联入口

- [[AI/Agent/Agent 工程生产化手册|Agent 工程生产化手册]]
- [[MOC|知识库导航]]
