# Slack × Claude Managed Agents

`@mention` a Claude [Managed Agent](https://platform.claude.com/docs/en/managed-agents/overview) in Slack and get the reply in-thread.

```
Slack @mention ──▶ /slack/events ──▶ sessions.create (+ metadata) ──▶ 200
                                              │
                            Claude runs to idle on Anthropic infra
                                              │
/cma-webhook ◀── session.status_idled ◀───────┘
      │
      └──▶ sessions.retrieve → read metadata → chat.postMessage
```

The CMA session's `metadata` (`slack_channel`, `slack_thread_ts`) is the entire routing state.

## Quickstart

```bash
cd managed_agents/slack
bun install
claude
```

Then ask: **"walk me through setting this up."** Claude reads [`skill.md`](./skill.md) and drives the config — Slack app, Anthropic agent + webhook, env vars, `bun run dev` — in the order that actually works.

## Files

| | |
|---|---|
| `setup/create-agent.ts` | One-time: `agents.create` + `environments.create` |
| `src/main.ts` | Bun server, routes |
| `src/slack-events.ts` | Verify Slack sig, `url_verification`, fire-and-forget kickoff |
| `src/agent.ts` | `sessions.create` + `user.message` with routing metadata |
| `src/cma-webhook.ts` | `beta.webhooks.unwrap` → filter by metadata → `chat.postMessage` |
| `skill.md` | Setup walkthrough, gotchas, debugging |

Requires `@anthropic-ai/sdk` ≥ 0.95.1.

---

## 中文翻译

# Slack × Claude Managed Agents

在 Slack 中 `@mention` 一个 Claude [Managed Agent](https://platform.claude.com/docs/en/managed-agents/overview)，然后在线程中获得回复。

```
Slack @mention ──▶ /slack/events ──▶ sessions.create (+ metadata) ──▶ 200
                                              │
                            Claude 在 Anthropic infra 上运行直到 idle
                                              │
/cma-webhook ◀── session.status_idled ◀───────┘
      │
      └──▶ sessions.retrieve → 读取 metadata → chat.postMessage
```

CMA session 的 `metadata`（`slack_channel`、`slack_thread_ts`）就是全部的路由状态。

## 快速开始

```bash
cd managed_agents/slack
bun install
claude
```

然后提问：**“walk me through setting this up.”** Claude 会读取 [`skill.md`](./skill.md)，并按真正可工作的顺序引导你完成配置——Slack app、Anthropic agent + webhook、环境变量、`bun run dev`。

## 文件

| | |
|---|---|
| `setup/create-agent.ts` | 一次性操作：`agents.create` + `environments.create` |
| `src/main.ts` | Bun server，路由 |
| `src/slack-events.ts` | 校验 Slack 签名、`url_verification`、fire-and-forget 启动 |
| `src/agent.ts` | `sessions.create` + 带路由 metadata 的 `user.message` |
| `src/cma-webhook.ts` | `beta.webhooks.unwrap` → 按 metadata 过滤 → `chat.postMessage` |
| `skill.md` | 搭建引导、注意事项、调试 |

需要 `@anthropic-ai/sdk` ≥ 0.95.1。
