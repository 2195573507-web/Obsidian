---
title: API 设计规范与错误码
type: wiki
category: software
updated: 2026-06-30
tags: [api, error-code, backend]
source: hermes-multi-agent-deep-expansion-wave2
---

# API 设计规范与错误码

> 关联：[[软件工程/FastAPI 后端工程实践|FastAPI 后端工程实践]]、[[软件工程/现代软件工程实战总纲|现代软件工程实战总纲]]

## 1. API 目标

API 是前后端和系统间契约。好 API 要稳定、一致、可测试、可文档化、错误可理解。

## 2. URL 与方法

资源名用复数名词：`/api/users/{id}`。GET 查询，POST 创建，PATCH 部分更新，DELETE 删除。动作类可用 `/api/orders/{id}/cancel`。

## 3. 响应格式

成功列表：`{items,total,page,page_size}`。错误统一：

```json
{"error":{"code":"PERMISSION_DENIED","message":"权限不足","details":{},"request_id":"req_x"}}
```

## 4. 错误码

错误码稳定、英文大写、可机器识别。分类：AUTH_REQUIRED、PERMISSION_DENIED、RESOURCE_NOT_FOUND、VALIDATION_FAILED、CONFLICT、RATE_LIMITED、INTERNAL_ERROR。

## 5. 幂等

创建订单、支付、webhook、导入任务必须考虑 Idempotency-Key、唯一约束、状态机、去重表。

## 6. 版本与兼容

新增字段兼容；删除字段分阶段；语义变化要升版本；文档和 OpenAPI 同步更新。

## 7. 分页、过滤与排序

列表接口必须分页。后台管理可用 page/page_size，时间线和大数据列表优先 cursor。

```text
GET /api/orders?page=1&page_size=20&status=paid&sort=-created_at
```

过滤字段要白名单，避免用户传任意字段拼 SQL。排序也要白名单。

## 8. 错误码字典模板

| 错误码 | HTTP | 含义 | 用户提示 | 开发排查 |
|---|---:|---|---|---|
| AUTH_REQUIRED | 401 | 未登录 | 请先登录 | 检查 token |
| PERMISSION_DENIED | 403 | 无权限 | 权限不足 | 检查 RBAC/ABAC |
| RESOURCE_NOT_FOUND | 404 | 资源不存在 | 数据不存在 | 检查 ID 和数据范围 |
| VALIDATION_FAILED | 422 | 参数错误 | 输入不合法 | 看 details |
| CONFLICT | 409 | 状态冲突 | 请刷新后重试 | 幂等/状态机 |

## 9. API 兼容性

新增字段通常兼容；删除字段、改类型、改语义不兼容。移动端/第三方调用者存在缓存和旧版本，API 变更应给迁移窗口。
