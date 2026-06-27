---
title: Hermes Agent Index
agent: hermes
updated: 2026-06-27
tags: [hermes, agent, index]
---

# Hermes Agent Index

Hermes 当前使用独立工作记忆 + shared persona/config：

- [[agents/hermes/MEMORY|Hermes Working Memory]]：Hermes 当前工作记忆，保留当前有效偏好/决策/状态/待办。
- [[agents/shared/SOUL|Shared SOUL]]：共享人格与语气。
- [[agents/shared/USER|Shared USER]]：共享用户画像和当前环境事实。
- [[agents/shared/IDENTITY|Shared IDENTITY]]：共享身份边界。
- [[agents/shared/TOOLS|Shared TOOLS]]：共享工具与记忆规则。
- [[plans/hermes-openclaw-memory-strategy|Hermes/OpenClaw 记忆策略]]：长期记忆分层、写入、召回和治理规则。

注意：OpenClaw 侧记忆暂不改；Hermes 侧若遇到服务/端口/Provider 状态，必须 live check，不把旧状态当当前事实。
