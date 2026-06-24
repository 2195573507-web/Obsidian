---
title: "Shared Memory Integration"
tags: ['memory', 'shared']
updated: 2026-06-24
---

# Shared Memory Integration

Hermes 和 OpenClaw 共享长期记忆已经接入本机 `shared-agent-memory` 服务。

## Service

- API: `http://127.0.0.1:9400`
- Health: `http://127.0.0.1:9400/health`
- Database: `/opt/shared-agent-memory/data/memory.sqlite`
- Markdown export: `/root/obsidian-vault/shared-memory/exports/shared-memory.md`

## CLI

```bash
shared-memory-client health
shared-memory-client add '记忆内容' --agent hermes --scope shared --kind fact --tag example
shared-memory-client search '关键词'
shared-memory-client context '当前任务关键词'
shared-memory-client export
```

## Mounted Paths

- Hermes: `/root/.hermes/SHARED_MEMORY.md`
- Hermes memory dir: `/root/.hermes/memories/SHARED_MEMORY.md`
- OpenClaw workspace: `/root/.openclaw/workspace/SHARED_MEMORY.md`
- OpenClaw memory dir: `/root/.openclaw/workspace/memory/shared-memory.md`

## Usage Rule

- 重要事实、偏好、服务器结构、项目状态、跨 Agent 决策写入共享记忆。
- 临时任务进度、一次性日志、密钥/token 不写入共享记忆。
- 回答前可用 `shared-memory-client context '<task>'` 拉取相关上下文。
