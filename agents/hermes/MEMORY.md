服务器 `38.55.146.137`：system-monitor `:9000` `/opt/hermes-system-monitor`，Dashboard `:9100` `/opt/hermes-dashboard`，OpenClaw `127.0.0.1:18789`，shared-memory `:9400` `/opt/shared-agent-memory`；均 systemd 管理，kill PID 自动重启。Hermes config `~/.hermes/config.yaml`。
§
创建 cron 定时任务默认以北京时间（UTC+8）为准，cron 表达式需转换为 UTC（BJT - 8h）。允许安排任意时段任务（包括 BJT 00:00-07:00），按用户指定执行。所有任务输出使用中文，多任务流水线标注 BJT/UTC。
§
用户常用中转站：Hermes/OpenClaw 主要走 `sub.zmjjkkk.fun/v1`；OpenClaw `myproxy` 默认 `gpt-5.5`。备用 `pool.gptstore.club/v1` key 不匹配。OpenClaw 本机有 curl chat-completions 兜底补丁，`openclaw-gateway.service` 已持久设置。
§
中转站诊断：排查 OpenClaw/Hermes 连接问题时优先测当前中转站 `/models` 和逐模型探活，区分 401/502/503；不要主动建议换 `baseUrl`，除非当前中转站整体不可用。
§
长研究任务用 cron 多轨道比 delegate_task 稳（后者易中断）。学习任务只生成报告不改代码。用户服务器技术栈：FastAPI + 原生HTML/JS/CSS，无现代前端框架。
§
服务器 `192.204.35.79`：OpenResty (Nginx) + MySQL:3306 + SSH:22，静态首页130字节。用户无SSH，仅远程操作。
§
GitHub: `git@github.com:2195573507-web/usserver.git` 存放服务器12个自建项目（monorepo）。第三方项目不上传：hermes-agent、openclaw、sub2api、upstream-hub-standalone。服务器SSH可直推GitHub。