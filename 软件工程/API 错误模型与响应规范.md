---
title: "API 错误模型与响应规范"
type: wiki
category: "软件工程"
updated: 2026-07-05
managed_by: Hermes
tags: [api, errors]
---

# API 错误模型与响应规范

> 全量扩写知识页。上级入口：[[软件工程/软件工程 MOC|软件工程 MOC]]、[[MOC|知识库导航]]。

API 错误响应应包含 code、message、detail、request_id，前端可展示 message，日志记录 detail。不要把 Python traceback、密钥、内部路径直接返回给浏览器。

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

- [[软件工程/软件工程 MOC|软件工程 MOC]]
- [[MOC|知识库导航]]
