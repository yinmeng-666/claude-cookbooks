# Docker demo — Self-Hosted Sandboxes

The host runs `ant beta:worker poll` directly; per claimed work item its
`--on-work` script (`on-work.sh`) `docker run`s a per-session container whose
entrypoint is `ant beta:worker run`. Each container gets a `/workspace` (the
agent's working tree; skills download here) backed by a per-session Docker
volume so the tree and skills survive across containers for one session.

This is the no-cloud variant of `../cf/` (which runs the same
`ant beta:worker run` entrypoint, but in Cloudflare Containers): same CLI, same
env contract, just plain Docker on a host you control.

- **`Dockerfile`** — the per-session image: pinned `ant` CLI, `WORKDIR
  /workspace`, `ENTRYPOINT ["ant","beta:worker","run", …]`. The CLI owns
  heartbeat, backlog reconcile, SSE, the `bash`/`read`/`write`/`edit`/`glob`/
  `grep` toolset, and the work-item force-stop on exit.
- **`on-work.sh`** — invoked by the poller per work item. Reads the
  `ANTHROPIC_{WORK_ID,ENVIRONMENT_ID,SESSION_ID,ENVIRONMENT_KEY}` the poller
  sets, drains the work JSON on stdin, and starts a detached `--rm`
  per-session container (idempotent: a duplicate item for a live session is a
  no-op). Exits immediately so the poller claims the next item.
- **`start.sh`** — builds the image and execs the host poller with
  `--on-work on-work.sh`.

Idle policy is the SDK default: each container exits 60s after
`session.status_idle` with `stop_reason: end_turn`; any other event resets the
clock. `--rm` then removes it; the per-session volume persists for the next
message in that session.

No org API key anywhere: the only credential is the **environment key**. The
poller uses it to claim work; it's passed into each container as
`ANTHROPIC_ENVIRONMENT_KEY` (the per-session calls — event stream, lease
heartbeat, force-stop) **and** as `ANTHROPIC_AUTH_TOKEN` (the CLI's
skill-download client resolves only `ANTHROPIC_API_KEY` /
`ANTHROPIC_AUTH_TOKEN`, not `ANTHROPIC_ENVIRONMENT_KEY` — without it skills
silently fail to download).

## Prerequisites

- Docker
- `ant` on the host's PATH, the **same build** pinned in `Dockerfile`
  (`ARG ANT_VERSION`). Install it:

  ```sh
  VERSION=1.9.0
  OS=$(uname -s | tr '[:upper:]' '[:lower:]')
  ARCH=$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/')
  curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
    | sudo tar -xz -C /usr/local/bin ant
  ant --version
  ```

## Run

```sh
export ANTHROPIC_ENVIRONMENT_ID=env_...
export ANTHROPIC_ENVIRONMENT_KEY=sk-ant-oat...
./start.sh
```

Then create a session pointing at that `ANTHROPIC_ENVIRONMENT_ID` and send it a
message. You should see, in order:

```
[start] polling env=env_... base=https://api.anthropic.com
[on-work] session=sesn_... work=work_... container=... (started)
```

Container logs (`docker logs cma-sesn_...`) show the `ant beta:worker run`
JSON: `downloaded skill …`, `executing tool …`, etc. The container removes
itself after it idles out; the `cma-ws-<session_id>` volume remains for that
session's next message (`docker volume rm` to discard).

Unlike the webhook-driven demos (Modal/Daytona/Vercel/Cloudflare), there is no
webhook here — `ant beta:worker poll` long-polls the environment directly, so
nothing needs to be exposed to the internet.

---

## 中文翻译

# Docker 示例 —— 自托管沙箱

宿主机直接运行 `ant beta:worker poll`；对于每个已领取的工作项，它的 `--on-work` 脚本（`on-work.sh`）会执行 `docker run`，启动一个按会话隔离的容器，其入口点为 `ant beta:worker run`。每个容器都会获得一个 `/workspace`（Agent 的工作树，skills 会下载到这里），它由一个按会话隔离的 Docker volume 支撑，因此该工作树和 skills 可以在同一会话的多个容器之间保留。

这是 `../cf/` 的无云版本（后者在 Cloudflare Containers 中运行相同的 `ant beta:worker run` 入口点）：相同的 CLI，相同的环境契约，只是改为运行在你可控主机上的原生 Docker。

- **`Dockerfile`** —— 按会话镜像：固定版本的 `ant` CLI、`WORKDIR /workspace`、`ENTRYPOINT ["ant","beta:worker","run", …]`。CLI 负责心跳、积压任务对账、SSE、`bash`/`read`/`write`/`edit`/`glob`/`grep` 工具集，以及退出时对工作项执行强制停止。
- **`on-work.sh`** —— 由 poller 针对每个工作项调用。读取 poller 设置的 `ANTHROPIC_{WORK_ID,ENVIRONMENT_ID,SESSION_ID,ENVIRONMENT_KEY}`，从 stdin 读取工作 JSON，并启动一个后台运行、带 `--rm` 的按会话容器（具备幂等性：如果同一活跃会话收到重复工作项，则为空操作）。它会立即退出，以便 poller 继续领取下一个工作项。
- **`start.sh`** —— 构建镜像，并以 `--on-work on-work.sh` 方式执行宿主机 poller。

空闲策略使用 SDK 默认值：每个容器会在 `session.status_idle` 且 `stop_reason: end_turn` 之后 60 秒退出；任何其他事件都会重置计时。随后 `--rm` 会删除容器；而按会话 volume 会保留，以供该会话的下一条消息使用。

全程都不需要组织级 API key：唯一凭证是**环境 key**。poller 使用它领取工作；它会以 `ANTHROPIC_ENVIRONMENT_KEY` 的形式传入每个容器（供按会话调用——事件流、租约心跳、强制停止——使用），**同时**也会以 `ANTHROPIC_AUTH_TOKEN` 的形式传入（CLI 的 skill 下载客户端只解析 `ANTHROPIC_API_KEY` / `ANTHROPIC_AUTH_TOKEN`，不会解析 `ANTHROPIC_ENVIRONMENT_KEY`——如果不这样设置，skills 会静默下载失败）。

## 前置要求

- Docker
- 宿主机 PATH 中可用的 `ant`，且必须与 `Dockerfile` 中固定的**同一构建版本**（`ARG ANT_VERSION`）。安装方式如下：

  ```sh
  VERSION=1.9.0
  OS=$(uname -s | tr '[:upper:]' '[:lower:]')
  ARCH=$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/')
  curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
    | sudo tar -xz -C /usr/local/bin ant
  ant --version
  ```

## 运行

```sh
export ANTHROPIC_ENVIRONMENT_ID=env_...
export ANTHROPIC_ENVIRONMENT_KEY=sk-ant-oat...
./start.sh
```

然后创建一个指向该 `ANTHROPIC_ENVIRONMENT_ID` 的会话，并向它发送一条消息。你应当按顺序看到：

```
[start] polling env=env_... base=https://api.anthropic.com
[on-work] session=sesn_... work=work_... container=... (started)
```

容器日志（`docker logs cma-sesn_...`）会显示 `ant beta:worker run` 的 JSON 输出：`downloaded skill …`、`executing tool …` 等。容器在空闲超时后会自行删除；`cma-ws-&lt;session_id&gt;` volume 会保留，以供该会话的下一条消息使用（可通过 `docker volume rm` 丢弃）。

与 webhook 驱动的示例（Modal/Daytona/Vercel/Cloudflare）不同，这里没有 webhook——`ant beta:worker poll` 会直接对环境执行长轮询，因此无需将任何服务暴露到互联网。
