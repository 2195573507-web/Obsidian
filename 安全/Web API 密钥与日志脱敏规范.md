---
title: Web API 密钥与日志脱敏规范
type: wiki
category: security
updated: 2026-06-30
tags: [web-security, api-security, secrets, redaction]
source: hermes-multi-agent-broad-expansion-wave3
---

# Web API 密钥与日志脱敏规范

> 关联：[[运维/安全基线与入侵排查|安全基线与入侵排查]]、[[软件工程/API 设计规范与错误码|API 设计规范与错误码]]

## 1. API 安全

认证、授权、输入校验、速率限制、CSRF/XSS/SQL 注入防护、文件上传限制、错误信息不泄露内部细节。

## 2. 密钥管理

密钥不进 Git、不进日志、不进截图。最小权限、定期轮换、按环境隔离、泄露后立即轮换。

## 3. 日志脱敏

脱敏字段：password、token、secret、api_key、authorization、cookie、phone、email、id_card。

## 4. 审计

记录谁在何时访问/修改了什么资源。高风险操作必须有 request_id 和操作者。

## 5. 脱敏策略

| 类型 | 示例 | 脱敏 |
|---|---|---|
| token | sk-abcdef123456 | sk-***3456 |
| 邮箱 | user@example.com | u***@example.com |
| 手机 | 13812345678 | 138****5678 |
| 身份证 | 110101... | 110101************ |
| Cookie | session=... | 不记录 |

## 6. 密钥轮换流程

1. 确认泄露范围；2. 创建新密钥；3. 更新服务配置；4. reload/restart 相关服务；5. 验证功能；6. 撤销旧密钥；7. 搜索日志和仓库残留；8. 记录事故。

## 7. API 安全清单

- [ ] 认证；
- [ ] 授权；
- [ ] 输入校验；
- [ ] 输出脱敏；
- [ ] 限流；
- [ ] 审计；
- [ ] 错误不泄露堆栈；
- [ ] 文件上传限制；
- [ ] CORS 白名单。
