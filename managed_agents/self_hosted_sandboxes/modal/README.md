# Modal demo — Self-Hosted Sandboxes

Reference implementation of the [usage guide](../docs/usage-guide.md)'s webhook flow on [Modal](https://modal.com). Two files:

- `modal_sandbox_webhook.py` — Modal app that receives the `session.status_run_started` webhook, verifies it with `client.beta.webhooks.unwrap()`, drains the environment work queue with `client.beta.environments.work.poller(drain=True, auto_stop=False)`, and spins up a per-session Modal Sandbox per item. A per-session `modal.Volume` is mounted at `/workspace` so the agent's working tree and downloaded skills persist across sandbox restarts for the same session. The sandbox env vars use the same `ANTHROPIC_*` contract as `ant beta:worker poll --on-work`.
- `sandbox_runner.py` — runs inside that Sandbox: `client.beta.environments.work.worker(environment_key=..., workdir="/workspace", unrestricted_paths=True).handle_item()`. It reads the `ANTHROPIC_*` env vars, builds the per-session `AgentToolContext` and downloads the agent's skills into `/workspace/skills/<name>/`, runs a `SessionToolRunner` (heartbeat + reconcile + event stream + `bash`/`read`/`write`/`edit`/`glob`/`grep` dispatch + result posting), and force-stops the work item on exit. Idle policy is the SDK default: it exits 60s after `session.status_idle` with `stop_reason: end_turn`; any other event resets the clock.

No org API key anywhere: the webhook polls with the environment key, and the runner authenticates with that same environment key — the single credential for both the control plane and the per-session calls.

## Prerequisites

```shell
pip install modal
modal setup   # auth to your Modal workspace
```

## Configure

```shell
modal secret create cma-self-hosted-sandboxes-secrets \
    ANTHROPIC_WEBHOOK_SECRET=placeholder \
    ANTHROPIC_ENVIRONMENT_ID='env_...' \
    ANTHROPIC_ENVIRONMENT_KEY='sk-ant-oat...'
```

## Deploy

```shell
modal deploy modal_sandbox_webhook.py
```

This prints a `*.modal.run` URL. Register that URL as a webhook for `session.status_run_started` in Console (or via the API), copy the issued secret, then update:

```shell
modal secret create cma-self-hosted-sandboxes-secrets \
    ANTHROPIC_WEBHOOK_SECRET='whsec_...' \
    ANTHROPIC_ENVIRONMENT_ID='env_...' \
    ANTHROPIC_ENVIRONMENT_KEY='sk-ant-oat...' \
    --force
```

(no redeploy needed — secrets are read at container start.)

## Test

Create a session pointing at your environment id and send it a message:

```py
session = client.beta.sessions.create(agent=agent_id, environment_id=ENVIRONMENT_ID)
client.beta.sessions.events.send(session.id, events=[{"type": "user.message", "content": "ls -la"}])
```

You should see, in order:

```shell
modal app logs cma-self-hosted-sandboxes
# [webhook] event=session.status_run_started session_id=...
# [webhook] acked work=... session=... sandbox=sb-... (created)
```

Sandbox stdout (the `[runner]` lines) shows in the Modal dashboard under
**Apps → cma-self-hosted-sandboxes → Sandboxes**.

## Iterating

Editing either Python file requires a redeploy (`modal deploy ...`). Editing only secrets does not. To force a clean slate while iterating, `modal app stop cma-self-hosted-sandboxes` before redeploying.

---

## 中文翻译

# Modal 示例 —— 自托管沙箱

这是在 [Modal](https://modal.com) 上实现 [使用指南](../docs/usage-guide.md) webhook 流程的参考实现。包含两个文件：

- `modal_sandbox_webhook.py` —— Modal 应用，接收 `session.status_run_started` webhook，使用 `client.beta.webhooks.unwrap()` 验证，通过 `client.beta.environments.work.poller(drain=True, auto_stop=False)` 清空环境工作队列，并为每个工作项启动一个按会话隔离的 Modal Sandbox。一个按会话隔离的 `modal.Volume` 会挂载到 `/workspace`，以便 Agent 的工作树和已下载的 skills 能在同一会话的沙箱重启后继续保留。沙箱环境变量使用与 `ant beta:worker poll --on-work` 相同的 `ANTHROPIC_*` 契约。
- `sandbox_runner.py` —— 在该 Sandbox 内运行：`client.beta.environments.work.worker(environment_key=..., workdir="/workspace", unrestricted_paths=True).handle_item()`。它读取 `ANTHROPIC_*` 环境变量，构建按会话隔离的 `AgentToolContext`，并将 Agent 的 skills 下载到 `/workspace/skills/&lt;name&gt;/`，运行一个 `SessionToolRunner`（负责心跳 + 对账 + 事件流 + `bash`/`read`/`write`/`edit`/`glob`/`grep` 分发 + 结果回传），并在退出时对工作项执行强制停止。空闲策略使用 SDK 默认值：它会在 `session.status_idle` 且 `stop_reason: end_turn` 之后 60 秒退出；任何其他事件都会重置计时。

全程都不需要组织级 API key：webhook 使用环境 key 轮询，运行器也使用同一个环境 key 进行认证——这是控制平面和按会话调用共用的唯一凭证。

## 前置要求

```shell
pip install modal
modal setup   # auth to your Modal workspace
```

## 配置

```shell
modal secret create cma-self-hosted-sandboxes-secrets \
    ANTHROPIC_WEBHOOK_SECRET=placeholder \
    ANTHROPIC_ENVIRONMENT_ID='env_...' \
    ANTHROPIC_ENVIRONMENT_KEY='sk-ant-oat...'
```

## 部署

```shell
modal deploy modal_sandbox_webhook.py
```

这会输出一个 `*.modal.run` URL。将该 URL 注册为 Console 中 `session.status_run_started` 的 webhook（或通过 API 注册），复制签发的 secret，然后更新：

```shell
modal secret create cma-self-hosted-sandboxes-secrets \
    ANTHROPIC_WEBHOOK_SECRET='whsec_...' \
    ANTHROPIC_ENVIRONMENT_ID='env_...' \
    ANTHROPIC_ENVIRONMENT_KEY='sk-ant-oat...' \
    --force
```

（无需重新部署——secrets 会在容器启动时读取。）

## 测试

创建一个指向你的 environment id 的会话，并向其发送一条消息：

```py
session = client.beta.sessions.create(agent=agent_id, environment_id=ENVIRONMENT_ID)
client.beta.sessions.events.send(session.id, events=[{"type": "user.message", "content": "ls -la"}])
```

你应当按顺序看到：

```shell
modal app logs cma-self-hosted-sandboxes
# [webhook] event=session.status_run_started session_id=...
# [webhook] acked work=... session=... sandbox=sb-... (created)
```

Sandbox stdout（即 `[runner]` 开头的行）可在 Modal 控制台的 **Apps → cma-self-hosted-sandboxes → Sandboxes** 中查看。

## 迭代开发

修改任一 Python 文件后都需要重新部署（`modal deploy ...`）。如果只修改 secrets，则不需要。若想在迭代时强制获得一个干净状态，请在重新部署前执行 `modal app stop cma-self-hosted-sandboxes`。
