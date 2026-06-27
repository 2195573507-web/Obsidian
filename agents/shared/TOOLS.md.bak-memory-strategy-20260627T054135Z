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
- 共享记忆每月执行一次衰减审查：`importance < 5` 且长时间未访问的条目进入 noise 候选
- 共享记忆每季度执行一次审计：重复检测、矛盾检测、合并建议报告
- Hermes 侧的工作记忆只保留当前有效内容，历史性内容放入共享记忆或日记归档

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

## Obsidian Vault 写入规范

写入 `/root/obsidian-vault/` 下的文件时，必须遵循以下规范：

### 1. 必须带 frontmatter

每个新文件开头必须有 YAML frontmatter：

```yaml
---
title: 文件标题
tags: [标签1, 标签2]
updated: 2026-06-24
---
```

### 2. 至少包含 1 个 wikilink

文件内容中至少用 `[[]]` 链接到 1 个相关笔记，例如：
- `[[MOC|知识库导航]]`
- `[[服务器架构]]`
- `[[agents/shared/SOUL|共用人格]]`

### 3. 文件命名

- 使用中文或英文短横线命名，避免空格
- 日期相关：`2026-06-24-主题.md`
- 报告类：`主题-report.md`
- 实体类：`服务器架构.md`

### 4. 目录归属

- 记忆治理报告 → `记忆治理/`
- 技能治理报告 → `技能治理/`
- 服务器审计 → `服务器审计/`
- 每日资讯 → `资讯更新/{分类}/`
- 全球调研 → `全球咨询调研/`
- 项目计划 → `plans/`

### 5. 更新 MOC

如果创建了重要的新文件或新目录，更新 `MOC.md` 添加链接。
