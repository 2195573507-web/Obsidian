---
source_path: /root/hermes-dashboard-research-report.md
imported_at: 2026-06-19T13:19:46
category: dashboards
---

# Hermes 管理 Dashboard 技术调研报告

> 调研日期：2026年6月18日
> 调研范围：6 个开源 LLM 管理平台 + Hermes Agent 自身数据源

---

## 一、Hermes Agent 现有数据基础设施

### 1.1 数据存储：state.db（SQLite）

**路径**：`~/.hermes/state.db`（约 5MB，WAL 模式，支持并发读写）

**核心表结构**：

```
sessions
├── id TEXT PRIMARY KEY                -- 会话ID（如 "20260618_094440_75b68920"）
├── source TEXT NOT NULL               -- 来源平台（cli/telegram/discord/subagent/cron/qqbot...）
├── model TEXT                         -- 模型名
├── model_config TEXT                  -- 模型配置 JSON
├── started_at / ended_at REAL         -- 起止时间戳
├── end_reason TEXT                    -- 结束原因
├── message_count INTEGER              -- 消息数
├── tool_call_count INTEGER            -- 工具调用数
├── input_tokens INTEGER               -- 输入 token 数
├── output_tokens INTEGER              -- 输出 token 数
├── cache_read_tokens INTEGER          -- 缓存读取 token 数 ⭐
├── cache_write_tokens INTEGER         -- 缓存写入 token 数 ⭐
├── reasoning_tokens INTEGER           -- 推理 token 数
├── billing_provider / billing_base_url / billing_mode -- 计费信息
├── estimated_cost_usd / actual_cost_usd / cost_status  -- 成本估算
├── title TEXT                         -- 会话标题
├── api_call_count INTEGER             -- API 调用次数
└── archived INTEGER                   -- 归档标记

messages
├── id INTEGER PRIMARY KEY AUTOINCREMENT
├── session_id TEXT → sessions(id)
├── role TEXT                          -- user/assistant/tool/system
├── content TEXT                       -- 消息内容
├── tool_calls TEXT                    -- 工具调用 JSON
├── tool_name TEXT                     -- 工具名
├── timestamp REAL
├── token_count INTEGER                -- 单条消息 token 数
├── finish_reason TEXT                 -- 完成原因（stop/tool_calls/...）
├── reasoning / reasoning_content      -- 推理/思考内容
└── active INTEGER DEFAULT 1           -- 活跃标记（用于压缩后标记失效消息）

messages_fts (FTS5 虚拟表)             -- 全文本搜索（unicode61 tokenizer）
messages_fts_trigram (FTS5 虚拟表)     -- CJK 子串搜索（trigram tokenizer）
state_meta                             -- 键值元数据
compression_locks                      -- 压缩锁
```

**关键设计**：
- **Token 统计**：session 级别聚合（input/output/cache_read/cache_write/reasoning），每条 message 有 per-message token_count
- **成本估算**：`estimated_cost_usd` 基于模型定价表实时计算（依赖 `agent/usage_pricing.py`）
- **FTS5 全文本搜索**：支持 Boolean 查询、短语匹配、前缀匹配、源平台过滤
- **WAL 模式**：支持多进程并发（gateway + CLI + web_server 共享一个 state.db）
- **写冲突处理**：应用层随机抖动重试（20-150ms，最多 15 次）

### 1.2 现有 Dashboard 后端（FastAPI + React）

**技术栈**：
- 后端：Python FastAPI + uvicorn（默认端口 9119）
- 前端：React (Vite) + shadcn/ui 组件库
- 插件系统：JavaScript SDK + Python FastAPI Router

**已有 API 端点**（`hermes_cli/web_server.py`）：

