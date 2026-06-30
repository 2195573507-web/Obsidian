---
title: SSH 加固实战手册
type: wiki
category: security
updated: 2026-06-30
tags: [ssh, hardening, ops]
source: hermes-multi-agent-broad-expansion-wave3
---

# SSH 加固实战手册

> 关联：[[运维/安全基线与入侵排查|安全基线与入侵排查]]

## 1. 基线

密钥登录、禁用密码、限制 root、fail2ban、审计 authorized_keys、必要时限制管理 IP。

## 2. 检查

```bash
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries|allowusers'
last -a | head -50
journalctl -u ssh --since '24 hours ago' --no-pager | grep -Ei 'Failed|Accepted|Invalid'
```

## 3. 变更前

保留当前 session，新开 session 验证，确认云控制台可用，备份 sshd_config。重启 SSH 前确认。

## 4. fail2ban

适合降低扫描噪音。策略应避免误封自己，先观察日志再启用。
