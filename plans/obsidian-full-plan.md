---
source_path: /root/.openclaw/workspace/obsidian-full-plan.md
imported_at: 2026-06-19T13:19:45
category: plans
---

# Obsidian 共享知识库 — 全链条实施计划

> 目标：用 Obsidian 管理服务器所有 .md 文件，打通 OpenClaw + Hermes 记忆，
> 最终在 Dashboard Web 端可视化知识图谱。

---

## 🗺️ 全链条总览

```
Phase 1           Phase 2            Phase 3             Phase 4
┌──────────┐     ┌──────────────┐    ┌──────────────┐    ┌────────────────┐
│ 你的电脑  │     │   服务器       │    │   Agent 双向   │    │  Dashboard Web │
│          │     │              │    │   记忆互通      │    │  知识图谱      │
│ Obsidian │←Git→│ obsidian-    │←── │ OpenClaw ─┐  │    │ 🕸️ 可视化     │
│ 桌面端   │     │ vault/       │    │ Hermes  ──┘  │    │ 文件关联关系    │
└──────────┘     └──────────────┘    └──────────────┘    └────────────────┘
```

---

## Phase 1：你的电脑 — 安装 Obsidian

### 1.1 下载安装
| 平台 | 地址 |
|---|---|
| macOS | https://obsidian.md/download |
| Windows | https://obsidian.md/download |
| Linux | AppImage / Snap / Flatpak |
| iOS/Android | App Store / Google Play |

### 1.2 推荐插件（先装 3 个核心）
| 插件 | 用途 |
|---|---|
| **Git** | 自动同步 vault 到 Git 仓库 |
| **Dataview** | 用查询语法聚合 Agent 数据 |
| **Calendar** | 日历视图，配合 daily notes |

---

## Phase 2：服务器 — 创建共享 Vault

### 2.1 目录结构
```
/root/obsidian-vault/          ← Git 仓库根目录 (Obsidian vault)
├── .obsidian/                 ← Obsidian 配置（自动生成）
├── MEMORY.md                  ← 长期记忆（Agent 读写）
├── USER.md                    ← 用户信息
├── SOUL.md                    ← 智能体人格
├── AGENTS.md                  ← 行为规则
├── TOOLS.md                   ← 工具配置
├── IDENTITY.md                ← 身份标识
├── memory/                    ← 每日笔记
│   ├── 2026-06-18.md
│   └── 2026-06-19.md
├── projects/                  ← 项目文档
│   └── hermes-dashboard.md
└── plans/                     ← 计划文档
    ├── hermes-optimization-plan.md
    └── knowledge-graph-plan.md
```

### 2.2 迁移现有文件
```
源位置                                       →  vault 位置
─────────────────────────────────────────────────────────────
/root/.openclaw/workspace/MEMORY.md          → /root/obsidian-vault/MEMORY.md
/root/.openclaw/workspace/SOUL.md            → /root/obsidian-vault/SOUL.md
/root/.openclaw/workspace/USER.md            → /root/obsidian-vault/USER.md
/root/.openclaw/workspace/AGENTS.md          → /root/obsidian-vault/AGENTS.md
/root/.openclaw/workspace/TOOLS.md           → /root/obsidian-vault/TOOLS.md
/root/.openclaw/workspace/IDENTITY.md        → /root/obsidian-vault/IDENTITY.md
/root/.hermes/memories/MEMORY.md             → (合并到) vault/MEMORY.md
/root/.hermes/memories/USER.md               → (合并到) vault/USER.md
/root/.hermes/SOUL.md                        → (合并到) vault/SOUL.md
/root/.openclaw/workspace/hermes-optimization-plan.md → vault/plans/
/root/.openclaw/workspace/knowledge-graph-plan.md     → vault/plans/
```

