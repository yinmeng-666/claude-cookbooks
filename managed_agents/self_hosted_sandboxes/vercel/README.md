# Vercel demo — Self-Hosted Sandboxes

Same webhook → drain-queue → per-session runner shape as the Modal, Daytona,
and Cloudflare demos, but the runner is a **Vercel Sandbox running the TS
`EnvironmentWorker.handleItem()`** with the SDK's `betaAgentToolset20260401()` against the
sandbox's real filesystem (`bash` / `read` / `write` / `edit` / `glob` / `grep`).

The webhook function (`api/webhook.ts`) is a wake-up signal only. It verifies
the Standard Webhooks signature with `client.beta.webhooks.unwrap()`, drains
`work.poll()` until empty (capped at 25), and per item: acks, creates a Vercel
Sandbox, uploads `runner/runner.mjs`, and starts it detached. A bad work item is
logged and skipped — it stays un-acked and reclaims on the next webhook, so it
can't wedge the rest of the queue.

Idle policy is the SDK default: the dispatcher exits 60s after
`session.status_idle` with `stop_reason: end_turn`; any other event — including
`requires_action` idle, where the agent is blocked on the sandbox — resets the
clock.

No org API key anywhere: the webhook polls with the environment key, and the
runner authenticates with that same environment key — the single credential for
both the control plane and the per-session calls.

## Files

- `api/webhook.ts` — Vercel Function: verify sig → drain → spawn sandbox per item.
- `runner/runner.mjs` — runs inside the sandbox: `EnvironmentWorker.handleItem()` —
  builds the per-session `AgentToolContext`, downloads the agent's skills, runs a
  `SessionToolRunner` while heartbeating, force-stops the work item on exit.

## Prerequisites

