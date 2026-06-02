# Self-Hosted Sandboxes

Reference implementations for running Claude Managed Agents sessions against
**self-hosted execution sandboxes**. Each variant implements the same contract
on a different compute provider:

1. Receive the `session.status_run_started` webhook (verified with
   `client.beta.webhooks.unwrap()`).
2. Drain the environment work queue so a single delivery recovers any earlier
   missed items.
3. Per work item, launch a per-session sandbox that runs the SDK/CLI tool
   runner (`bash`/`read`/`write`/`edit`/`glob`/`grep`), heartbeats the lease,
   and posts `tool_result`s back to the session.

No org API key reaches the runner — the sandbox authenticates with the
**environment key**, the single credential for both the control plane and the
per-session calls.

| Variant | Compute | Runner |
|---|---|---|
| [`docker/`](docker/) | Plain Docker on a host you control | `ant beta:worker run` in a per-session container |
| [`cf/`](cf/) | Cloudflare Containers | `ant beta:worker run` in a per-session Cloudflare Container |
| [`cf-worker/`](cf-worker/) | Cloudflare Workers (no container) | TS `SessionToolRunner` in a Durable Object with an in-isolate fake filesystem |
| [`modal/`](modal/) | [Modal](https://modal.com) | Python `sandbox_runner.py` in a Modal Sandbox with a per-session Volume |
| [`daytona/`](daytona/) | [Daytona](https://www.daytona.io/) | Same `sandbox_runner.py` uploaded to a Daytona sandbox |
| [`vercel/`](vercel/) | Vercel Functions + Sandbox | Node `runner.mjs` in a Vercel Sandbox |

## Getting started

See [`docs/usage-guide.md`](docs/usage-guide.md) for the full flow: creating a
self-hosted environment, registering the webhook, and wiring up the
environment key. Each variant's `README.md` covers its provider-specific
deploy steps.

See [`docs/upgrade-guide.md`](docs/upgrade-guide.md) for migrating between SDK
versions.

---

## 中文翻译

# 自托管沙箱

用于让 Claude Managed Agents 会话运行在**自托管执行沙箱**上的参考实现。每个变体都在不同的计算提供方上实现了相同的契约：

1. 接收 `session.status_run_started` webhook（使用 `client.beta.webhooks.unwrap()` 验证）。
2. 清空环境工作队列，以便单次投递也能恢复任何先前遗漏的项目。
3. 对每个工作项，启动一个按会话隔离的沙箱，运行 SDK/CLI 工具执行器（`bash`/`read`/`write`/`edit`/`glob`/`grep`），为租约发送心跳，并将 `tool_result` 回传到会话。

组织级 API key 不会到达运行器——沙箱使用**环境 key**进行认证，这是控制平面和按会话调用共用的唯一凭证。

| 变体 | 计算环境 | 运行器 |
|---|---|---|
| [`docker/`](docker/) | 你可控主机上的原生 Docker | 在按会话容器中运行 `ant beta:worker run` |
| [`cf/`](cf/) | Cloudflare Containers | 在按会话 Cloudflare Container 中运行 `ant beta:worker run` |
| [`cf-worker/`](cf-worker/) | Cloudflare Workers（无容器） | 在 Durable Object 中运行 TS `SessionToolRunner`，并使用 isolate 内的伪文件系统 |
| [`modal/`](modal/) | [Modal](https://modal.com) | 在带按会话 Volume 的 Modal Sandbox 中运行 Python `sandbox_runner.py` |
| [`daytona/`](daytona/) | [Daytona](https://www.daytona.io/) | 将相同的 `sandbox_runner.py` 上传到 Daytona 沙箱中运行 |
| [`vercel/`](vercel/) | Vercel Functions + Sandbox | 在 Vercel Sandbox 中运行 Node `runner.mjs` |

## 开始使用

完整流程请参见 [`docs/usage-guide.md`](docs/usage-guide.md)：创建自托管环境、注册 webhook，以及接入环境 key。每个变体目录下的 `README.md` 都说明了对应提供方的部署步骤。

有关在 SDK 版本之间迁移的说明，请参见 [`docs/upgrade-guide.md`](docs/upgrade-guide.md)。
