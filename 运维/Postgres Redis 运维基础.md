---
title: Postgres Redis 运维基础
type: wiki
category: ops
updated: 2026-06-30
tags: [postgres, redis, database]
source: hermes-multi-agent-broad-expansion-wave3
---

# Postgres Redis 运维基础

> 关联：[[软件工程/数据库设计与性能优化|数据库设计与性能优化]]、[[运维/Docker 服务部署与容器运维|Docker 服务部署与容器运维]]

## 1. Postgres

检查：连接数、磁盘、慢查询、备份、WAL、锁。备份用 `pg_dump -Fc`，恢复必须演练。Postgres 不应暴露公网。

## 2. Redis

检查：内存、key 数、连接数、持久化、淘汰策略。Redis 不应暴露公网，不要无确认执行 FLUSHALL。

## 3. Docker 场景

确认 volume、健康检查、日志、资源限制、端口是否 internal。数据库容器重启需确认影响。