| 端点 | 功能 | 对 Dashboard 价值 |
|------|------|-------------------|
| `GET /api/analytics/usage` | 每日 token/成本/API 调用趋势 + 按模型汇总 | ⭐⭐⭐ 核心指标面板 |
| `GET /api/analytics/models` | 按模型详细统计（含能力元数据、缓存命中率） | ⭐⭐⭐ 模型对比 |
| `GET /api/sessions` | 分页会话列表（支持 source/include_children） | ⭐⭐ 会话管理 |
| `GET /api/profiles/sessions` | 跨 Profile 会话聚合（只读挂载其他 Profile DB） | ⭐⭐ 多 Profile 视图 |
| `GET /api/sessions/search` | FTS5 全文本搜索（支持 source 过滤） | ⭐⭐ 搜索 |
| `GET /api/config` | 当前配置（含 provider/model/toolsets/skills） | ⭐⭐ 配置查看 |
| `PUT /api/config` | 更新配置 | ⭐⭐ 配置编辑 |
| `GET /api/model/info` | 当前模型详情 | ⭐ 模型状态 |
| `POST /api/model/set` | 切换模型 | ⭐ 模型切换 |
| `GET /api/status` | Gateway 运行状态、活跃 sessions、代理信息 | ⭐⭐ 系统监控 |
| `GET /api/system/stats` | 系统资源统计 | ⭐ 资源监控 |
| `GET /api/skills` | Skills 列表及状态 | ⭐ 能力管理 |
| `PUT /api/skills/toggle` | 启用/禁用 Skills | ⭐ |
| `GET /api/tools/toolsets` | Toolsets 列表及状态 | ⭐ 能力管理 |
| `GET /api/profiles` | Profile 列表 | ⭐ 多 Profile |
| `GET /api/portal` | Nous Portal 连接状态 | ⭐ |
| `GET /api/dashboard/themes` | 主题列表 | ⭐ 主题管理 |
| `PUT /api/dashboard/theme` | 切换主题 | ⭐ 主题切换 |

### 1.3 Insights 引擎（`agent/insights.py`）

`hermes insights` 命令提供的分析维度，可直接复用到 Dashboard：

```
📊 面板内容                     Dashboard 映射
───────────────────────────────────────────────────
📋 Overview                    → 顶部 KPI 卡片区
   - Sessions / Messages / Tool calls / User msgs
   - Input / Output / Total tokens
   - Estimated cost / Actual cost
   - Active hours / Avg session duration / Avg msgs per session

🤖 Models Used                → 模型用量饼图/柱状图
   - Per-model sessions, token breakdown, cost

📱 Platforms                   → 平台用量分类饼图
   - cli / telegram / discord / qqbot / cron / subagent

🔧 Top Tools                   → 工具使用排行
   - Tool name, call count, percentage

🧠 Top Skills                  → 技能加载排行
   - Skill name, view count, edit count, last used

📅 Activity Patterns           → 活动热力图/日历
   - Daily/hourly distribution, peak hours, streaks

🏆 Notable Sessions            → 会话排行
   - Most messages / tokens / tool calls / longest
```

### 1.4 现有 Dashboard 的插件/主题扩展系统

- **主题系统**：YAML 文件（palette/typography/layout/assets/componentStyles），放在 `~/.hermes/dashboard-themes/`
- **UI 插件**：`manifest.json` + JS Bundle (IIFE) + 可选 Python API 后端
- **Shell Slot 系统**：sidebar / header-left / header-right / footer 等命名槽位
- **页面覆盖**：插件可替换或增强内置页面

---

## 二、6 个开源 LLM 管理平台 Dashboard 分析

### 2.1 LiteLLM（github.com/BerriAI/litellm）

| 维度 | 详情 |
|------|------|
| **定位** | LLM 代理网关，统一多 Provider API |
| **前端** | Svelte/SvelteKit（Admin UI 从 proxy 分离） |
| **后端** | Python (FastAPI 风格) + SQLite/PostgreSQL |
| **图表库** | Chart.js |