### 2.3 关键操作：创建 Symlink
让两个 Agent 不改代码就能读/写 vault：
```bash
# OpenClaw → vault
ln -sf /root/obsidian-vault/MEMORY.md  /root/.openclaw/workspace/MEMORY.md
ln -sf /root/obsidian-vault/SOUL.md    /root/.openclaw/workspace/SOUL.md
ln -sf /root/obsidian-vault/USER.md    /root/.openclaw/workspace/USER.md
ln -sf /root/obsidian-vault/memory     /root/.openclaw/workspace/memory

# Hermes → vault
ln -sf /root/obsidian-vault/MEMORY.md  /root/.hermes/memories/MEMORY.md
ln -sf /root/obsidian-vault/USER.md    /root/.hermes/memories/USER.md
ln -sf /root/obsidian-vault/SOUL.md    /root/.hermes/SOUL.md
```

---

## Phase 3：Git 同步 — 打通服务器和你的电脑

### 3.1 服务器端
```bash
cd /root/obsidian-vault
git init
git config user.name "Server Agent"
git config user.email "agent@hermes.local"
echo ".obsidian/" > .gitignore
git add -A && git commit -m "初始化共享知识库"
# 推到你的私有仓库
git remote add origin git@github.com:yourname/hermes-knowledge.git
git push -u origin main
```

### 3.2 自动化：Agent 写入 → 自动 commit
写一个 cron 脚本（或 agent 写完文件后触发）：
```bash
#!/bin/bash
cd /root/obsidian-vault
if ! git diff --quiet || ! git diff --cached --quiet; then
  git add -A
  git commit -m "auto: $(date -Iseconds)"
  git push
fi
```
每 5 分钟跑一次，或 Agent 写入后手动触发。

### 3.3 你的电脑
```bash
git clone git@github.com:yourname/hermes-knowledge.git ~/Documents/hermes-knowledge
```
然后用 Obsidian 打开 `~/Documents/hermes-knowledge` 作为 vault。

---

## Phase 4：Dashboard Web — 知识图谱可视化

### 4.1 后端：`GET /api/knowledge-graph`
扫描 vault 下所有 `.md`，提取引用关系：
```python
# 伪代码
for file in vault.glob("**/*.md"):
    content = read(file)
    # 匹配 [[wikilink]] 和 [text](file.md)
    refs = re.findall(r'\[\[(.+?)\]\]|\[.+?\]\((.+?\.md)\)', content)
    for ref in refs:
        links.append({source: file, target: resolve(ref)})
```

### 4.2 前端：ECharts 力导向图
- 节点 = 文件，边 = 引用关系
- 颜色按目录分类（memory=绿, openclaw=蓝, hermes=紫, plans=橙）
- 节点大小按引用次数
- 拖拽/缩放/点击看详情
- 侧边栏：选中文件的信息 + 引用列表

### 4.3 刷新机制
- 首次加载全量
- 每 60 秒自动刷新（文件可能有 Agent 新写入）
- 手动刷新按钮

---

## ⏱️ 时间表

| 阶段 | 内容 | 预计耗时 |
|---|---|---|
| Phase 1 | 你下载 Obsidian + 装插件 | 10 分钟 |
| Phase 2 | 我创建 vault / 迁移文件 / 建 symlink | 15 分钟 |
| Phase 3 | 配 Git 仓库 / auto-commit / 你 clone | 10 分钟 |
| Phase 4 | 后端 API + 前端知识图谱 | 30 分钟 |
| **合计** | | **~65 分钟** |

---

## ⚠️ 注意事项

| 事项 | 处理 |
|---|---|
| Obsidian 免费吗 | 个人使用完全免费。同步($5/月)可用 Git 替代 |
| 文件合并冲突 | Hermes/Obsidian/OpenClaw 同时写 MEMORY.md 几率低；Git merge 兜底 |
| 敏感信息泄漏 | Git 仓库必须设为 **private** |
| SOUL.md 等 Agent 配置文件 | Obsidian 里可编辑但别改结构，否则 Agent 行为异常 |
| 图谱性能 | 50 个节点 ECharts 力导向图毫无压力 |
