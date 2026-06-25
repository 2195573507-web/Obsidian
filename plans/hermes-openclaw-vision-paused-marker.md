---
title: Hermes / OpenClaw 读图能力集成暂停标记
status: paused
created: 2026-06-25
updated: 2026-06-25
tags: [vision, openclaw, hermes, paused, todo]
related:
  - [[MOC|知识库导航]]
  - [[agents/planning/Hermes-OpenClaw-智能增强路线图|Hermes / OpenClaw 智能增强路线图]]
---

# Hermes / OpenClaw 读图能力集成暂停标记

## 当前状态

用户要求：先不继续做读图能力集成，先检查 OpenClaw 当前状态，并留下后续继续的记号。

时间：2026-06-25 16:34 UTC。

## 已完成的事实

1. 当前本地 OpenAI-compatible 代理：`http://127.0.0.1:18888/v1`。
2. 已验证 `myproxy/gpt-5.5` 支持图片输入。
3. 已验证 `myproxy/gpt-5.4-mini` 也支持图片输入。
4. 已给 OpenClaw 配置里的以下模型标注 `input: ["text", "image"]`：
   - `gpt-5.5`
   - `gpt-5.4-mini`
5. 已配置 Hermes `auxiliary.vision` 指向：
   - provider: `openai`
   - model: `gpt-5.5`
   - base_url: `http://127.0.0.1:18888/v1`
6. 已部署共享图片分析服务：
   - systemd: `agent-image-analyzer.service`
   - 地址：`http://127.0.0.1:9501`
   - 健康检查：`GET /health`
   - 识图接口：`POST /api/image/analyze`
   - OCR 接口：`POST /api/image/ocr`

## 验证结果

测试图片：`/tmp/vision-test.png`，内容是红色矩形框和文字 `TEST 123`。

### OpenClaw CLI 验证

命令：

```bash
openclaw infer image describe \
  --file /tmp/vision-test.png \
  --model myproxy/gpt-5.5 \
  --prompt '请读取图片中文字和主要形状' \
  --json
```

结果：成功识别 `TEST123` 和红色矩形边框。

### Hermes vision 工具验证

`vision_analyze('/tmp/vision-test.png')` 成功识别图片中文字 `TEST 123` 和红色矩形边框。

### 共享 image-analyzer 验证

命令：

```bash
curl -fsS http://127.0.0.1:9501/health
```

结果：

```json
{"ok":true,"provider":"myproxy","base_url":"http://127.0.0.1:18888/v1","default_model":"gpt-5.5"}
```

接口测试成功返回图片描述。

## 当前 OpenClaw 状态

检查结果：

- `openclaw-gateway.service`：active/running。
- `/health`：`{"ok":true,"status":"live"}`。
- 端口：`127.0.0.1:18789` 正常监听。
- QQBot：日志显示：
  - `Access token obtained successfully`
  - `WebSocket connected`
  - `Gateway ready`
- 最近有正常模型调用：`myproxy/gpt-5.5 status=200`。

注意：重启前旧日志里出现过 `token_missing_config / unauthorized`，但重启完成后 QQBot 已重新 ready，目前 OpenClaw 网关健康。

## 后续继续时要做

1. 不要重复做已完成的模型图片能力验证。
2. 先确认用户是否要继续：
   - 只保留 Hermes 读图；
   - 只保留 OpenClaw CLI/内部能力；
   - 进一步做 QQ 图片自动识别；
   - 接入 Dashboard/Obsidian 记录。
3. 如果继续做 QQ 图片自动识别，需要重点排查 QQBot 插件对入站图片的事件结构：
   - 图片 URL 是否进入 event metadata；
   - 是否已下载到本地 inbound media path；
   - OpenClaw 是否会自动把 image attachments 传给 media-understanding runtime。
4. 如果不继续做，保留现状即可：OpenClaw/Hermes 配置已支持读图，image-analyzer 服务已运行。

## 回滚参考

自动备份：

- `/root/.openclaw/openclaw.json.bak.vision-20260625-162158`
- `/root/.hermes/config.yaml.bak.vision-20260625-162158`

如果要回滚读图配置：

```bash
cp /root/.openclaw/openclaw.json.bak.vision-20260625-162158 /root/.openclaw/openclaw.json
cp /root/.hermes/config.yaml.bak.vision-20260625-162158 /root/.hermes/config.yaml
systemctl restart openclaw-gateway
systemctl restart hermes-gateway
```

如果要停用共享图片分析服务：

```bash
systemctl disable --now agent-image-analyzer.service
```