**Dashboard 核心功能**：
1. **Overview 仪表盘** — 总请求数、成功率、P50/P95/P99 延迟、活跃 Key 数、总 Spend
2. **Spend & Usage 页面** — 按时间范围/模型/Key/团队过滤的消费趋势图，支持 CSV 导出
3. **Models 页面** — 所有接入模型列表，显示每分钟 RPM/TPM 限制、上下文窗口、价格
4. **API Keys 页面** — Key 生成、额度分配、速率限制（RPM/TPM）、过期时间、使用日志
5. **Teams 页面** — 团队管理，成员分配，团队级预算和限制
6. **Logs 页面** — 请求/响应日志（含原始 payload），按状态码/模型/Key 过滤
7. **Virtual Keys** — 外部 LLM Provider 的 API Key 的管理和轮换
8. **Guardrails** — PII 检测、内容过滤规则

**Token 统计方案**：通过 Provider 返回的 `usage` 字段（prompt_tokens/completion_tokens）实时记录，存储在 `LiteLLM_SpendLogs` 表，每小时聚合一次。

**配置管理 UI 模式**：表单式配置，Model/Key/RateLimit 分 Tab 管理，支持 YAML 导入。

**可借鉴模式**：
- **Spend 趋势图**：时间序列折线图 + 预算进度条（已用/上限）
- **Key 管理面板**：表格 + 弹窗表单 + 一键复制/轮换
- **速率限制配置**：RPM/TPM 数值输入 + 实时状态指示灯

### 2.2 Open WebUI（github.com/open-webui/open-webui）

| 维度 | 详情 |
|------|------|
| **定位** | LLM 聊天前端，类似 ChatGPT 的本地化 UI |
| **前端** | SvelteKit → 正在迁移 React/Vite |
| **后端** | Python (FastAPI) + SQLite |
| **图表库** | 自定义组件（轻量） |

**Admin Panel 核心功能**：
1. **Dashboard 首页** — 模型统计、用户数、文档数、最近活动
2. **Connections 页面** — 管理 OpenAI-compatible API 连接（URL + API Key）
3. **Models 页面** — 模型列表、启用/禁用、模型能力标签（vision/tools/...）
4. **Users 页面** — 用户管理、权限控制（user/admin）、分组
5. **Documents 页面** — RAG 知识库管理、文件上传、向量化状态
6. **Settings 页面** — 系统设置（默认模型、界面参数、外部服务）
7. **Evaluations 页面**（Arena 模式）— 模型对比盲测

**Token 统计方案**：通过 OpenAI usage 响应字段提取，前端实现 Token 计数显示（无持久化聚合 Dashboard）

**配置管理 UI 模式**：列表 + 行内编辑 + 开关切换。API 连接用卡片展示，每张卡片管理单个 Provider。

**可借鉴模式**：
- **连接管理卡片**：每个 Provider 一张卡片，显示名称/URL/模型数/状态指示灯
- **模型能力标签**：用彩色 Badge 标记 vision/tools/reasoning 等能力
- **设置分组**：左侧导航树 + 右侧表单面板

### 2.3 Helicone（github.com/Helicone/helicone）

| 维度 | 详情 |
|------|------|
| **定位** | LLM 可观测性/代理平台 |
| **前端** | Next.js (React) + TailwindCSS |
| **后端** | Node.js (tRPC) + PostgreSQL + ClickHouse |
| **图表库** | Recharts / Tremor |

**Dashboard 核心功能**：
1. **Requests 页面** — 所有请求的表格视图（模型、延迟、状态码、Token、成本），支持高级过滤
2. **Metrics 页面** — 延迟 P50/P95/P99、请求量、错误率、Token 消耗趋势图
3. **Cost 页面** — 按模型/用户/时间的成本分解，预算告警
4. **Cache 页面** — 缓存命中率分析，按模型/时间对比
5. **Users 页面** — 终端用户用量排行，每个用户的 Token/成本/请求统计
6. **Prompts 页面** — Prompt 模板管理和版本控制
7. **Experiments 页面** — A/B 测试和评估
8. **Rate Limiting** — 按用户/全局设置速率限制
9. **Sessions 页面** — 按 session 归组的请求流（多轮对话跟踪）

