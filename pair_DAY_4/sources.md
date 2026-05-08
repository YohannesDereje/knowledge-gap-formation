# Day 4 — Sources

**Canonical sources read (minimum 2):**

1. **Two Guidelines for Bootstrap Hypothesis Testing** — Hall, P. & Wilson, S. R. (1991) — Biometrics, 47(2), 757–762 — https://www.semanticscholar.org/paper/Two-guidelines-for-bootstrap-hypothesis-testing-Hall-Wilson/2f58848d50441502a03ac7d4a98d9d5c4e8034d0
   - Guideline 1 establishes that resampling must be done in a way that reflects the null hypothesis, even when the true hypothesis is distant from the null — directly naming the error in your peer's script, which bootstraps from the observed data (centered at the true lift) rather than from a null-centered distribution. The paper proves that violation of this guideline can spectacularly reduce test power — or in the opposite direction, produce p-values that are far smaller than the truth because the null is never given a fair chance. Guideline 2 establishes that bootstrap hypothesis tests should use methods already recognized as good for confidence interval construction — applied in the explainer to argue that the existing bootstrap CI is valid and complementary to the McNemar p-value, while only the p-value computation needs to be replaced.

2. **The McNemar test for binary matched-pairs data: mid-p and asymptotic are better than exact conditional** — Fagerland, M. W., Lydersen, S. & Laake, P. (2013) — BMC Medical Research Methodology, 13:91 — https://pmc.ncbi.nlm.nih.gov/articles/PMC3716987/
   - Establishes McNemar's test as the canonical procedure for paired binary outcomes — exactly your peer's setup: 47 tasks each producing a (baseline=0/1, trained=0/1) pair. Shows that the test conditions on discordant pairs only (b and c in the 2×2 table) because concordant pairs carry zero information about which system is better, and that under H₀ the discordant pairs follow Binomial(b+c, 0.5). Validates the exact conditional test and mid-p version as superior to asymptotic approximations at small-to-moderate sample sizes — directly applicable at n=47. Confirms the correct null distribution for your peer's specific hypothesis: "the trained judge has no true advantage over baseline on matched tasks."

---

**Tool or pattern used hands-on:**

- **`demo_paired_binary_test.py`** — A self-contained Python script (numpy + scipy only, runs on Python 3.8+) demonstrating all three approaches on synthetic data matching your peer's evaluation setup: 47 tasks, concordant/discordant pair counts matching the held-out evaluation structure, bootstrap from observed data (invalid approach), bootstrap under the null with centered differences (valid approach), and McNemar exact test via `scipy.stats.binomtest` (best approach). Produces the full comparison table showing the bootstrap from observed data reporting p=0.0001 versus McNemar's calibrated p=0.0013 — a 13× inflation from never imposing the null hypothesis. Runnable with `pip install numpy scipy && python demo_paired_binary_test.py`.

---

**Additional references:**

- **Replicability Analysis for Natural Language Processing: Testing Significance with Multiple Datasets** — Dror, R., Baumer, G., Shlain, M., Umansky, R. & Reichart, R. (2017) — Transactions of the Association for Computational Linguistics (TACL) — https://aclanthology.org/Q17-1033/
  - Canonical reference for paired bootstrap and permutation tests in NLP evaluation settings, directly applicable to LLM benchmark evaluation. Discusses the conditions under which paired resampling is valid versus when unpaired methods are appropriate, and shows how within-subject experimental designs in NLP require paired inference. Cited as the wider landscape pointer for your peer's evaluation methodology.

- **Computer Age Statistical Inference** — Efron, B. & Hastie, T. (2016) — Cambridge University Press — https://hastie.su.domains/CASI/
  - Chapter 11 provides the authoritative treatment of bootstrap confidence intervals for paired designs, including the variance-reduction mechanism and when pairing provides meaningful vs negligible precision gains. Cited as the foundational reference for the adjacent concept — why the existing paired bootstrap CI is valid for confidence interval construction even when the p-value computation was incorrect.
