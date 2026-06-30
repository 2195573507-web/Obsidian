---
title: 权限系统 RBAC 与 ABAC
type: wiki
category: software
updated: 2026-06-30
tags: [rbac, abac, security, authorization]
source: hermes-multi-agent-deep-expansion-wave2
---

# 权限系统 RBAC 与 ABAC

> 关联：[[软件工程/API 设计规范与错误码|API 设计规范与错误码]]、[[AI/Agent 安全攻防与权限设计|Agent 安全攻防与权限设计]]

## 1. 权限模型

认证回答“你是谁”，授权回答“你能做什么”。权限必须在后端验证，前端隐藏按钮只是体验优化。

## 2. RBAC

用户 → 角色 → 权限。适合后台系统。示例：admin、operator、viewer。权限点如 `project.read`、`project.delete`。

## 3. ABAC

根据属性决策：用户部门、资源 owner、时间、IP、环境、数据等级。适合复杂数据范围。

## 4. 数据权限

常见规则：本人数据、部门数据、全局数据、项目成员数据。列表查询必须加数据范围过滤，详情接口也必须校验。

## 5. 审计

高风险操作记录：用户、动作、资源、前后值、IP、时间、request_id。

## 6. 常见漏洞

只在前端控制、IDOR 越权、列表过滤但详情不校验、角色硬编码、超级管理员泛滥、审计缺失。

## 7. 权限检查位置

权限必须在后端执行。前端隐藏按钮只改善体验，不能作为安全边界。列表、详情、更新、删除都要校验数据范围。

## 8. RBAC 表结构示例

```sql
users(id, name)
roles(id, code, name)
permissions(id, code, name)
user_roles(user_id, role_id)
role_permissions(role_id, permission_id)
```

## 9. ABAC 示例

规则：用户只能访问自己部门的数据，管理员可访问全部。

```python
def can_read_project(user, project):
    if user.has_role('admin'):
        return True
    return project.department_id == user.department_id
```

## 10. 审计

高风险操作记录 before/after、操作者、资源、IP、request_id。没有审计的后台不适合做高风险操作。
