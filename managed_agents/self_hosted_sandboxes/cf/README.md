# Cloudflare demo — Self-Hosted Sandboxes (Container variant)

The Worker (`src/index.ts`) verifies the `session.status_run_started` webhook with `client.beta.webhooks.unwrap()`, then **drains the environment work queue** (poll → ack until empty) so any single delivery recovers earlier missed ones. Per work item it starts a per-session **Cloudflare Container** (`src/container.ts`) whose entrypoint is `ant beta:worker run` — the CLI handles heartbeat, backlog reconcile, SSE, the default tool set (`bash`/`read`/`write`/`edit`/`glob`/`grep`), and the work-item force-stop on exit.

The CLI owns the idle policy (`--max-idle 60s` after `session.status_idle` with `stop_reason: end_turn`; any other event resets the clock). The DO owns the *Cloudflare* container lifetime: it streams session status to renew `sleepAfter` so CF doesn't reclaim the VM out from under a live `ant beta:worker run`.

No org API key reaches the runner: the container authenticates with the environment key — the single credential `ant beta:worker run` uses for the event stream, the lease heartbeat, and the work-item force-stop.

See `../cf-worker/` for the pure-Worker variant that runs the TS `SessionToolRunner` with an in-isolate fake filesystem instead of a real container.

```sh
npm i
wrangler secret put ANTHROPIC_WEBHOOK_SECRET
wrangler secret put ANTHROPIC_ENVIRONMENT_KEY
# edit ANTHROPIC_ENVIRONMENT_ID in wrangler.toml, then:
wrangler deploy
```

---

## 中文翻译

# Cloudflare 示例 —— 自托管沙箱（容器变体）

Worker（`src/index.ts`）使用 `client.beta.webhooks.unwrap()` 验证 `session.status_run_started` webhook，然后**清空环境工作队列**（poll → ack，直到为空），这样任意一次投递都能恢复此前遗漏的任务。对于每个工作项，它会启动一个按会话隔离的**Cloudflare Container**（`src/container.ts`），其入口点是 `ant beta:worker run`——CLI 负责心跳、积压任务对账、SSE、默认工具集（`bash`/`read`/`write`/`edit`/`glob`/`grep`），以及退出时对工作项执行强制停止。

CLI 负责空闲策略（在 `session.status_idle` 且 `stop_reason: end_turn` 之后，`--max-idle 60s`；任何其他事件都会重置计时）。DO 负责 *Cloudflare* 容器的生命周期：它会流式跟踪会话状态以续期 `sleepAfter`，避免 CF 在仍有活动的 `ant beta:worker run` 运行时回收底层 VM。

组织级 API key 不会到达运行器：容器使用环境 key 进行认证——这是 `ant beta:worker run` 用于事件流、租约心跳和工作项强制停止的唯一凭证。

参见 `../cf-worker/` 获取纯 Worker 变体：它运行 TS `SessionToolRunner`，并使用 isolate 内的伪文件系统，而不是实际容器。

```sh
npm i
wrangler secret put ANTHROPIC_WEBHOOK_SECRET
wrangler secret put ANTHROPIC_ENVIRONMENT_KEY
# edit ANTHROPIC_ENVIRONMENT_ID in wrangler.toml, then:
wrangler deploy
```
