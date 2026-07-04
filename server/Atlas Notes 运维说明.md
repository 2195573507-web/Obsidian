---
title: "Atlas Notes 运维说明"
type: wiki
category: "server"
updated: 2026-07-05
managed_by: Hermes
tags: [atlas, notes]
---

# Atlas Notes 运维说明

> 全量扩写知识页。上级入口：[[server/Atlas 服务与知识库集成架构|Atlas 服务与知识库集成架构]]、[[MOC|知识库导航]]。

Atlas Notes 负责展示 Obsidian vault。健康检查至少包括本地 19200 页面、recall 19410 health、vault 文件数和最近更新时间。

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

- [[server/Atlas 服务与知识库集成架构|Atlas 服务与知识库集成架构]]
- [[MOC|知识库导航]]
