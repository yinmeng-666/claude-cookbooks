# Knowledge Graph Extraction Evaluation

Scores entity and relation extraction against the hand-labeled gold set in `../data/sample_triples.json`.

## Running

From the repository root, install dependencies and set your API key:

```bash
uv sync --all-extras
cp .env.example .env  # then edit .env to add ANTHROPIC_API_KEY
```

Then:

```bash
uv run python capabilities/knowledge_graph/evaluation/eval_extraction.py
```

## Metrics

**Entity P/R/F1** — an extracted entity counts as a true positive if its canonicalized name matches a gold entity in the same document. Canonicalization lowercases and maps known surface-form variants ("National Aeronautics and Space Administration" → "nasa") via `data/alias_map.json`.

**Relation P/R/F1** — a relation counts as a true positive if both canonicalized endpoints match a gold (source, target) pair. **Predicate wording is ignored**: "commanded" and "was commander of" both count, but so would a semantically wrong predicate like "destroyed" between the same two entities. This makes the reported relation recall an upper bound — it measures whether the extractor found the right *connections*, not whether it labeled them correctly. For stricter scoring you would add a predicate-similarity check (e.g. a Claude judge call per candidate pair).

## Expected baseline

With `claude-haiku-4-5` and the extraction prompt from the guide, expect roughly:

| Metric | P | R | F1 |
|---|---|---|---|
| Entities | 0.80–0.90 | 0.70–0.85 | 0.75–0.85 |
| Relations | 0.70–0.85 | 0.55–0.70 | 0.60–0.75 |

These ranges are indicative; actual scores vary run-to-run due to model non-determinism.

Recall on relations is the hard number — the extractor tends to be conservative, preferring fewer high-confidence edges over exhaustive coverage. Tuning the extraction prompt for higher recall (e.g. "extract every stated relationship, even minor ones") trades precision for recall.

---

## 中文翻译

### Knowledge Graph Extraction Evaluation

对照 `../data/sample_triples.json` 中人工标注的 gold set，对实体和关系抽取进行评分。

### 运行方式

在仓库根目录下，安装依赖并设置你的 API key：

```bash
uv sync --all-extras
cp .env.example .env  # 然后编辑 .env，添加 ANTHROPIC_API_KEY
```

然后：

```bash
uv run python capabilities/knowledge_graph/evaluation/eval_extraction.py
```

### 指标

**Entity P/R/F1** —— 如果抽取得到的实体其 canonicalized name 与同一文档中的某个 gold entity 匹配，则该实体计为 true positive。Canonicalization 会将名称转为小写，并通过 `data/alias_map.json` 将已知的 surface-form variants（例如 `"National Aeronautics and Space Administration"` → `"nasa"`）映射到统一形式。

**Relation P/R/F1** —— 如果关系两端 canonicalized 后的端点同时匹配某个 gold 的 `(source, target)` 对，则该关系计为 true positive。**Predicate wording 不参与判断**：`"commanded"` 和 `"was commander of"` 都会被视为正确，但即使是语义错误的 predicate，例如发生在同一对实体之间的 `"destroyed"`，也同样会被算入。这意味着报告出来的 relation recall 是一个上界——它衡量的是抽取器是否找到了正确的*连接关系*，而不是是否给它们打上了正确标签。若要进行更严格的评分，你可以增加 predicate-similarity 检查（例如对每个候选 pair 调用一次 Claude judge）。

### 预期基线

使用 `claude-haiku-4-5` 和指南中的 extraction prompt 时，预期结果大致如下：

| Metric | P | R | F1 |
|---|---|---|---|
| Entities | 0.80–0.90 | 0.70–0.85 | 0.75–0.85 |
| Relations | 0.70–0.85 | 0.55–0.70 | 0.60–0.75 |

这些区间仅供参考；由于模型的非确定性，实际分数在不同运行间会有所波动。

Relation 的 recall 是最难提升的指标——抽取器往往较为保守，更倾向于输出较少但高置信度的边，而不是穷尽所有可能关系。若将 extraction prompt 调整为追求更高 recall（例如“提取所有陈述过的关系，即使是次要关系”），通常会以 precision 下降为代价换取 recall 提升。
