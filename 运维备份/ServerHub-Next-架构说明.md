---
title: "ServerHub Next 说明"
tags: ['ops', 'backup']
updated: 2026-06-24
---

# ServerHub Next 说明

## 目标
- 独立新建一个统一控制台，不复用现有任何 Web 的 UI、前端代码或后端实现。
- 功能覆盖当前已有 Web 能力：入口总览、系统监控、Agent、知识库、文件、共享记忆、每日总结、日志、备份、安全。
- 项目结构清晰，便于未来新增模块。

## 目录结构
- `/opt/serverhub-next/backend.py`：FastAPI 后端，负责聚合、状态探测、只读 API、页面路由。
- `/opt/serverhub-next/static/index.html`：全新单页前端。
- `/opt/serverhub-next/services.yml`：服务清单配置，后续新增服务只改这里。
- `/etc/systemd/system/serverhub-next.service`：独立 systemd 服务。

## 设计原则
1. UI 不借鉴现有任何页面。
2. 前后端全部新写。
3. 配置驱动，减少以后改代码的成本。
4. 默认只读，危险写操作后续单独加鉴权。
5. 每个页面按功能分区，避免首页臃肿。

## 扩展方式
- 新增服务：只在 `services.yml` 增加一项。
- 新增能力页：在 `backend.py` 增加 API，在 `static/index.html` 增加视图。
- 新增日志源：在 `backend.py` 的日志映射里加 unit 或文件。
- 新增自动化任务：建议未来单独加 `tasks.yml` 或 `jobs/` 模块，不混在首页里。

## 当前验证
- `serverhub-next.service` 已运行于 `9600`
- `GET /api/health` 正常
- `GET /api/state` 正常
- `GET /` 正常返回全新页面
- `HEAD /` 已支持
