# CMA as an MCP server

A thin [MCP](https://modelcontextprotocol.io) server that wraps the Claude [Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) Sessions API — so Claude Desktop **or** claude.ai web can start and chat with your org's hosted agents as if they were tools.

```
User ─▶ Claude (Desktop or claude.ai) ─▶ MCP: send_message + wait_for_idle ─▶ CMA session
  ▲                                                                              │
  └──────────────────────── agent's reply ◀─── stream-to-idle ◀──────────────────┘
```

Nine tools — eight are 1:1 with CMA endpoints, one (`wait_for_idle`) is the SSE→request/response shim. Same handlers, two transports.

## Quickstart

```bash
cd managed_agents/cma-mcp
bun install
claude
```

Then ask: **"walk me through setting this up."** Claude reads [`skill.md`](./skill.md) and drives whichever path you pick:

| Client | Transport | Entrypoint |
|---|---|---|
| **Claude Desktop / Claude Code** | stdio (local process) | `src/server.ts` |
| **claude.ai web** (custom Connector) | Streamable HTTP (deployed URL + bearer token) | `src/server-http.ts` |

## Tools

| Tool | CMA endpoint |
|---|---|
| `list_agents` / `get_agent` | `GET /v1/agents[/{id}]` |
| `create_session` | `POST /v1/sessions` |
| `send_message` / `interrupt` | `POST /v1/sessions/{id}/events` |
| `get_session` | `GET /v1/sessions/{id}` |
| `list_events` | `GET /v1/sessions/{id}/events` |
| `archive_session` | `POST /v1/sessions/{id}/archive` |
| **`wait_for_idle`** | streams `…/events/stream` until idle, returns reply text |

## Files

| | |
|---|---|
| `src/cma.ts` | Anthropic SDK calls — shared |
| `src/tools.ts` | Nine `server.tool(...)` registrations — shared |
| `src/server.ts` | stdio entrypoint (~10 LOC) |
| `src/server-http.ts` | HTTP entrypoint + bearer auth (~40 LOC) |
| `Dockerfile` | Fly / Railway / Render deploy for the HTTP path |

Requires `@anthropic-ai/sdk` ≥ 0.95.1.

---

## 中文翻译

# 作为 MCP server 的 CMA

这是一个轻量级 [MCP](https://modelcontextprotocol.io) server，对 Claude [Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) Sessions API 做了一层封装——因此 Claude Desktop **或** claude.ai web 可以像调用工具一样启动并与贵组织托管的 agents 进行对话。

```
User ─▶ Claude（Desktop 或 claude.ai） ─▶ MCP: send_message + wait_for_idle ─▶ CMA session
  ▲                                                                              │
  └──────────────────────── agent 的回复 ◀─── stream-to-idle ◀──────────────────┘
```

共有九个工具——其中八个与 CMA endpoint 是 1:1 对应关系，另一个（`wait_for_idle`）则是 SSE→请求/响应 的适配层。同一套 handlers，两种 transport。

## 快速开始

```bash
cd managed_agents/cma-mcp
bun install
claude
```

然后提问：**“walk me through setting this up.”** Claude 会读取 [`skill.md`](./skill.md)，并引导你走完你选择的那条路径：

| Client | Transport | Entrypoint |
|---|---|---|
| **Claude Desktop / Claude Code** | stdio（本地进程） | `src/server.ts` |
| **claude.ai web**（custom Connector） | Streamable HTTP（已部署 URL + bearer token） | `src/server-http.ts` |

## 工具

| Tool | CMA endpoint |
|---|---|
| `list_agents` / `get_agent` | `GET /v1/agents[/{id}]` |
| `create_session` | `POST /v1/sessions` |
| `send_message` / `interrupt` | `POST /v1/sessions/{id}/events` |
| `get_session` | `GET /v1/sessions/{id}` |
| `list_events` | `GET /v1/sessions/{id}/events` |
| `archive_session` | `POST /v1/sessions/{id}/archive` |
| **`wait_for_idle`** | 流式读取 `…/events/stream` 直到 idle，并返回回复文本 |

## 文件

| | |
|---|---|
| `src/cma.ts` | Anthropic SDK 调用 —— 共享 |
| `src/tools.ts` | 九个 `server.tool(...)` 注册 —— 共享 |
| `src/server.ts` | stdio 入口（约 10 LOC） |
| `src/server-http.ts` | HTTP 入口 + bearer auth（约 40 LOC） |
| `Dockerfile` | 用于 HTTP 路径部署到 Fly / Railway / Render |

需要 `@anthropic-ai/sdk` ≥ 0.95.1。
