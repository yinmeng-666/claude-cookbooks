# Iterate, get the tests green

A `calc.py` with three planted bugs and a `test_calc.py` with three assertions that catch them. Used by `CMA_iterate_fix_failing_tests.py`.

The interesting bug is `test_mean`: `mean()` calls `add` and `divide` internally, so it goes green on its own once the other two are fixed. An agent that edits `mean()` directly is over-fixing.

---

## 中文翻译

# Iterate，让测试全部变绿

一个 `calc.py`，其中埋了三个 bug；以及一个 `test_calc.py`，其中有三条断言会捕获这些 bug。供 `CMA_iterate_fix_failing_tests.py` 使用。

其中最有意思的 bug 是 `test_mean`：`mean()` 内部会调用 `add` 和 `divide`，所以只要前两个问题被修好，它自己也会随之变绿。若一个 agent 直接去修改 `mean()`，那就是过度修复了。
