---
title: 威胁建模 STRIDE 手册
type: wiki
category: security
updated: 2026-06-30
tags: [stride, threat-modeling]
source: hermes-multi-agent-broad-expansion-wave4
---

# 威胁建模 STRIDE 手册

> 关联：[[安全/安全 MOC|安全 MOC]]

| STRIDE | 含义 | 例子 |
|---|---|---|
| S | Spoofing 冒充 | 伪造用户身份 |
| T | Tampering 篡改 | 修改请求参数 |
| R | Repudiation 抵赖 | 无审计日志 |
| I | Information Disclosure | 日志泄露 token |
| D | Denial of Service | 大量请求拖垮服务 |
| E | Elevation of Privilege | 普通用户越权管理员 |

流程：画数据流图 → 标出信任边界 → 按 STRIDE 枚举威胁 → 设计缓解 → 写入验收标准。
