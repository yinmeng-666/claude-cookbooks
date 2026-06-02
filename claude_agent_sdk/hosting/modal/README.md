# Tier 2 — Modal

Runs the **same** `hosting/Dockerfile` image on [Modal](https://modal.com) via
`modal.Sandbox`. You get a public HTTPS URL, scale-to-zero, and no
infrastructure to manage.

## Prerequisites

```bash
pip install modal
modal setup            # opens a browser to authenticate
modal secret create anthropic ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY"
```

## Deploy

```bash
cd claude_agent_sdk/
python hosting/modal/modal_app.py
```

This builds the Dockerfile remotely on Modal (build context = `claude_agent_sdk/`,
same as local), starts a sandbox running `serve`, and prints a tunnel URL.

## Talk to it

The deploy script prints a `url:` and a `token:` line. The tunnel URL is public,
so the server requires the bearer token on the messages endpoint (only
`/health` is open):

```bash
curl -N -X POST "$MODAL_URL/sessions/demo-1/messages" \
  -H "Authorization: Bearer $MODAL_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"What are the latest AI agent trends?"}'
```

## Persistence

By default `modal_app.py` mounts a `modal.Volume` at `/data` — the same
`CLAUDE_CONFIG_DIR` trick as tier 1. Session transcripts survive sandbox
restarts.

> **Note:** `modal.Volume` has explicit commit semantics. If your workload has
> many sandboxes writing many sessions concurrently and you see lost writes,
> switch to a [`SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage)
> backed by Modal `Dict` or an external store — that's also what tier 3 uses.

## Liveness

Modal keeps the sandbox running until the `timeout=` passed to `Sandbox.create`
expires; it does not probe `GET /health`. If the server inside dies, the sandbox
stays up and requests fail until the timeout reaps it. For anything longer-lived
than a demo, point your own monitoring at `/health` and recreate the sandbox
when it stops answering.

## Teardown

```bash
python hosting/modal/teardown.py
```

Stops sandboxes and removes the volume so you're not billed for idle resources.

---

## 中文翻译

# 第 2 层 — Modal

通过 `modal.Sandbox` 在 [Modal](https://modal.com) 上运行**同一个** `hosting/Dockerfile` 镜像。你将获得一个公开的 HTTPS URL、scale-to-zero，以及无需自行管理的基础设施。

## 前置条件

```bash
pip install modal
modal setup            # 打开浏览器进行认证
modal secret create anthropic ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY"
```

## 部署

```bash
cd claude_agent_sdk/
python hosting/modal/modal_app.py
```

这会在 Modal 上远程构建 Dockerfile（build context = `claude_agent_sdk/`，与本地一致），启动一个运行 `serve` 的 sandbox，并打印出一个 tunnel URL。

## 如何与其交互

部署脚本会打印一行 `url:` 和一行 `token:`。该 tunnel URL 是公开的，因此 server 会在 messages endpoint 上要求 bearer token（只有 `/health` 是开放的）：

```bash
curl -N -X POST "$MODAL_URL/sessions/demo-1/messages" \
  -H "Authorization: Bearer $MODAL_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"What are the latest AI agent trends?"}'
```

## 持久化

默认情况下，`modal_app.py` 会在 `/data` 挂载一个 `modal.Volume`——与第 1 层相同，使用 `CLAUDE_CONFIG_DIR` 这套方式。会话记录在 sandbox 重启后仍可保留。

> **注意：** `modal.Volume` 具有显式提交语义。如果你的工作负载中有很多 sandbox 并发写入许多 sessions，且你观察到写入丢失，请切换为由 Modal `Dict` 或外部存储支持的 [`SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage)——这也是第 3 层所使用的方式。

## 存活性

Modal 会让 sandbox 一直运行，直到传给 `Sandbox.create` 的 `timeout=` 到期；它不会主动探测 `GET /health`。如果内部 server 挂掉，sandbox 仍会保持运行，而请求会持续失败，直到 timeout 将其回收。对于任何比演示更长生命周期的场景，请使用你自己的监控探测 `/health`，并在它不再响应时重新创建 sandbox。

## 清理

```bash
python hosting/modal/teardown.py
```

这会停止 sandboxes 并移除 volume，以避免你为闲置资源持续计费。
