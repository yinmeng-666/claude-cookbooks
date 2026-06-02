# Claude Managed Agents cookbooks

Claude Managed Agents is Anthropic's hosted runtime for stateful, tool-using
agents. You define an agent and a sandboxed environment once, then run
them in sessions that persist files, tool state, and conversation
across turns. These tutorials show it end to end.

## Applied cookbooks

- **[data_analyst_agent.ipynb](data_analyst_agent.ipynb)** builds an
  analyst that turns a CSV into a narrative HTML report using pandas
  and plotly. You'll configure an environment and agent, mount a
  dataset, stream the run, and retrieve the generated artifacts.
- **[slack_data_bot.ipynb](slack_data_bot.ipynb)** wraps that agent in
  a Slack bot. Mention it with a CSV to get the report in-thread;
  replies continue the same session.
- **[sre_incident_responder.ipynb](sre_incident_responder.ipynb)** puts
  Managed Agents on the on-call path: a pager alert starts a session,
  the agent investigates and opens a PR, then pauses for human
  approval before merging. You'll wire the alert webhook, attach a
  Skill and custom tools, and review the full run in the Console.

## Guided tutorials

End-to-end tutorials that teach the Managed Agents API surface
through realistic workflows. There's no strict reading order,
but `CMA_iterate_fix_failing_tests.ipynb` is a good entry point,
it introduces every API shape the others build on.

| Notebook | What it teaches |
|----------|-----------------|
| [`CMA_iterate_fix_failing_tests.ipynb`](CMA_iterate_fix_failing_tests.ipynb) | Do → observe → fix loop on a failing test suite. The entry-point notebook: introduces agent / environment / session, file mounts, and the streaming event loop through the lens of getting a buggy package to green. |
| [`CMA_orchestrate_issue_to_pr.ipynb`](CMA_orchestrate_issue_to_pr.ipynb) | Issue → fix → PR → CI → review → merge through a mock `gh` CLI. Multi-turn steering, mid-chain recovery from a CI failure and a review comment. Sidebar shows how to swap the file mount for a `github_repository` resource against a real repo. |
| [`CMA_explore_unfamiliar_codebase.ipynb`](CMA_explore_unfamiliar_codebase.ipynb) | Grounding in an unfamiliar codebase, with a planted stale-doc trap. Sidebar shows how to add resources to a running session via `sessions.resources.add`. |
| [`CMA_gate_human_in_the_loop.ipynb`](CMA_gate_human_in_the_loop.ipynb) | Human-in-the-loop expense approval via custom-tool `decide()` / `escalate()`. Covers the custom-tool round-trip pattern, the `requires_action` idle bounce, and parallel-tool-call dedupe. |
| [`CMA_prompt_versioning_and_rollback.ipynb`](CMA_prompt_versioning_and_rollback.ipynb) | Server-side prompt versioning: create v1, evaluate against a labelled test set, ship v2, detect a regression, roll back by pinning sessions to version 1. Covers `agents.update`, version pinning on `sessions.create`, and where the review gate moves when prompts are not code. |
| [`CMA_operate_in_production.ipynb`](CMA_operate_in_production.ipynb) | Production setup: MCP toolsets, vaults for per-end-user credentials, the `session.status_idled` webhook pattern for HITL without long-lived connections, and the resource lifecycle CRUD verbs. |
| [`CMA_remember_user_preferences.ipynb`](CMA_remember_user_preferences.ipynb) | Memory stores: a shopping agent that learns a customer's preferences in one session and recalls them in the next. Covers `memory_stores.create`, the `resources` attachment with per-attachment `instructions`, inspecting and seeding memories from your own application, and combining a per-customer read-write store with a brand-wide read-only store. |
| [`CMA_coordinate_specialist_team.ipynb`](CMA_coordinate_specialist_team.ipynb) | Heterogeneous team via the `multiagent` coordinator config: a coordinator runs three specialists (web-search researcher, file-reading librarian, rules-based pricer) with scoped toolsets to assemble a sales proposal. Covers the `multiagent` field, the `thread_created` / `thread_message_received` event types, and why per-role tool scoping matters. |
| [`CMA_verify_with_outcome_grader.ipynb`](CMA_verify_with_outcome_grader.ipynb) | Build a grade-and-revise loop with Outcomes: a writer drafts a cited research brief, a stateless grader fetches every URL and checks every quote against a rubric, and feedback drives revisions until the brief passes. Covers `user.define_outcome`, the `span.outcome_evaluation_*` events, and how to write a rubric the grader can act on. |

