# Orchestrate, drive an issue to a merged PR

Self-contained mock of a maintainer workflow, used by `CMA_orchestrate_issue_to_pr.py`. The cookbook zips this directory and hands it to the agent.

- `gh-mock`, bash script that fakes the relevant `gh` subcommands. State persists in `.gh-state/`.
- `issue_42.json`, Unicode bug report (`Café Culture` → `caf-culture`). Vague enough that the agent has to read code.
- `src/url_utils.py` + `src/blog.py` + `tests/test_urls.py`, buggy `slugify()` and the failing tests that catch it.

Two recovery points are planted: an incomplete first fix fails CI with a pytest traceback, and the mock reviewer-bot blocks the merge if `slugify()` is missing a docstring. A healthy run ends with `.gh-state/pr_101.json` showing `state: merged`, `ci/test: pass`, and an `APPROVED` review.

---

## 中文翻译

# Orchestrate，推动 issue 最终合并为 PR

这是一个自包含的维护者工作流 mock，供 `CMA_orchestrate_issue_to_pr.py` 使用。cookbook 会将这个目录打包后交给 agent。

- `gh-mock`：一个 bash 脚本，用来伪造相关 `gh` 子命令。状态会持久保存在 `.gh-state/` 中。
- `issue_42.json`：一个 Unicode bug report（`Café Culture` → `caf-culture`）。描述足够模糊，因此 agent 必须去读代码。
- `src/url_utils.py` + `src/blog.py` + `tests/test_urls.py`：带缺陷的 `slugify()` 以及能够捕获该问题的失败测试。

这里埋了两个恢复点：第一次修复若不完整，CI 会以 pytest traceback 失败；如果 `slugify()` 缺少 docstring，mock reviewer-bot 会阻止合并。一次健康的运行最终会在 `.gh-state/pr_101.json` 中呈现 `state: merged`、`ci/test: pass` 以及一个 `APPROVED` review。
