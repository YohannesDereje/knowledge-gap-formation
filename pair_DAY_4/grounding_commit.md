# Grounding Commit — Day 4

**Artifacts edited:** `ablations/paired_bootstrap_delta_a.py`, `ablations/run_ablations.py`, `README.md`, CFO memo,
evidence graph, dataset README, ablation summary (Week 11 repo)
**Commit:** https://github.com/Natnael-Alemseged/SalesConversion-Bench/commit/c841f47631bfa3df8d02e8bcc24a73b83adaac53

---

## What changed

The line `p_one_sided = sum(mean <= 0) / n_bootstrap` was replaced with McNemar's exact test via
`scipy.stats.binomtest`:

```python
# Before
p_one_sided = sum(mean <= 0 for mean in bootstrap_means) / n_bootstrap

# After
from scipy.stats import binomtest

b = sum(1 for base, trained in pairs if base == 1 and trained == 0)
c = sum(1 for base, trained in pairs if base == 0 and trained == 1)
mcnemar_result = binomtest(k=c, n=b + c, p=0.5, alternative='greater')
p_one_sided = mcnemar_result.pvalue
```

## Why

The original line bootstrapped from the observed data, which is centered at the true lift, not at zero. It answered "how
often does a large observed effect randomly look like no effect?" — not "how often would we see this effect if the null
were true?" The result was a p-value approximately 13× smaller than the correct answer, not because the evidence was
stronger but because the null hypothesis was never imposed. McNemar's exact test conditions only on the discordant
pairs (the tasks where the two systems disagreed), models them as a fair coin flip under the null — Binomial(b+c, 0.5) —
and produces a genuine one-sided p-value for "trained judge has no true advantage over baseline on matched tasks." The
bootstrap CI computation was left unchanged; it remains valid as a confidence interval.

The fix propagated across every artifact that cited the significance claim: the CFO memo, README, evidence graph,
dataset README, and ablation summary all previously carried the invalid p-value as a load-bearing number. All were
updated to reflect the McNemar result.

## Verification

The commit passed the following checks before merging:

```
python3 -m py_compile ablations/paired_bootstrap_delta_a.py ablations/run_ablations.py
python3 ablations/paired_bootstrap_delta_a.py --seed 42 --bootstrap 50000
python3 ablations/run_ablations.py --delta B --seed 42 --bootstrap 50000
```

Pre-commit ruff-check, ruff-format, and gitlint all passed.
