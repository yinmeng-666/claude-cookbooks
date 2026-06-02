# Cloudflare demo — Self-Hosted Sandboxes (pure-Worker variant)

Same webhook → drain-queue → per-session runner shape as `../cf/`, but the runner is a **Durable Object running the TS `client.beta.sessions.events.toolRunner()`** (`src/runner.ts`) with an **in-isolate fake filesystem** (`src/tools.ts`) instead of a real container. `read`/`write`/`edit`/`glob`/`grep` operate on a `Map<string,string>` held in the DO; `bash` returns a not-available stub.

Idle policy is the SDK default: the dispatcher exits 60s after `session.status_idle` with `stop_reason: end_turn`; any other event resets the clock.

This is the TS library-usage reference for the lower-level `SessionToolRunner` (custom tools, the DO owns the heartbeat + force-stop). For a real shell — and for `EnvironmentWorker`, which composes the whole work-item lifecycle but pulls in the Node-only `agent-toolset/node` module that a Workers isolate can't run — use the Container or Vercel/Modal demos.

No org API key anywhere: the webhook polls with the environment key, and the runner DO authenticates with that same environment key — the single credential for both the control plane and the per-session calls.

```sh
npm i
wrangler secret put ANTHROPIC_WEBHOOK_SECRET
wrangler secret put ANTHROPIC_ENVIRONMENT_KEY
# edit ANTHROPIC_ENVIRONMENT_ID in wrangler.toml, then:
wrangler deploy
```

---

## 中文翻译

# Cloudflare 示例 —— 自托管沙箱（纯 Worker 变体）

与 `../cf/` 具有相同的 webhook → 清空队列 → 按会话运行器结构，但运行器是一个**运行 TS `client.beta.sessions.events.toolRunner()` 的 Durable Object**（`src/runner.ts`），并使用**isolate 内的伪文件系统**（`src/tools.ts`），而不是真实容器。`read`/`write`/`edit`/`glob`/`grep` 操作的是保存在 DO 中的 `Map&lt;string,string&gt;`；`bash` 返回一个不可用的桩实现。

空闲策略使用 SDK 默认值：调度器会在 `session.status_idle` 且 `stop_reason: end_turn` 之后 60 秒退出；任何其他事件都会重置计时。

这是更底层 `SessionToolRunner` 的 TS 库用法参考（自定义工具、由 DO 负责心跳和强制停止）。如果你需要真实 shell——以及 `EnvironmentWorker`（它能组合完整的工作项生命周期，但会引入 Node-only 的 `agent-toolset/node` 模块，而 Workers isolate 无法运行该模块）——请使用容器版或 Vercel/Modal 示例。

全程都不需要组织级 API key：webhook 使用环境 key 轮询，运行器 DO 也使用同一个环境 key 进行认证——这是控制平面和按会话调用共用的唯一凭证。

```sh
npm i
wrangler secret put ANTHROPIC_WEBHOOK_SECRET
wrangler secret put ANTHROPIC_ENVIRONMENT_KEY
# edit ANTHROPIC_ENVIRONMENT_ID in wrangler.toml, then:
wrangler deploy
```
