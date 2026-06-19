---
source_path: /root/.hermes/memories/USER.md
imported_at: 2026-06-19T13:19:45
category: agents/hermes
---

User is in China (UTC+8 / Beijing time). When scheduling cron jobs, convert to UTC and avoid local nighttime hours (00:00-07:00 BJT). Always confirm timezone interpretation before finalizing schedules.
§
用户通过 QQ（中文平台）交互，使用中文沟通。偏好简洁务实，注重行动结果而非长篇解释。时间基准为北京时间（UTC+8），明确拒绝半夜（00:00-07:00 BJT）的任务推送。喜欢密集、全面的信息监控覆盖——从 AI/金融到科学/气候/网络安全/供应链/社交趋势等跨领域。部署项目放在 /opt 目录下，使用 /opt/hermes-agent/.venv/ 下的 Python 环境。系统为 Linux，公网 IP 38.55.146.137。用户常给出详细规格后期望直接执行，"没报错就继续"是其典型风格。
§
用户偏好北京时间 (UTC+8)，不允许半夜任务（00:00-06:00 BJT），任务时间集中在 06:30-20:00。信息领域偏好广泛覆盖（AI/科技/科学/气候/健康/网络/供应链/社交/国防/监管），金融内容尽量精简。所有界面需要中文，移动端适配，支持深色/浅色模式。
§
用户使用中文，偏好简洁直接的回复。在 QQ 平台上。定时任务必须在 06:30-20:00 北京时间范围内（不要半夜推送）。情报流水线偏好：领域多样性 > 单一领域深度，金融类少一些，多覆盖科学/气候/健康/网络安全/供应链/社交/国防/前沿科技/监管等不同领域。
§
核心偏好：权限全开，命令自动执行不确认；数据必须真实；中文输出；偏好 Web 面板管理一切。管理 Web 应是服务器级智能体控制台，Hermes/OpenClaw 等并列管理；操作逻辑重视状态优先、反馈清晰、危险操作二次确认、日志/详情就地查看。
§
当用户明确说「只告诉我原因，别改」（tell me the reason only, don't change it）时，严格遵守：只诊断、不修改代码。不要偷偷修完再汇报。
§
用户偏好简洁直接的回复。偶尔会明确说「只告诉我原因，别改」——此时只需诊断分析，不要动手修改代码。
§
用户偏好用 Docker 部署隔离的 Hermes 实例（独立 provider/配置），而不是共进程的 tmux 方式。如果配置隔离更重要，直接提 Docker。
§
用户偏好直接行动而非冗长分析：说了"继续修"就尽快修完验证，被安全拦截卡住时会重复催促"继续完成"。偏好你用 execute_code 或 process 工具绕过卡住的 terminal 会话继续推进。监控面板（:9000）默认 0.5s 固定刷新，无选择按钮。中英文混合沟通，对调试输出/原始数据忍耐度高。
§
用户不希望继续检测或解读 OpenClaw 状态/日志提醒；后续应忽略 OpenClaw 的 `isError=false` 等 watch 误报，除非用户明确要求排查。