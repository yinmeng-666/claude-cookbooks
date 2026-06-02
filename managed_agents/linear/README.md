# Linear × Claude Managed Agents

`@mention` a Claude [Managed Agent](https://platform.claude.com/docs/en/managed-agents/overview) in a Linear issue and get the reply as a comment.

```
Linear @mention ──▶ /linear-webhook ──▶ sessions.create (+ metadata) ──▶ 200
                                                 │
                               Claude runs to idle on Anthropic infra
                                                 │
/cma-webhook ◀── session.status_idled ◀──────────┘
      │
      └──▶ sessions.retrieve → read metadata → createAgentActivity
```

The CMA session's `metadata` (`linear_session_id`, `linear_org_id`) is the entire routing state.

## Quickstart

```bash
cd managed_agents/linear
bun install
claude
```

Then ask: **"walk me through setting this up."** Claude reads [`skill.md`](./skill.md) and drives the config — Linear OAuth app, Anthropic agent + webhook, env vars, `bun run dev` — in the order that actually works.

## Files

| | |
|---|---|
| `setup/create-agent.ts` | One-time: `agents.create` + `environments.create` |
| `src/main.ts` | Bun server, routes |
| `src/oauth.ts` | Linear OAuth (`actor=app`) + token store |
| `src/agent.ts` | `sessions.create` + `user.message` with routing metadata |
| `src/cma-webhook.ts` | `beta.webhooks.unwrap` → filter by metadata → post reply |
| `skill.md` | Setup walkthrough, gotchas, debugging |

Requires `@anthropic-ai/sdk` ≥ 0.95.1.

---

## 中文翻译

# Linear × Claude Managed Agents

在 Linear issue 中 `@mention` 一个 Claude [Managed Agent](https://platform.claude.com/docs/en/managed-agents/overview)，然后将回复作为 comment 返回。

```
Linear @mention ──▶ /linear-webhook ──▶ sessions.create (+ metadata) ──▶ 200
                                                 │
                               Claude 在 Anthropic infra 上运行直到 idle
                                                 │
/cma-webhook ◀── session.status_idled ◀──────────┘
      │
      └──▶ sessions.retrieve → 读取 metadata → createAgentActivity
```

CMA session 的 `metadata`（`linear_session_id`、`linear_org_id`）就是全部的路由状态。

## 快速开始

```bash
cd managed_agents/linear
bun install
claude
```

然后提问：**“walk me through setting this up.”** Claude 会读取 [`skill.md`](./skill.md)，并按真正可工作的顺序引导你完成配置——Linear OAuth app、Anthropic agent + webhook、环境变量、`bun run dev`。

## 文件

| | |
|---|---|
| `setup/create-agent.ts` | 一次性操作：`agents.create` + `environments.create` |
| `src/main.ts` | Bun server，路由 |
| `src/oauth.ts` | Linear OAuth（`actor=app`）+ token store |
| `src/agent.ts` | `sessions.create` + 带路由 metadata 的 `user.message` |
| `src/cma-webhook.ts` | `beta.webhooks.unwrap` → 按 metadata 过滤 → 发布回复 |
| `skill.md` | 搭建引导、注意事项、调试 |

需要 `@anthropic-ai/sdk` ≥ 0.95.1。
