---
title: "工具使用偏好"
tags: ['agent', 'config']
updated: 2026-06-24
---

# 工具使用偏好

## 通用原则

- 优先用工具验证，不要凭记忆猜
- 给命令时给完整可执行的一行，不要拆开让人拼
- 修改配置前先查现状（`config.get`、`cat`、`grep`）
- 破坏性命令（rm、systemctl stop、kill）前先确认

## Shell / Exec

- 默认 shell：bash
- 长输出用 `| head -50` 或 `| tail -30` 截断
- 需要 sudo 时明确标注
- 后台任务用 systemd service，不要用 nohup 裸跑

## 文件操作

- 小改动用 `edit`（精确替换），不要用 `write` 覆盖整个文件
- 新建文件用 `write`
- 读取图片用 `read`（支持 jpg/png/gif/webp）

## 浏览器

- 优先用 OpenClaw browser 工具做自动化
- 需要用户已登录状态时用 `profile="user"`
- 默认用 isolated browser（无 profile）

## 搜索

- 技术问题：先搜本地 docs，再 web_search
- 代码搜索：用 `exec grep -r` 而非文件逐个读

## 定时任务

- Hermes 用内置 cron
- OpenClaw 用 Gateway cron 工具
- 创建时标注 BJT 和 UTC 双时区

## 记忆

- 短期会话记忆：自动维护
- 长期记忆：写入 MEMORY.md 或 memory/*.md
- 跨 agent 共享：通过 shared-agent-memory 服务（:9400）
- 不要手动删除记忆，让维护脚本处理

### 写入记忆时必须自带完整分类

写入 shared-agent-memory 时，必须显式提供以下字段，不要依赖默认值：

- `kind`：preference / fact / service_state / decision / success / failure / todo / temporary
- `importance`：1-100（关键决策 90+，常规事实 5-10，噪音 1-3）
- `sensitivity`：shared（双方可见）/ public / private
- `tags`：至少 2 个相关标签
- `subject`：一句话摘要

示例：
```json
{
  "kind": "decision",
  "importance": 95,
  "sensitivity": "shared",
  "tags": ["model", "routing", "gpt"],
  "subject": "所有请求走中转 GPT，不使用本地模型",
  "content": "..."
}
```

写入 MEMORY.md / memory/*.md 时，用 `[kind/scope]` 前缀标注类型和范围。

## 模型调用

- 默认走中转站 GPT（myproxy/gpt-5.5）
- 不使用千问/Qwen（2026-06-23 用户明确决定）
- 所有请求走中转 GPT

## 配置修改

- 用 `gateway config.patch` 做局部修改，不要 `config.apply` 全量替换
- 修改前用 `config.schema.lookup` 查字段含义
- 修改后用 `config.get` 验证生效
