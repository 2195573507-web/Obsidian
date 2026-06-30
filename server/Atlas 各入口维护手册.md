---
title: Atlas 各入口维护手册
type: wiki
category: server
updated: 2026-06-30
tags: [atlas, ops]
source: hermes-multi-agent-broad-expansion-wave4
---

# Atlas 各入口维护手册

> 关联：[[server/Atlas Hermes OpenClaw Shared Memory 专属架构|Atlas Hermes OpenClaw Shared Memory 专属架构]]

## 入口

monitor、agent、notes、files、daily、server、collab 分别对应 19000、19100、19200、19300、19500、19600、19700。

## 检查

```bash
systemctl status --no-pager atlas-observe.service
systemctl status --no-pager atlas-agents.service
systemctl status --no-pager atlas-notes.service
ss -tulpn | grep -E '19000|19100|19200|19300|19500|19600|19700'
curl -I https://monitor.zmjjkkk.fun
```

## 风险

server/agent/files/notes 入口涉及管理、Agent、文件、知识库，必须关注鉴权和日志。