**Token 统计方案（最完善的方案之一）**：
- **实时记录**：每次 API 调用从响应体中提取 `usage` 并结构化存储
- **缓存命中率**：根据 `usage.prompt_tokens_details.cached_tokens` 计算 `cached_tokens / total_prompt_tokens`
- **ClickHouse 时序存储**：支持秒级聚合和长时间范围查询
- **数据 Pipeline**：Proxy → Kafka → ClickHouse，前端查询用 tRPC

**配置管理 UI 模式**：左侧分 NavGroup 导航 + 右侧数据密集型表格 + 顶部时间范围选择器 + 过滤器栏。

**可借鉴模式（最值得参考的可观测性设计）**：
- **时间范围选择器**：统一在顶部，影响所有图表（与 Langfuse 类似）
- **过滤器栏**：多条件组合过滤（模型/状态码/用户/时间），类似日志分析工具
- **Cache 分析页**：折线图显示缓存命中率趋势 + 按模型的命中率对比表
- **成本管理**：预算进度条 + 超过阈值变色告警
- **请求详情展开**：点击表格行展开完整 request/response payload（JSON 高亮）

### 2.4 Langfuse（github.com/langfuse/langfuse）

| 维度 | 详情 |
|------|------|
| **定位** | LLM 工程平台（Tracing + Evals + Prompt Mgmt） |
| **前端** | Next.js (React) + TailwindCSS + shadcn/ui |
| **后端** | Next.js API Routes + PostgreSQL + ClickHouse |
| **图表库** | Recharts / Tremor |

**Dashboard 核心功能**：
1. **Dashboard 首页** — 总 Traces/Observations、总 Token、总成本、延迟分布、模型分布
2. **Tracing 页面** — 树形 Span 视图（类似 Jaeger/Zipkin），含输入输出/Token/成本/延迟
3. **Generations 页面** — LLM 调用列表，显示 Prompt/Completion/Token/Cost/延迟
4. **Scores 页面** — 评分/评估面板，支持人工打分和自动评估
5. **Datasets 页面** — 测试数据集管理
6. **Prompts 页面** — Prompt 模板（版本化、标签、关联评测）
7. **Playground 页面** — 在线 Prompt 调试

**Token 统计方案**：
- **SDK 级别**：在 `generation` span 上记录 `usage`（由调用方或 proxy 上报）
- **自动聚合**：Dashboard 首页按时间/模型/环境聚合 Token
- **ClickHouse**：时序数据支持大时间窗口聚合（周/月/季度）

**配置管理 UI 模式**：项目级导航 + 顶部时间过滤器 + 数据表格 + 侧边详情面板。

**可借鉴模式**：
- **Dashboard 首页设计**：KPI 卡片 + 趋势图 + TopN 列表，信息密度高
- **时间过滤器**：预设按钮（24h/7d/30d/90d）+ 自定义日期范围
- **环境标签**：production/staging/development 过滤

### 2.5 Portkey（github.com/Portkey-AI/gateway）

| 维度 | 详情 |
|------|------|
| **定位** | AI 网关（路由、负载均衡、金丝雀部署） |
| **前端** | React + TailwindCSS |
| **后端** | Node.js + PostgreSQL |
| **图表库** | Recharts |

**Dashboard 核心功能**：
1. **Overview** — 总请求量、错误率、延迟 P50/P95、活跃 Keys
2. **Analytics** — 请求量和延迟趋势、按模型/provider 的分布
3. **Virtual Keys** — API Key 管理和分发系统（支持从主 Key 派生子 Key）
4. **Configs** — 路由规则（负载均衡权重、回退链、重试逻辑、缓存策略）
5. **Guardrails** — 内容审核规则（PII/有害内容/自定义规则）
6. **Logs** — 标准化请求日志（支持下载和 webhook 推送）
7. **Spend** — 成本分析（按 provider/model/key 维度）
8. **Canary Testing** — 金丝雀部署（A/B 分流到不同模型/版本对比）

