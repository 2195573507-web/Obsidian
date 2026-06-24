---
title: "server.zmjjkkk.fun 统一服务器入口计划"
tags: ['plan', 'project']
updated: 2026-06-24
---

# server.zmjjkkk.fun 统一服务器入口计划

> 状态：规划中，暂不改代码  
> 目标域名：`server.zmjjkkk.fun`  
> 服务器：`38.55.146.137`  
> 当前时间：2026-06-19

## 1. 目标

把当前服务器上的多个 Web 面板整合成一个统一入口：

- 一个域名访问：`server.zmjjkkk.fun`
- 一个首页导航：集中进入监控、Agent 控制台、知识库、文件结构
- 一个状态总览：显示各服务是否在线
- 后续可扩展为服务器管理中心

当前先做计划，不启动实施，不修改现有服务代码。

## 2. 当前已有 Web 服务

| 端口 | 服务 | 当前用途 | 状态 |
|---|---|---|---|
| `9000` | Hermes System Monitor | 系统监控面板 | 已运行 |
| `9100` | Hermes Dashboard | Agent 管理控制台 | 已运行 |
| `9200` | Obsidian Knowledge Web | Obsidian 知识库 Web | 已运行 |
| `9300` | Server File Structure Web | 服务器文件结构浏览器 | 已运行 |

## 3. 最终命名

推荐最终入口名称：

```text
server.zmjjkkk.fun
```

理由：

- 简洁，表示这是整台服务器的统一入口
- 不局限于 Agent、监控或知识库
- 后续可以继续挂更多功能，例如备份、日志、服务管理、数据库管理

## 4. 访问结构设计

### 方案 A：统一首页 + 子路径代理（推荐）

```text
https://server.zmjjkkk.fun/          → 统一入口首页
https://server.zmjjkkk.fun/monitor   → 9000 系统监控
https://server.zmjjkkk.fun/agent     → 9100 Agent 控制台
https://server.zmjjkkk.fun/notes     → 9200 Obsidian 知识库
https://server.zmjjkkk.fun/files     → 9300 文件结构浏览器
```

优点：

- 只有一个域名，最清晰
- 用户访问体验最好
- 首页可以做成服务器总控台

缺点：

- 有些现有前端如果写死了绝对路径，可能需要适配子路径
- Nginx 配置稍复杂

### 方案 B：统一首页 + 端口跳转

```text
https://server.zmjjkkk.fun/          → 统一入口首页
http://38.55.146.137:9000            → 系统监控
http://38.55.146.137:9100            → Agent 控制台
http://38.55.146.137:9200            → 知识库
http://38.55.146.137:9300            → 文件结构
```

优点：

- 实施最快
- 不需要改现有前端
- 风险最低

缺点：

- 仍然暴露多个端口
- 体验不如子路径统一

### 方案 C：多个子域名

```text
monitor.zmjjkkk.fun → 9000
agent.zmjjkkk.fun   → 9100
notes.zmjjkkk.fun   → 9200
files.zmjjkkk.fun   → 9300
server.zmjjkkk.fun  → 统一首页
```

优点：

- 每个服务路径最干净
- 对现有前端兼容性好

缺点：

- DNS 记录更多
- 入口分散一点

## 5. 推荐实施路线

推荐采用“两阶段”：

### Phase 1：先做统一首页

目标：快速获得一个统一入口，不影响现有服务。

内容：

1. 新建一个轻量首页服务或静态页面
2. 绑定到 `server.zmjjkkk.fun`
3. 首页显示四个入口卡片：
   - 系统监控 `9000`
   - Agent 控制台 `9100`
   - Obsidian 知识库 `9200`
   - 文件结构 `9300`
4. 首页显示基础状态：
   - 每个端口是否在线
   - 服务名称
   - 简短说明
5. 暂时只跳转到现有端口，不代理子路径

Phase 1 完成后效果：

```text
server.zmjjkkk.fun
└── 首页
    ├── 打开系统监控
    ├── 打开 Agent 控制台
    ├── 打开知识库
    └── 打开文件结构
```

### Phase 2：再接 Nginx 反向代理

目标：让所有服务都进入 `server.zmjjkkk.fun` 名下。

内容：

1. 安装/配置 Nginx
2. 配置 `server.zmjjkkk.fun`
3. 配置反向代理：
   - `/monitor` → `127.0.0.1:9000`
   - `/agent` → `127.0.0.1:9100`
   - `/notes` → `127.0.0.1:9200`
   - `/files` → `127.0.0.1:9300`
4. 如有前端路径问题，再逐个适配
5. 可选：申请 HTTPS 证书

