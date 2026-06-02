# Gate, expense approver

A `policy.yaml` and twelve receipts (`inbox/receipts.jsonl`) used by `CMA_gate_human_in_the_loop.py`. The agent classifies each receipt against the policy with two custom tools: `decide()` for clear approves and rejects, `escalate()` for anything ambiguous.

The twelve receipts are designed to hit every branch of the policy: a handful that should auto-approve cleanly, one with no receipt image where the policy demands one, a couple in the manager-approval band, two over the threshold, one travel charge that always escalates regardless of amount, and one with a deliberately ambiguous category. A healthy run produces a mix of `approve`, `reject`, and `escalated` decisions, never all of one lane.

---

## 中文翻译

# Gate，费用审批器

一个 `policy.yaml` 和十二条收据记录（`inbox/receipts.jsonl`），供 `CMA_gate_human_in_the_loop.py` 使用。agent 会使用两个自定义工具，按照策略对每张收据进行分类：对明确可批准或拒绝的情况使用 `decide()`，对任何模糊情况使用 `escalate()`。

这十二条收据被专门设计为命中策略的每一个分支：有几条应当被顺利自动批准；有一条缺少收据图片，而策略明确要求必须有图片；有几条落在需要经理批准的金额区间；有两条超过阈值；有一条无论金额多少都必须升级处理的差旅费用；还有一条类别被故意设置得含糊不清。一次健康的运行结果应当同时包含 `approve`、`reject` 和 `escalated` 决策，而不是全部落在同一条处理路径上。