**Token 统计方案**：Gateway 层拦截所有请求/响应，从 OpenAI-format response 提取 usage 字段，聚合存储。

**配置管理 UI 模式（最丰富的网关配置 UI）**：
- **路由规则**：拖拽式负载均衡权重配置 + 优先级排序
- **Virtual Key**：列表 + 创建向导（分步表单）
- **Config as Code**：支持 JSON/YAML 导入导出

**可借鉴模式**：
- **Config 页面设计**：以"配置对象"为粒度的卡片列表，每个卡片可展开/折叠详情
- **负载均衡可视化**：拖拽调节权重条
- **Virtual Key 表**：支持列排序、过滤、快速复制

### 2.6 One API（github.com/songquanpeng/one-api）

| 维度 | 详情 |
|------|------|
| **定位** | OpenAI 接口管理 & 分发系统（最轻量） |
| **前端** | React + Ant Design |
| **后端** | Go (Gin) + SQLite/MySQL |
| **图表库** | Ant Design Charts / ECharts |

**Dashboard 核心功能**：
1. **首页仪表盘** — 今日请求量/Token/收入，24h 请求趋势图，模型请求分布
2. **渠道管理** — 上游 API 渠道（Base URL + API Key + 模型映射 + 权重 + 分组）
3. **令牌管理** — 面向用户的 API Key 生成（额度限制、过期时间、模型白名单、IP 白名单）
4. **用户管理** — 用户分组、配额管理
5. **日志** — 请求日志（消耗 Token、渠道、IP、状态码）
6. **兑换** — Token 兑换码系统（预充值码）
7. **设置** — 系统设置、价格配置（自定义各模型价格）

**Token 统计方案（最简实现）**：
- **渠道 → 模型 → 价格**三层映射：管理员为每个渠道-模型组合设定价格
- **按令牌统计**：每次请求根据渠道模型组合乘以对应价格，实时扣减令牌额度
- **SQLite 存储**：日志表 `logs` 记录每次请求的 token 消耗和渠道信息

**配置管理 UI 模式**：Ant Design Pro 标准管理后台布局 — 左侧菜单 + 表格 + 弹窗表单。

**可借鉴模式**：
- **渠道健康检查**：手动/自动测试渠道连通性，显示状态指示灯
- **价格配置**：模型-价格映射表，行内编辑
- **令牌管理**：一键生成 + 复制 + 批量操作

---

## 三、Dashboard 功能设计建议

### 3.1 页面结构

基于以上 7 个项目的共同模式，推荐以下页面导航：

```
Hermes Dashboard
├── 📊 Overview           -- 总览仪表盘（KPI卡片 + 趋势图）
├── 💬 Sessions           -- 会话管理（列表 + 搜索 + 详情）
├── 📈 Analytics           -- 深度分析（按平台/模型/时间维度的 Token 和成本）
├── 🛠️ Tools & Skills     -- 能力和工具管理
├── ⚙️ Configuration       -- Provider/API Key/模型配置管理
├── 📱 Platforms           -- 多平台管理（Cli/Gateway状态）
└── 🔌 Plugins & Themes    -- 扩展管理
```

### 3.2 Overview 页面设计

借鉴 Helicone + Langfuse 的 Dashboard 首页模式：