### Phase 3：升级成服务器管理中心

目标：从“导航页”升级为“管理中心”。

可加入功能：

- systemd 服务状态
- 一键重启指定服务
- 最近错误日志
- 自动备份状态
- 磁盘/内存/CPU 摘要
- OpenClaw/Hermes 运行状态
- Obsidian vault 状态

## 6. 首页 UI 设计

首页建议布局：

```text
┌────────────────────────────────────────────┐
│ server.zmjjkkk.fun                         │
│ 至秦的服务器控制中心                         │
├────────────────────────────────────────────┤
│ CPU / 内存 / 磁盘 / 运行时间                 │
├──────────────┬──────────────┬──────────────┤
│ 系统监控      │ Agent 控制台   │ 知识库        │
│ :9000        │ :9100        │ :9200        │
├──────────────┴──────────────┴──────────────┤
│ 文件结构浏览器 :9300                         │
├────────────────────────────────────────────┤
│ 服务状态：9000 ✅ 9100 ✅ 9200 ✅ 9300 ✅     │
└────────────────────────────────────────────┘
```

风格建议：

- 暗色科技风，与现有 Dashboard 风格一致
- 卡片式入口
- 每个服务有状态灯
- 移动端也能访问
- 显示服务器 IP、域名、当前时间

## 7. 后端 API 设计

如果首页不是纯静态页，可以加这些 API：

```text
GET /api/status
```

返回：

```json
{
  "server": "38.55.146.137",
  "domain": "server.zmjjkkk.fun",
  "services": [
    { "name": "系统监控", "port": 9000, "url": "http://38.55.146.137:9000", "online": true },
    { "name": "Agent 控制台", "port": 9100, "url": "http://38.55.146.137:9100", "online": true },
    { "name": "知识库", "port": 9200, "url": "http://38.55.146.137:9200", "online": true },
    { "name": "文件结构", "port": 9300, "url": "http://38.55.146.137:9300", "online": true }
  ]
}
```

可选 API：

```text
GET /api/system-summary
GET /api/services
GET /api/logs/recent
GET /api/backups/status
```

## 8. DNS 计划

需要在域名管理处添加：

```text
类型：A
主机记录：server
记录值：38.55.146.137
```

最终解析：

```text
server.zmjjkkk.fun → 38.55.146.137
```

如果后续使用多个子域名，可以追加：

```text
monitor.zmjjkkk.fun → 38.55.146.137
agent.zmjjkkk.fun   → 38.55.146.137
notes.zmjjkkk.fun   → 38.55.146.137
files.zmjjkkk.fun   → 38.55.146.137
```

## 9. Nginx 草案

仅作为后续参考，当前不执行。

```nginx
server {
    listen 80;
    server_name server.zmjjkkk.fun;

    location / {
        proxy_pass http://127.0.0.1:9400;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /monitor/ {
        proxy_pass http://127.0.0.1:9000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /agent/ {
        proxy_pass http://127.0.0.1:9100/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /notes/ {
        proxy_pass http://127.0.0.1:9200/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /files/ {
        proxy_pass http://127.0.0.1:9300/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 10. 端口规划

当前端口：

| 端口 | 用途 |
|---|---|
| `9000` | 系统监控 |
| `9100` | Agent 控制台 |
| `9200` | Obsidian 知识库 |
| `9300` | 文件结构浏览器 |

建议新增：

| 端口 | 用途 |
|---|---|
| `9400` | `server.zmjjkkk.fun` 统一入口首页后端 |

原因：

- 不占用已有服务
- 逻辑清晰
- 后续 Nginx 只需要把 `/` 转发到 `9400`

## 11. 风险点

1. 子路径代理可能导致前端静态资源路径错误
2. HTTPS 证书需要 DNS 正确解析后再申请
3. 如果直接关闭端口只走 Nginx，需要确认所有代理都正常
4. 一键重启服务属于敏感操作，建议后续单独设计
5. 当前用户要求“先别改代码”，所以本计划只记录设计，不执行

## 12. 推荐下一步

下一步建议只做 Phase 1：

1. 新建 `/opt/server-home/`
2. 使用 `9400` 端口跑统一入口首页
3. 首页卡片跳转到现有端口
4. 不碰 `9000/9100/9200/9300` 现有代码
5. 等确认好 UI 后，再考虑 Nginx 和域名绑定

## 13. 最终结论

最终名字建议确定为：

```text
server.zmjjkkk.fun
```

最稳妥实施路径：

```text
先做统一首页 → 再接域名 → 再做 Nginx 代理 → 最后升级管理中心
```

当前阶段只保留计划，暂不实施。
