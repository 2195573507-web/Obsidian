---
title: 原生前端与 UIUX 产品化实践
type: wiki
category: software
updated: 2026-06-30
tags: [frontend, uiux, vanilla-js, product]
source: hermes-multi-agent-knowledge-expansion
---

# 原生前端与 UIUX 产品化实践

> 关联：[[软件工程/现代软件工程实战总纲|现代软件工程实战总纲]]、[[软件工程/FastAPI 后端工程实践|FastAPI 后端工程实践]]

## 1. 原生前端适用场景

HTML/CSS/JS 无框架适合管理后台小工具、监控面板、内部系统、轻量 UI、FastAPI 静态页。优点是部署简单、依赖少、加载快；缺点是大型状态管理困难，需要规范。

## 2. 结构

```text
static/
  index.html
  css/base.css
  css/layout.css
  css/components.css
  js/api.js
  js/state.js
  js/render.js
  js/events.js
```

单文件也可以，但必须按 Design Tokens、Reset、Layout、Components、Responsive、API、State、Render、Events 分区。

## 3. CSS

先定义 design tokens：颜色、字体、间距、圆角、阴影。布局类和组件类分离。移动端考虑触控 44px、表格横滚、单列卡片、不依赖 hover、图表不溢出。

## 4. JavaScript

状态集中管理，fetch 封装到 API 层，render 函数只负责渲染，事件层只负责绑定。任何插入 HTML 的用户内容都要 escape，防 XSS。

## 5. API 契约

前端必须匹配后端返回结构。后端返回 `{items,total}` 时不能直接 `data.map`。接口文档记录 query、request、response、error code。

## 6. UIUX 原则

用户不是来欣赏界面，而是完成任务。每个操作都要有反馈：加载中、成功、失败、空状态、错误原因、下一步建议。危险操作要二次确认。

## 7. 产品化

产品化要考虑首次使用、默认配置、权限、导入导出、搜索筛选、批量操作、审计日志、帮助文档、错误恢复、升级迁移、移动端体验。
