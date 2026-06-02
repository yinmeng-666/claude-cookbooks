# Daytona demo — Self-Hosted Sandboxes

Reference implementation of the [usage guide](../docs/usage-guide.md) on [Daytona](https://www.daytona.io/). `daytona_webhook.py` is a FastAPI app that handles the `session.status_run_started` webhook (verified with `client.beta.webhooks.unwrap()`), **drains the environment work queue** with `client.beta.environments.work.poller(drain=True, auto_stop=False)` so any single delivery recovers earlier missed ones, and per item creates a Daytona sandbox, uploads the **same provider-agnostic `sandbox_runner.py`** the Modal demo uses, and starts it. Daytona sandboxes are full Linux containers, so `beta_agent_toolset_20260401` (bash/read/write/edit/glob/grep) works as-is.

No org API key reaches the runner: the webhook polls with the environment key, and each sandbox authenticates with that same environment key — the single credential for both the control plane and the per-session calls.

```sh
# standardwebhooks backs `client.beta.webhooks.unwrap()` — only the orchestrator
# host needs it; the inner Daytona sandbox never sees raw webhook deliveries.
pip install fastapi uvicorn daytona-sdk standardwebhooks anthropic

export DAYTONA_API_KEY=... DAYTONA_API_URL=...
export ANTHROPIC_WEBHOOK_SECRET=... \
       ANTHROPIC_ENVIRONMENT_ID=env_... ANTHROPIC_ENVIRONMENT_KEY=sk-ant-oat...

uvicorn daytona_webhook:app --host 0.0.0.0 --port 8080
```

Deploy the FastAPI app anywhere that can serve HTTP and reach the Daytona API (Fly, Render, a VM behind a tunnel, etc.), then register its URL as the webhook endpoint.

> **Cold-start note:** `_spawn()` runs `pip install anthropic` inside each fresh Daytona sandbox, which adds ~10–15s before the runner starts. For production, pre-bake the SDK into a custom Daytona image and drop the `pip install` line.

---

## 中文翻译

# Daytona 示例 —— 自托管沙箱

这是在 [Daytona](https://www.daytona.io/) 上实现 [使用指南](../docs/usage-guide.md) 的参考实现。`daytona_webhook.py` 是一个 FastAPI 应用，用于处理 `session.status_run_started` webhook（使用 `client.beta.webhooks.unwrap()` 验证），并通过 `client.beta.environments.work.poller(drain=True, auto_stop=False)` **清空环境工作队列**，从而让任意一次投递都能恢复此前遗漏的任务；对每个工作项，它会创建一个 Daytona 沙箱，上传与 Modal 示例使用的**同一个与提供方无关的 `sandbox_runner.py`**，然后启动它。Daytona 沙箱是完整的 Linux 容器，因此 `beta_agent_toolset_20260401`（bash/read/write/edit/glob/grep）可以直接使用。

组织级 API key 不会到达运行器：webhook 使用环境 key 轮询，每个沙箱也使用同一个环境 key 进行认证——这是控制平面和按会话调用共用的唯一凭证。

```sh
# standardwebhooks backs `client.beta.webhooks.unwrap()` — only the orchestrator
# host needs it; the inner Daytona sandbox never sees raw webhook deliveries.
pip install fastapi uvicorn daytona-sdk standardwebhooks anthropic

export DAYTONA_API_KEY=... DAYTONA_API_URL=...
export ANTHROPIC_WEBHOOK_SECRET=... \
       ANTHROPIC_ENVIRONMENT_ID=env_... ANTHROPIC_ENVIRONMENT_KEY=sk-ant-oat...

uvicorn daytona_webhook:app --host 0.0.0.0 --port 8080
```

将这个 FastAPI 应用部署到任意能够提供 HTTP 服务并访问 Daytona API 的地方（Fly、Render、通过隧道暴露的 VM 等），然后把它的 URL 注册为 webhook 端点。

&gt; **冷启动说明：** `_spawn()` 会在每个全新的 Daytona 沙箱内执行 `pip install anthropic`，这会在运行器启动前增加大约 10–15 秒。生产环境中，建议将 SDK 预先打包进自定义 Daytona 镜像，并删除 `pip install` 这一行。