```
┌────────────────────────────────────────────────────────────┐
│  时间范围选择器:  [24h] [7d] [30d] [90d] [自定义...]        │
├──────────┬──────────┬──────────┬──────────┬───────────────┤
│ Sessions │ Messages │ Tools    │ Total    │ Est. Cost     │  ← KPI 卡片
│   5      │   260    │  140     │ 8.9M     │  $1.23        │
├──────────┴──────────┴──────────┴──────────┴───────────────┤
│ ┌──────────────────────────┐ ┌───────────────────────────┐ │
│ │ Token 消耗趋势 (折线图)   │ │ 按模型用量 (饼图/柱状图)  │ │
│ │ input/output/cache 分色  │ │ deepseek-v4-pro: 8.9M    │ │
│ └──────────────────────────┘ └───────────────────────────┘ │
│ ┌──────────────────────────┐ ┌───────────────────────────┐ │
│ │ 按平台分解 (堆叠柱状图)   │ │ 活动模式 (热力图/日历)    │ │
│ │ cli/telegram/cron/...    │ │ 每天/每小时的会话分布     │ │
│ └──────────────────────────┘ └───────────────────────────┘ │
│ ┌────────────────────────────────────────────────────────┐  │
│ │ Top 工具 / Top 技能 列表                                │  │
│ └────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

### 3.3 Token 统计详细方案

**数据已就绪** — Hermes 的 `sessions` 表已完整记录：
- `input_tokens` — 输入 Token
- `output_tokens` — 输出 Token
- `cache_read_tokens` — 缓存读取 Token（⭐ 缓存命中率的关键）
- `cache_write_tokens` — 缓存写入 Token
- `reasoning_tokens` — 推理 Token

**缓存命中率计算**：
```sql
-- 每个 session 的缓存命中率
SELECT 
    cache_read_tokens * 1.0 / NULLIF(cache_read_tokens + input_tokens, 0) AS cache_hit_rate
FROM sessions WHERE cache_read_tokens > 0;

-- 按模型的聚合缓存命中率
SELECT model,
    SUM(cache_read_tokens) AS total_cache_read,
    SUM(input_tokens) AS total_input,
    ROUND(SUM(cache_read_tokens) * 100.0 / NULLIF(SUM(cache_read_tokens) + SUM(input_tokens), 0), 1) AS cache_hit_rate_pct
FROM sessions
GROUP BY model;
```

**Dashboard 应展示的 Token 指标**：

| 指标 | 展示形式 | 来源 |
|------|---------|------|
| 总 Token 消耗 | KPI 卡片 + 趋势折线图 | sessions.input + output |
| 输入 vs 输出 Token 比 | 堆叠面积图 | sessions.input_tokens / output_tokens |
| 缓存命中率 | 仪表盘/百分比 + 趋势线 | cache_read / (cache_read + input) |
| 推理 Token（如果模型支持） | 单独折线图 | sessions.reasoning_tokens |
| 按模型 Token 分布 | 饼图/环形图 | GROUP BY model |
| 按平台 Token 分布 | 堆叠柱状图 | GROUP BY source |
| 每日 Token 趋势 | 折线图（带预测线） | GROUP BY date(started_at) |
| 预估成本 | KPI 卡片 + 趋势线 | sessions.estimated_cost_usd |
| API 调用次数 | 次要指标 | sessions.api_call_count |

### 3.4 配置管理 UI 模式

借鉴 LiteLLM + Portkey + One API 的配置管理模式：

**Provider / API Key 管理**：
```
┌────────────────────────────────────────────────────────────┐
│  Provider: DeepSeek                            [状态: ✅]  │
│  API Key: sk-****a1b2                    [编辑] [测试连通]  │
│  Base URL: https://api.deepseek.com                       │
│  ───────────────────────────────────────────────────────── │
│  已配置模型:                                               │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │ deepseek-v4  │ │ deepseek-v3  │ │ deepseek-r1  │       │
│  │ ✅ 活跃      │ │ ⏸ 待启用    │ │ ❌ 不可用    │       │
│  │ Input: $2/M  │ │ Input: $1/M  │ │ Input: $4/M  │       │
│  │ Output: $8/M │ │ Output: $4/M │ │ Output: $16/M│       │
│  │ Cache: $0.5/M│ │ Cache: $0.25/M│ │ Cache: $1/M │       │
│  └──────────────┘ └──────────────┘ └──────────────┘       │
│  [+ 添加新 Provider]                                       │
└────────────────────────────────────────────────────────────┘
```

**Fallback 链管理**（借鉴 Portkey 的负载均衡配置）：
```
主模型: deepseek-v4-pro
    ↓ (失败/超时时回退)
