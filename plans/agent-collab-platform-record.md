---
title: "Agent 协作平台实施记录"
tags: ['plan', 'project']
updated: 2026-06-24
---

# Agent 协作平台实施记录

> 服务名：Agent Collab Web  
> 端口：`7000`  
> 路径：`/opt/agent-collab-web/`  
> 状态：第一版已上线  
> 时间：2026-06-19

## 目标

在同一台服务器上，让 OpenClaw 和 Hermes 能围绕同一个任务进行协作：

- 用户创建任务房间
- 指定主 Agent 和辅助 Agent
- 在一个时间线里记录用户、Hermes、OpenClaw、系统消息
- 后续接入自动分派，让一个 Agent 可以请求另一个 Agent 辅助

## 当前第一版能力

已完成：

- 独立 Web 服务：`http://38.55.146.137:7000`
- SQLite 数据库：`/opt/agent-collab-web/collab.db`
- systemd 服务：`agent-collab-web.service`
- UFW 已开放：`7000/tcp`
- 任务房间：创建、列表、打开
- Agent 角色：Hermes / OpenClaw
- 消息时间线：记录发送者、目标、内容、时间
- 状态管理：active / paused / done
- 第一版采用人工转发，不自动让两个 Agent 无限互聊

## 当前服务文件

```text
/opt/agent-collab-web/main.py
/opt/agent-collab-web/static/index.html
/opt/agent-collab-web/collab.db
/etc/systemd/system/agent-collab-web.service
```

## 访问地址

```text
http://38.55.146.137:7000
```

未来可以挂到：

```text
https://server.zmjjkkk.fun/agents
```

## 当前限制

第一版只做“协作记录平台”，暂不直接调用 Hermes/OpenClaw API。

原因：

- 避免两个 Agent 无限互相调用
- 先保证任务房间、时间线、角色分工稳定
- 后续再接自动分派能力

## 下一阶段建议

### Phase 2：接入自动分派

目标：在平台里点击按钮即可把消息发送给指定 Agent。

需要研究：

- Hermes 是否有可用的本地会话发送 API
- OpenClaw 是否可以通过 sessions_send 或 Gateway API 收消息
- 两边返回结果如何回写到任务房间

### Phase 3：协作策略

加入固定协作模板：

- Hermes 主做，OpenClaw 审查
- OpenClaw 主规划，Hermes 执行
- 双 Agent 轮询，但最多 3 轮
- 用户确认后再执行危险操作

### Phase 4：归档到 Obsidian

每个任务完成后生成 Markdown：

```text
/root/obsidian-vault/tasks/<task-id>.md
```

内容包括：

- 任务目标
- Agent 分工
- 全部对话记录
- 最终结论
- 后续 TODO

## 安全/控制规则

必须避免无限对话：

- 每个任务必须有主 Agent
- 辅助 Agent 默认只建议/验证
- 自动互相调用最多 3~5 轮
- 涉及文件修改、服务重启、配置变更时仍由用户确认
- 全部跨 Agent 消息必须落库

## 当前结论

第一版 Agent 协作平台已经可用，适合作为 Hermes 和 OpenClaw 的共享任务房间。

下一步重点不是 UI，而是接入两个 Agent 的实际消息发送链路。
