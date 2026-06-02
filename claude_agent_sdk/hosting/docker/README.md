# Tier 1 — Local Docker

Runs the shared image locally. Two paths:

## Ephemeral (no server)

One prompt, one process, then exit. Good for job-shaped work (batch processing,
one-off analysis) where there's no conversation to resume.

```bash
cd claude_agent_sdk/
docker build -f hosting/Dockerfile -t research-agent .
docker run --rm \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  -e PROMPT="What is the Claude Agent SDK?" \
  research-agent
```

## Hybrid (with a server)

Starts the FastAPI server and mounts `./sessions` at `/data` so conversations
survive container restarts.

```bash
cd claude_agent_sdk/hosting/docker/
docker compose up --build
```

Then, from another shell:

```bash
curl -N -X POST http://localhost:8000/sessions/demo-1/messages \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"What are the latest AI agent trends?"}'

# Follow-up — the agent remembers the first turn:
curl -N -X POST http://localhost:8000/sessions/demo-1/messages \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"Tell me more about the second one."}'

curl http://localhost:8000/health
```

Stop the container, `docker compose up` again, send another follow-up — the
agent still has context because `./sessions` persisted `/data`.

---

## 中文翻译

# 第 1 层 — 本地 Docker

在本地运行共享镜像。有两种路径：

## 临时模式（无 server）

一个 prompt，一个进程，然后退出。适合任务型工作（批处理、一次性分析），这类场景不需要恢复会话。

```bash
cd claude_agent_sdk/
docker build -f hosting/Dockerfile -t research-agent .
docker run --rm \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  -e PROMPT="What is the Claude Agent SDK?" \
  research-agent
```

## 混合模式（带 server）

启动 FastAPI server，并将 `./sessions` 挂载到 `/data`，这样会话在容器重启后仍然保留。

```bash
cd claude_agent_sdk/hosting/docker/
docker compose up --build
```

然后，在另一个 shell 中：

```bash
curl -N -X POST http://localhost:8000/sessions/demo-1/messages \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"What are the latest AI agent trends?"}'

# 后续追问 —— agent 记得第一轮对话：
curl -N -X POST http://localhost:8000/sessions/demo-1/messages \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"Tell me more about the second one."}'

curl http://localhost:8000/health
```

停止容器后，再次执行 `docker compose up`，然后再发送一次后续追问——agent 仍然保有上下文，因为 `./sessions` 已将 `/data` 持久化。