备选 1: claude-sonnet-4  [权重: 50%]
    ↓
备选 2: gpt-4o           [权重: 30%]
    ↓
备选 3: gemini-2.5-pro   [权重: 20%]
```

---

## 四、技术架构建议

### 4.1 前端框架

- **推荐方案**：沿用现有 React + Vite + shadcn/ui
- **图表库**：Recharts（轻量、与 React 生态兼容）或 Tremor（Dashboard 专用组件库）
- **数据获取**：React Query (TanStack Query) 用于 API 缓存和自动刷新
- **路由**：React Router

### 4.2 后端增强

**已有基础非常好**，建议新增以下端点：

| 新端点 | 功能 |
|--------|------|
| `GET /api/analytics/cache` | 缓存命中率趋势和按模型的详细分析 |
| `GET /api/analytics/cost` | 成本分解（按模型/平台/时间段） |
| `GET /api/analytics/activity` | 活动模式（按天/小时的热力图数据） |
| `GET /api/sessions/{id}/token-details` | 单个会话的 Token 消耗详情 |
| `GET /api/config/providers` | Provider 列表及连通性测试 |
| `POST /api/config/providers/test` | 测试 Provider 连通性 |
| `GET /api/analytics/export` | 导出统计数据（CSV/JSON） |

### 4.3 Dashboard 页面数据流

```
state.db (SQLite, WAL)
    ↓ 查询 (read-only 或 read-write)
FastAPI 后端 (web_server.py)
    ↓ JSON API
React 前端 (Vite + shadcn/ui)
    ↓ React Query 缓存
Dashboard UI 组件
    - KPI Cards
    - Time-series Charts (Recharts/Tremor)
    - Data Tables (TanStack Table)
    - Filter/Search bars
```

---

## 五、关键发现总结

### 5.1 Hermes 已有的强大基础

1. **Token 数据已完整记录**：`sessions` 表包含 input/output/cache_read/cache_write/reasoning 五个维度的 token 数据，以及 estimated_cost_usd 成本估算
2. **Insights 引擎已成熟**：`agent/insights.py` 提供了 Overview/Models/Platforms/Tools/Skills/Activity 六大分析维度，可直接作为 Dashboard 后端数据源
3. **API 端点已完善**：`/api/analytics/usage` 和 `/api/analytics/models` 已可实现核心 Dashboard 数据展示
4. **插件系统已就绪**：Dashboard 可通过插件扩展，无需修改核心代码

### 5.2 需要补充的能力

1. **缓存命中率展示**：数据已有（cache_read_tokens），但 Dashboard 尚未展示缓存命中率图表
2. **时间范围热力图**：类似 GitHub 贡献图的每日活动热力图
3. **成本告警**：预算设置和超阈值通知
4. **Provider 连通性测试**：一键测试 API Key 是否有效
5. **数据导出**：统计数据的 CSV/JSON 导出

### 5.3 最佳参考项目

| 功能领域 | 最佳参考 |
|---------|---------|
| 可观测性 Dashboard | **Helicone** — 最完善的 Token/Cost/Cache 分析面板 |
| 配置管理 UI | **Portkey** — 网关配置（路由/负载均衡）的可视化设计 |
| Dashboard 首页布局 | **Langfuse** — KPI + 趋势图 + TopN 的信息密度设计 |
| 轻量级参考 | **One API** — 适合快速实现的管理后台模式 |

---

*Report generated by Hermes Agent sub-agent, June 18, 2026*
