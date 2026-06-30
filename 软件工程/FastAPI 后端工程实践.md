---
title: FastAPI 后端工程实践
type: deep-wiki
category: software
updated: 2026-06-30
tags: [fastapi, python, backend]
source: hermes-deep-expansion
---

# FastAPI 后端工程实践

> 关联：[[软件工程/API 设计规范与错误码|API 设计规范与错误码]]、[[软件工程/数据库设计与性能优化|数据库设计与性能优化]]

## 1. 适用场景

FastAPI 适合 API 服务、管理后台、数据服务、AI 工具后端、轻量单页应用后端。优势是类型标注、Pydantic、OpenAPI、依赖注入和异步 IO。

## 2. 推荐结构

```text
app/
  main.py
  core/config.py logging.py security.py errors.py
  db/session.py
  models/
  schemas/
  repositories/
  services/
  api/routes/
  api/deps.py
  tests/
```

小项目可简化，大项目按领域 modules 拆分。

## 3. 分层

API 层只处理 HTTP；Schema 是契约；Service 写业务；Repository 封装查询；Core 放配置日志安全。不要在路由里拼 SQL 和复杂业务规则。

## 4. API 规范

URL 面向资源，错误响应统一，列表分页，写操作考虑幂等。敏感字段绝不进 response model。

## 5. 依赖注入

Depends 用于 session、当前用户、权限、service、外部 client。测试时 override dependency，避免全局状态。

## 6. 事务

事务边界通常在 Service。不要在事务中调用慢外部 API。并发写要用唯一约束、锁、幂等键或状态机。

## 7. 异步

async 适合 DB/HTTP/Redis 等 IO；CPU 密集任务用后台队列或进程池。同步阻塞放进 async route 会拖垮事件循环。

## 8. 生产

需要 Nginx、systemd/Docker、健康检查、日志、迁移、监控、配置校验和回滚。`/health` 应至少检查应用，关键服务可检查 DB/Redis。

## 9. 安全

后端必须做权限检查；CORS 不要过宽；上传限制大小和类型；错误不暴露堆栈；日志脱敏。
