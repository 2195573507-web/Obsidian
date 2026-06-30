---
title: Linux 命令速查表
type: wiki
category: ops
updated: 2026-06-30
tags: [linux, cheatsheet]
source: hermes-multi-agent-broad-expansion-wave4
---

# Linux 命令速查表

> 关联：[[运维/Linux 系统管理与故障排查 SOP|Linux 系统管理与故障排查 SOP]]

## 系统
`date`、`uptime`、`hostnamectl`、`uname -a`、`cat /etc/os-release`

## 资源
`free -h`、`df -h`、`df -ih`、`ps aux --sort=-%cpu | head`、`ps aux --sort=-%mem | head`

## 网络
`ip a`、`ip r`、`ss -tulpn`、`curl -Iv URL`、`dig domain`

## 日志
`journalctl -u SERVICE -n 200 --no-pager`、`journalctl -p err --since '24 hours ago'`

## 文件
`du -xhd1 /path | sort -h`、`find /path -type f -size +500M`

## 安全提醒
只读命令优先；`rm`、`kill`、`systemctl restart`、`ufw` 变更需确认。