The streaming event loop is walked through line by line in the
iterate notebook and then factored into
`utilities.stream_until_end_turn` so the other notebooks can
import it instead of repeating the `match ev.type:` block. The
gate notebook is the exception: it keeps the loop inline because
custom-tool agents need to handle `requires_action` idle bounces
in addition to `end_turn`, which the helper doesn't cover.

## Getting started

Set `ANTHROPIC_API_KEY` in your environment, then open
`data_analyst_agent.ipynb` in Jupyter and run the cells top to
bottom. Each notebook installs its own dependencies and prompts
for any credentials it needs. The orchestrate-to-PR sidebar in
`CMA_orchestrate_issue_to_pr.ipynb` and the vault-backed MCP
example in `CMA_operate_in_production.ipynb` additionally need
`GITHUB_TOKEN` set (a fine-grained PAT with public-repo read is
enough).

All cookbook fixture data — input CSVs and supporting assets for
the applied cookbooks, plus the planted-trap fixtures the guided
tutorials read from — lives under `example_data/`. See
[`example_data/OVERVIEW.md`](example_data/OVERVIEW.md) for the
directory map.

---

## 中文翻译

# Claude Managed Agents cookbooks

Claude Managed Agents 是 Anthropic 提供的托管式运行时，用于支持有状态、可使用工具的 agents。你只需定义一次 agent 和一个沙箱化环境，然后就可以在 sessions 中运行它们，并在多轮之间持久保留文件、工具状态和对话。这些教程会端到端地展示完整流程。

## 应用型 cookbooks

- **[data_analyst_agent.ipynb](data_analyst_agent.ipynb)** 会构建一个分析师 agent，它使用 pandas 和 plotly 将 CSV 转换为叙事式 HTML 报告。你将配置环境和 agent、挂载数据集、流式获取运行过程，并取回生成的产物。
- **[slack_data_bot.ipynb](slack_data_bot.ipynb)** 将这个 agent 封装为一个 Slack bot。`@mention` 它并附上 CSV，即可在线程中获得报告；后续回复会继续同一个 session。
- **[sre_incident_responder.ipynb](sre_incident_responder.ipynb)** 将 Managed Agents 放到 on-call 流程中：pager alert 会启动一个 session，agent 进行调查并打开一个 PR，然后暂停等待人工批准后再合并。你将接入 alert webhook、挂载一个 Skill 和自定义工具，并在 Console 中查看完整运行过程。

## 引导式教程

这些端到端教程通过真实工作流来讲解 Managed Agents API 的使用面。它们没有严格的阅读顺序，但 `CMA_iterate_fix_failing_tests.ipynb` 是一个很好的入口，因为它介绍了其他教程所依赖的所有 API 形态。

