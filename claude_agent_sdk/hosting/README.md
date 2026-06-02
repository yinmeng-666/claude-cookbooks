# Hosting the research agent

This directory deploys the research agent from
[`00_The_one_liner_research_agent.ipynb`](../00_The_one_liner_research_agent.ipynb)
through three tiers: local Docker, Modal, and Kubernetes. The agent, the
container image, and the HTTP interface are the **same** across all three —
only the operational machinery around the container changes.

Walk through [`07_Hosting_the_agent.ipynb`](../07_Hosting_the_agent.ipynb) for
the full narrative.

## Interface contract

All three tiers conform to this contract. The Kubernetes gateway routes against it.

```
The agent image exposes:

  GET  /health
       → 200 {"status": "ok"}
       Liveness check for orchestration.

  POST /sessions/{session_id}/messages
       Body: {"prompt": "<user message>"}
       → 200 text/event-stream
         event: message  — data is a serialized SDK message
                           (SystemMessage | AssistantMessage | ResultMessage)
         event: done     — turn complete
         event: error    — data is {"message": "..."}
       Resumes the session if it exists, creates if not.
       Stream contains ONLY the new turn's events, not history.

  session_id MUST match ^[A-Za-z0-9][A-Za-z0-9_-]{0,63}$. Invalid → 400.

  Port: 8000
  Required env: ANTHROPIC_API_KEY
  Optional env: MODEL (default: claude-sonnet-4-6),
                CLAUDE_CONFIG_DIR (default: /data — mount this for persistence),
                AGENT_AUTH_TOKEN (when set, /sessions/* requires
                                  Authorization: Bearer <token>; /health stays open)

  ⚠️  SECURITY: This server has no authentication by default. It MUST sit
      behind a gateway/proxy that (1) authenticates callers and (2) only
      forwards session_ids that belong to the authenticated caller. Do not
      expose port 8000 to the internet directly.

      If there is no gateway (tier 2: Modal hands out a public tunnel), set
      AGENT_AUTH_TOKEN. It is the minimal stand-in, not a replacement — it
      does not scope session_ids to callers.

  LIFECYCLE: The server does not self-terminate. The orchestrator is
      responsible for killing idle containers.
```

## Directory layout

```
hosting/
  README.md            ← you are here
  Dockerfile           ← shared agent image (build context = claude_agent_sdk/)
  Dockerfile.dockerignore
  server.py            ← FastAPI + SSE server (the hybrid path)
  run_once.py          ← ephemeral path: one prompt, print result, exit
  entrypoint.sh        ← dispatches: ephemeral run vs. start server
  requirements.txt
  .env.example         ← ANTHROPIC_API_KEY, MODEL
  docker/              ← Tier 1: local Docker / docker-compose
  modal/               ← Tier 2: Modal Sandbox
  kubernetes/          ← Tier 3: pod-per-session on your own k8s cluster
```

## Build

The Docker build context is the **parent** directory (`claude_agent_sdk/`), not
`hosting/`, because the image needs `research_agent/` and `utils/` alongside
`hosting/`:

```bash
cd claude_agent_sdk/
docker build -f hosting/Dockerfile -t research-agent .
```

> The build uses the sibling `Dockerfile.dockerignore` (note the prefixed name),
> which requires BuildKit — the default builder in Docker 23.0+ and Docker
> Desktop. On an older engine, run the build with `DOCKER_BUILDKIT=1`.

---

## 中文翻译

这个目录通过三个层级部署来自 [`00_The_one_liner_research_agent.ipynb`](../00_The_one_liner_research_agent.ipynb) 的 research agent：本地 Docker、Modal 和 Kubernetes。三个层级中的智能体、容器镜像以及 HTTP 接口都是**相同的**——变化的只有围绕容器的运维机制。

请阅读 [`07_Hosting_the_agent.ipynb`](../07_Hosting_the_agent.ipynb) 以了解完整叙述。

## 接口契约

三个层级都遵循这个契约。Kubernetes gateway 会基于它进行路由。

```
该 agent 镜像暴露：

  GET  /health
       → 200 {"status": "ok"}
       用于编排的存活检查。

  POST /sessions/{session_id}/messages
       Body: {"prompt": "<user message>"}
       → 200 text/event-stream
         event: message  — data 是序列化后的 SDK message
                           (SystemMessage | AssistantMessage | ResultMessage)
         event: done     — 当前轮次完成
         event: error    — data 为 {"message": "..."}
       如果 session 已存在则恢复，不存在则创建。
       stream 中仅包含本轮新增事件，不包含历史记录。

  session_id 必须匹配 ^[A-Za-z0-9][A-Za-z0-9_-]{0,63}$。非法值 → 400。

  端口: 8000
  必需环境变量: ANTHROPIC_API_KEY
  可选环境变量: MODEL (默认: claude-sonnet-4-6),
                CLAUDE_CONFIG_DIR (默认: /data — 如需持久化请挂载该路径),
                AGENT_AUTH_TOKEN (设置后，/sessions/* 需要
                                  Authorization: Bearer <token>; /health 保持开放)

  ⚠️  安全性：该 server 默认不带认证。它必须部署在 gateway/proxy 之后，
      gateway/proxy 需要做到：(1) 认证调用方；(2) 只转发属于该已认证调用方
      的 session_id。不要将 8000 端口直接暴露到互联网。

      如果没有 gateway（第 2 层：Modal 会提供一个公开隧道），请设置
      AGENT_AUTH_TOKEN。它只是一个最小替代方案，并不是完整替代——
      它不会把 session_id 绑定到具体调用方。

  生命周期：server 不会自我终止。由 orchestrator 负责清理空闲容器。
```

## 目录结构

```
hosting/
  README.md            ← 你当前所在位置
  Dockerfile           ← 共享的 agent 镜像（build context = claude_agent_sdk/）
  Dockerfile.dockerignore
  server.py            ← FastAPI + SSE server（hybrid 路径）
  run_once.py          ← 临时路径：一个 prompt，打印结果，然后退出
  entrypoint.sh        ← 分发：临时运行 或 启动 server
  requirements.txt
  .env.example         ← ANTHROPIC_API_KEY, MODEL
  docker/              ← 第 1 层：本地 Docker / docker-compose
  modal/               ← 第 2 层：Modal Sandbox
  kubernetes/          ← 第 3 层：在你自己的 k8s cluster 上按 session 分配 pod
```

## 构建

Docker build context 是**父目录**（`claude_agent_sdk/`），而不是 `hosting/`，因为镜像除了 `hosting/` 外还需要 `research_agent/` 和 `utils/`：

```bash
cd claude_agent_sdk/
docker build -f hosting/Dockerfile -t research-agent .
```

> 构建会使用同级目录中的 `Dockerfile.dockerignore`（注意这个带前缀的文件名），这要求启用 BuildKit——它在 Docker 23.0+ 和 Docker Desktop 中是默认 builder。如果你使用较旧的 engine，请在构建时加上 `DOCKER_BUILDKIT=1`。
