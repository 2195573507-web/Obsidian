---
title: Atlas Hermes OpenClaw Shared Memory 专属架构
type: wiki
category: server
updated: 2026-06-30
tags: [atlas, hermes, openclaw, shared-memory]
source: hermes-multi-agent-broad-expansion-wave3
---

# Atlas Hermes OpenClaw Shared Memory 专属架构

> 关联：[[server/服务器架构 Wiki|服务器架构 Wiki]]、[[shared-memory/README|共享记忆说明]]、[[agents/hermes/MEMORY|Hermes 工作记忆]]

## 1. 当前定位

本服务器承载 Atlas Web 入口、Hermes legacy/support、OpenClaw gateway 状态、shared-agent-memory、Obsidian vault。它们共同形成“Agent 操作层 + 知识层 + 记忆层 + Web 管理层”。

## 2. Atlas 入口

monitor/agent/notes/files/daily/server/collab 通过 Nginx HTTPS 反代到本机 19000-19700 系列端口。Atlas 是当前主要 Web 入口，不要和 legacy dashboard 混淆。

## 3. Hermes 与 OpenClaw

Hermes provider 与 OpenClaw provider 独立维护；不要自动互相同步。Hermes 适合 cron、多工具编排；OpenClaw 适合多通道和浏览器自动化。

## 4. Shared Memory

shared-agent-memory:9400 是共享事实服务；Obsidian 是人类可读知识中枢；Hermes MEMORY.md 只保留当前工作记忆。

## 5. 风险

Agent 管理入口、server 控制入口、memory 原始数据、files 文件入口都属于高风险边界，需要鉴权、审计和确认机制。