- `npm`, the [Vercel CLI](https://vercel.com/docs/cli) (`npm i -g vercel`)
- A Vercel project linked (`vercel link`)
- A registered self-hosted environment + its environment key

## Configure

```sh
npm install
vercel link
vercel env add ANTHROPIC_WEBHOOK_SECRET   # placeholder for now
vercel env add ANTHROPIC_ENVIRONMENT_ID
vercel env add ANTHROPIC_ENVIRONMENT_KEY
```

## Deploy

```sh
vercel deploy --prod
```

This prints a `https://<project>.vercel.app` URL. Register
`https://<project>.vercel.app/api/webhook` as a webhook for
`session.status_run_started` in Console (or via the API), copy the issued
secret, then update:

```sh
vercel env rm ANTHROPIC_WEBHOOK_SECRET production
vercel env add ANTHROPIC_WEBHOOK_SECRET production
vercel deploy --prod
```

## Test

Create a session pointing at your environment id and send it a message:

```py
session = client.beta.sessions.create(agent=agent_id, environment_id=ENVIRONMENT_ID)
client.beta.sessions.events.send(session.id, events=[{"type": "user.message", "content": "ls -la"}])
```

You should see, in order:

```
vercel logs <project>.vercel.app
# [webhook] event=session.status_run_started session_id=...
# [webhook] acked work=... session=... sandbox=sbx_... (created)
```

Sandbox stdout/stderr (the `[runner]` lines) shows under the project's
**Observability → Sandboxes** tab in the Vercel dashboard.

## Notes

- `Sandbox.create()` + `writeFiles()` + `npm install` is ~15–25s before the
  function responds; `vercel.json` sets `maxDuration: 60`. The runner itself is
  `detached`. The function also mirrors the runner's first ~30s of output into
  the function log so it's debuggable from `vercel logs` without the Sandboxes
  observability tab.
- **Sandbox reuse** — Vercel Sandbox has no native tag/label/get-by-name API, so
  reuse needs external state. If a [Vercel KV](https://vercel.com/storage/kv)
  store is attached (`KV_REST_API_URL`/`KV_REST_API_TOKEN` are set), the webhook
  records `session_id → sandbox_id` and reuses a running sandbox on the next
  `run_started`, calling `extendTimeout()` so an active session keeps its VM
  alive across turns. Without KV, every delivery creates a fresh sandbox —
  `SessionToolRunner` dedups via `seen`/`answered` so a duplicate runner is
  wasteful, not wrong.
- **Working dirs** — the webhook creates `/mnt/session` and `/workspace` at
  sandbox start and uses `/workspace` as the dispatch workdir, matching the
  Modal/CF demos (skills download to `/workspace/skills/<name>/`). These are
  plain ephemeral directories, *not* persistent volumes — Vercel Sandbox has no
  volume API. The agent's working tree is gone when the sandbox stops.
- `npm install` runs inside the sandbox at spawn time so this demo has no build
  step. To shave ~10s off cold start, pre-bundle `runner/runner.mjs` with
  esbuild and upload the bundle instead.

---

## 中文翻译

# Vercel 示例 —— 自托管沙箱

与 Modal、Daytona 和 Cloudflare 示例具有相同的 webhook → 清空队列 → 按会话运行器结构，但运行器是一个**运行 TS `EnvironmentWorker.handleItem()` 的 Vercel Sandbox**，并通过 SDK 的 `betaAgentToolset20260401()` 操作沙箱的真实文件系统（`bash` / `read` / `write` / `edit` / `glob` / `grep`）。

webhook function（`api/webhook.ts`）仅作为唤醒信号。它使用 `client.beta.webhooks.unwrap()` 验证 Standard Webhooks 签名，循环调用 `work.poll()` 直到队列清空（上限 25），并对每个工作项执行：ack、创建 Vercel Sandbox、上传 `runner/runner.mjs`，然后以 detached 模式启动它。无效的工作项会被记录并跳过——它会保持未 ack 状态，并在下一次 webhook 时重新领取，因此不会卡住队列中的其他任务。

空闲策略使用 SDK 默认值：调度器会在 `session.status_idle` 且 `stop_reason: end_turn` 之后 60 秒退出；任何其他事件——包括 `requires_action` 状态下的 idle（此时 agent 正阻塞等待 sandbox）——都会重置计时。

全程都不需要组织级 API key：webhook 使用环境 key 轮询，运行器也使用同一个环境 key 进行认证——这是控制平面和按会话调用共用的唯一凭证。

## 文件

- `api/webhook.ts` —— Vercel Function：验证签名 → 清空队列 → 为每个工作项启动 sandbox。
- `runner/runner.mjs` —— 在 sandbox 内运行：`EnvironmentWorker.handleItem()` —— 构建按会话隔离的 `AgentToolContext`，下载 Agent 的 skills，在发送心跳的同时运行 `SessionToolRunner`，并在退出时对工作项执行强制停止。

## 前置要求

- `npm`，以及 [Vercel CLI](https://vercel.com/docs/cli)（`npm i -g vercel`）
- 已关联的 Vercel project（`vercel link`）
- 已注册的自托管 environment 及其 environment key

## 配置

```sh
npm install
vercel link
vercel env add ANTHROPIC_WEBHOOK_SECRET   # placeholder for now
vercel env add ANTHROPIC_ENVIRONMENT_ID
vercel env add ANTHROPIC_ENVIRONMENT_KEY
```

## 部署

```sh
vercel deploy --prod
```

这会输出一个 `https://&lt;project&gt;.vercel.app` URL。将 `https://&lt;project&gt;.vercel.app/api/webhook` 注册为 Console 中 `session.status_run_started` 的 webhook（或通过 API 注册），复制签发的 secret，然后更新：

```sh
vercel env rm ANTHROPIC_WEBHOOK_SECRET production
vercel env add ANTHROPIC_WEBHOOK_SECRET production
vercel deploy --prod
```

## 测试

创建一个指向你的 environment id 的会话，并向其发送一条消息：

```py
session = client.beta.sessions.create(agent=agent_id, environment_id=ENVIRONMENT_ID)
client.beta.sessions.events.send(session.id, events=[{"type": "user.message", "content": "ls -la"}])
```

你应当按顺序看到：

```
vercel logs &lt;project&gt;.vercel.app
# [webhook] event=session.status_run_started session_id=...
# [webhook] acked work=... session=... sandbox=sbx_... (created)
```

Sandbox stdout/stderr（即 `[runner]` 开头的行）可在 Vercel 控制台项目的 **Observability → Sandboxes** 标签页下查看。

## 说明

- `Sandbox.create()` + `writeFiles()` + `npm install` 需要约 15–25 秒，之后 function 才会响应；`vercel.json` 将 `maxDuration` 设为 `60`。运行器本身以 `detached` 模式启动。该 function 还会将运行器前约 30 秒的输出镜像到 function 日志中，因此即使不打开 Sandboxes observability 标签页，也可以通过 `vercel logs` 进行调试。
- **Sandbox 复用** —— Vercel Sandbox 没有原生的 tag/label/get-by-name API，因此复用需要外部状态。如果挂载了 [Vercel KV](https://vercel.com/storage/kv) 存储（即已设置 `KV_REST_API_URL`/`KV_REST_API_TOKEN`），webhook 会记录 `session_id → sandbox_id`，并在下一次 `run_started` 时复用仍在运行的 sandbox，同时调用 `extendTimeout()`，以便活跃会话能跨轮次保持其 VM 存活。若没有 KV，则每次投递都会创建一个全新的 sandbox——`SessionToolRunner` 会通过 `seen`/`answered` 去重，因此重复运行器只是浪费资源，不会导致错误。
- **工作目录** —— webhook 会在 sandbox 启动时创建 `/mnt/session` 和 `/workspace`，并使用 `/workspace` 作为调度工作目录，这与 Modal/CF 示例保持一致（skills 会下载到 `/workspace/skills/&lt;name&gt;/`）。这些只是普通的临时目录，*不是* 持久化 volume——Vercel Sandbox 没有 volume API。sandbox 停止后，Agent 的工作树也会消失。
- `npm install` 会在 sandbox 启动时于内部执行，因此此示例无需构建步骤。若想将冷启动时间缩短约 10 秒，可以预先使用 esbuild 打包 `runner/runner.mjs`，然后上传打包产物。