| Notebook | 它会教什么 |
|----------|-------------|
| [`CMA_iterate_fix_failing_tests.ipynb`](CMA_iterate_fix_failing_tests.ipynb) | 围绕失败测试套件进行“执行 → 观察 → 修复”循环。作为入门 notebook，它借助将一个有缺陷的包修复到测试通过这一过程，引入 agent / environment / session、文件挂载以及流式事件循环。 |
| [`CMA_orchestrate_issue_to_pr.ipynb`](CMA_orchestrate_issue_to_pr.ipynb) | 通过一个 mock `gh` CLI 演示从 issue → 修复 → PR → CI → review → merge 的完整流程。包括多轮引导、在 CI 失败和 review comment 后的中途恢复。侧边栏还展示了如何将文件挂载替换为针对真实仓库的 `github_repository` resource。 |
| [`CMA_explore_unfamiliar_codebase.ipynb`](CMA_explore_unfamiliar_codebase.ipynb) | 演示如何在一个陌生代码库中建立正确上下文，其中还埋了一个过时文档陷阱。侧边栏展示如何通过 `sessions.resources.add` 向正在运行的 session 添加 resources。 |
| [`CMA_gate_human_in_the_loop.ipynb`](CMA_gate_human_in_the_loop.ipynb) | 通过自定义工具 `decide()` / `escalate()` 实现 human-in-the-loop 的费用审批。涵盖自定义工具往返模式、`requires_action` 空闲回弹，以及并行工具调用去重。 |
| [`CMA_prompt_versioning_and_rollback.ipynb`](CMA_prompt_versioning_and_rollback.ipynb) | 服务端 prompt 版本管理：创建 v1，在带标签的测试集上评估，发布 v2，发现回归后通过将 sessions 固定到 version 1 来回滚。涵盖 `agents.update`、在 `sessions.create` 上固定版本，以及当 prompts 不是代码时 review gate 应该放在哪里。 |
| [`CMA_operate_in_production.ipynb`](CMA_operate_in_production.ipynb) | 生产环境配置：MCP toolsets、面向最终用户凭证的 vaults、用于无长连接 HITL 的 `session.status_idled` webhook 模式，以及 resource 生命周期的 CRUD verbs。 |
| [`CMA_remember_user_preferences.ipynb`](CMA_remember_user_preferences.ipynb) | Memory stores：一个购物 agent 在一个 session 中学习客户偏好，并在下一个 session 中回忆出来。涵盖 `memory_stores.create`、带有每个 attachment 独立 `instructions` 的 `resources` 挂载、如何从你自己的应用中查看和预置 memories，以及如何组合“每位客户一个可读写 store”与“品牌级只读 store”。 |
| [`CMA_coordinate_specialist_team.ipynb`](CMA_coordinate_specialist_team.ipynb) | 通过 `multiagent` coordinator 配置构建异构团队：一个 coordinator 运行三个 specialist（web-search researcher、file-reading librarian、rules-based pricer），并通过分范围的 toolsets 来共同组装销售提案。涵盖 `multiagent` 字段、`thread_created` / `thread_message_received` 事件类型，以及为什么按角色限制工具范围很重要。 |
| [`CMA_verify_with_outcome_grader.ipynb`](CMA_verify_with_outcome_grader.ipynb) | 使用 Outcomes 构建评分与修订循环：writer 起草一份带引用的研究简报，stateless grader 获取每个 URL 并根据 rubric 检查每条引用，然后利用反馈持续修订直到简报通过。涵盖 `user.define_outcome`、`span.outcome_evaluation_*` 事件，以及如何编写 grader 可执行的 rubric。 |

流式事件循环会在 iterate notebook 中逐行讲解，随后被抽取为 `utilities.stream_until_end_turn`，以便其他 notebooks 可直接导入，而不必重复书写 `match ev.type:` 代码块。gate notebook 是一个例外：它保留了内联循环，因为 custom-tool agents 除了 `end_turn` 之外，还需要处理 `requires_action` 空闲回弹，而这个 helper 尚未覆盖该情形。

## 快速开始

在你的环境中设置 `ANTHROPIC_API_KEY`，然后在 Jupyter 中打开 `data_analyst_agent.ipynb`，按顺序从上到下运行各单元。每个 notebook 都会安装自己所需的依赖，并提示输入它所需的凭证。`CMA_orchestrate_issue_to_pr.ipynb` 中的 orchestrate-to-PR 侧边栏，以及 `CMA_operate_in_production.ipynb` 中基于 vault 的 MCP 示例，还额外需要设置 `GITHUB_TOKEN`（拥有 public-repo read 权限的 fine-grained PAT 就足够）。

所有 cookbook 的示例数据——应用型 cookbooks 的输入 CSV 和辅助资源，以及引导式教程读取的埋点陷阱数据——都位于 `example_data/` 目录下。目录映射见 [`example_data/OVERVIEW.md`](example_data/OVERVIEW.md)。
