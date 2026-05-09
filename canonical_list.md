# Canonical Reading List — Week 12 Contribution

**Yohannes Dereje | 10 Academy | Week 12**

An annotated list of papers, tools, and engineering patterns that closed real gaps in production AI systems this week. Organized by domain. Each entry names the exact sections worth reading and the FDE question it answers.

---

## 1. Inference-Time Mechanics

### Papers

**Kwon, W. et al. (2023). *Efficient Memory Management for Large Language Model Serving with PagedAttention.* SOSP 2023.**
https://arxiv.org/abs/2309.06180

*Read:* Section 2.2 (the two-phase inference model), Section 3 (KV cache memory footprint), Section 4.4 (shared prefix / KV cache reuse).

*FDE question it answers:* Why do output tokens cost more than input tokens, and what can I do about it at the infrastructure level?

Section 2.2 defines the prefill/decode split and explains why decode is memory-bandwidth-limited while prefill is compute-limited. Section 3 puts numbers on it — 800 KB per token for a 13B model's KV cache entry, growing with every output token. Section 4.4 is the source of the prefix caching optimization: if your pipeline sends the same system prompt on every call, the KV vectors for those tokens can be computed once and reused, making them effectively free after the first request. Any FDE debugging LLM serving costs should read this paper before touching pricing configs.

---

**Yuan, Z. et al. (2024). *LLM Inference Unveiled: Survey and Roofline Model Insights.* arXiv:2402.16363.**
https://arxiv.org/abs/2402.16363

*Read:* Section 2.2 (Roofline model framework), Table 1 (arithmetic intensity measurements).

*FDE question it answers:* How do I quantify whether a given LLM call is compute-bound or memory-bound, and by how much?

Table 1 measures arithmetic intensity directly for LLaMA-2-7B on an NVIDIA A6000 GPU. The numbers: prefill attention layers hit ~1024 operations per byte; decode attention layers collapse to ~1 operation per byte — a 1024× efficiency gap from the same layer just because the phase changed. This is the paper to cite when a stakeholder asks why output tokens cost more. Section 2.2 introduces the Roofline model, a framework for diagnosing whether any layer is hitting the compute ceiling or the memory-bandwidth ceiling. Generalizes to any GPU workload, not just LLMs.

---

**Kamath, A. K. et al. (2025). *POD-Attention: Unlocking Full Prefill-Decode Overlap for Faster LLM Inference.* arXiv:2410.18038.**
https://arxiv.org/abs/2410.18038

*Read:* Introduction and Section 2 (problem statement).

*FDE question it answers:* What is the systems-level research response to the prefill/decode asymmetry?

Read this for orientation on where the field is going, not as a primary source. The paper confirms the prefill-is-compute-bound / decode-is-memory-bound characterization as the accepted baseline and motivates hybrid batching kernels. Useful context for explaining to an engineering team why speculative decoding and hybrid batching exist.

---

## 2. Agentic Tool-Use and Routing

### Papers

**Yao, S. et al. (2023). *ReAct: Synergizing Reasoning and Acting in Language Models.* ICLR 2023.**
https://arxiv.org/abs/2210.03629

*Read:* Section 2 (Thought-Action-Observation loop definition), Section 4 / Table 2 (ALFWorld results and failure mode analysis).

*FDE question it answers:* Why does an agentic system that records only actions — not reasoning — become impossible to debug?

Table 2 is the key result: ReAct agents achieve 71% success on ALFWorld versus 45% for Act-only agents. The dominant failure mode for Act-only agents is repetitive loops — the agent keeps taking the same action because it has no written record of having already tried it. Applied directly: any agentic routing system that writes only the final tool chosen (not the signals read, thresholds checked, and validation outcome) will exhibit the same failure mode. The Thought-Action-Observation loop is the structural pattern that makes agent behavior auditable rather than opaque. This paper is the canonical argument for why every routing decision needs a reasoning trace alongside its outcome.

---

**Chen, L., Zaharia, M. & Zou, J. (2023). *FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance.* arXiv:2305.05176.**
https://arxiv.org/abs/2305.05176

*Read:* Section 2 (cascade formalization), Section 3 (FrugalGPT system), Table 2 (cost reduction results).

*FDE question it answers:* What are the required components of a tiered tool-routing system, and what causes it to always default to the expensive tier?

This paper formalizes the three things every cascade needs to work: a routing policy, a post-query quality scorer, and an escalation policy. The cascade collapse pathology — always routing to the most expensive model — is caused by a missing or miscalibrated post-query scorer. Without it, the system has no basis for accepting cheaper outputs, so it defaults to escalation every time. Table 2 shows up to 98% cost reduction when the post-query scorer is present and calibrated. The paper applies directly to any system with multiple tools at different cost/quality tiers: document extractors, LLM models, retrieval backends. The Three-Gate Model (Gate 1: pre-query routing, Gate 2: post-query quality measurement, Gate 3: escalation decision) is a concrete engineering instantiation of the FrugalGPT framework.

---

**Xin, J. et al. (2021). *The Art of Abstention: Selective Prediction and Error Regularization for Natural Language Processing.* ACL 2021.**
https://aclanthology.org/2021.acl-long.84/

*Read:* Sections 2–3 (selective prediction formalization and confidence calibration).

*FDE question it answers:* What is the theoretical grounding for the quality threshold decision in a cascade escalation gate?

Secondary source, but important for the conceptual underpinning. A system cannot make a principled "abstain and escalate" decision unless it can measure output quality — and that measurement is only meaningful if the quality scorer is calibrated. The paper formalizes when abstaining from a model's output is the correct choice. Useful reference when defending escalation thresholds to a reviewer or stakeholder.

---

## 3. Training and Parameter-Efficient Adaptation

### Papers

**Hu, E. J. et al. (2021). *LoRA: Low-Rank Adaptation of Large Language Models.* arXiv:2106.09685.**
https://arxiv.org/abs/2106.09685

*Read:* Section 4.1 (ΔW = BA decomposition), Section 4.2 (which modules to apply LoRA to), Section 7.2 (rank ablation on GPT-3), Section 7.3 (amplification factor analysis).

*FDE question it answers:* What does rank actually control, which layers should I target, and how do I defend my rank choice to a reviewer?

Section 4.1 is the geometric foundation: rank controls the dimensionality of the subspace the adapter can move base weights through. Alpha/r controls the magnitude of those moves — they are independent knobs. Section 4.2 is historically important: the authors applied LoRA to Wq and Wv only, noting MLP adaptation was left to future work. This is the origin of the "q_proj + v_proj only" default that most training scripts inherit. Section 7.2 gives the empirical answer to "what rank should I use?" — performance plateaus at r=4–8 for most tasks on GPT-3; r=16 is in the safe zone with diminishing returns beyond r=32. Section 7.3 explains why lower rank can sometimes outperform higher rank: the amplification factor alpha/r is larger at lower rank, concentrating the adaptation energy in fewer but more strongly amplified directions.

---

**Aghajanyan, A., Zettlemoyer, L. & Gupta, S. (2020). *Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning.* arXiv:2012.13255.**
https://arxiv.org/abs/2012.13255

*Read:* Sections 3–4 (intrinsic dimension definition and measurement), Table 1 (intrinsic dimensions across tasks and models).

*FDE question it answers:* Why does low-rank fine-tuning work at all, and how do I know if my task's intrinsic dimensionality is low enough for a given rank?

The central finding: optimizing only 200 trainable parameters randomly projected back into the full space achieves 90% of full fine-tuning performance on RoBERTa. Pre-trained models live in a very low-dimensional optimization subspace. This is the paper that explains *why* LoRA works. For FDE use: when someone asks why r=16 is not under-parameterized for a 200-pair behavioral task, the answer is Aghajanyan et al. — the task's intrinsic dimensionality is likely far below 16 regardless of adapter parameter count.

---

**Liu, S. et al. (2024). *DoRA: Weight-Decomposed Low-Rank Adaptation.* arXiv:2402.09353.**
https://arxiv.org/abs/2402.09353

*Read:* Sections 3–4 (decomposition formulation and comparison with LoRA).

*FDE question it answers:* When LoRA on attention layers fails to produce specific required output phrases, is there a better adapter formulation?

DoRA decomposes the weight update into magnitude and direction components separately, giving the adapter more expressive freedom within the same rank budget. For tasks requiring generative production of specific token sequences (rather than suppression of existing ones), DoRA on FFN layers is the more targeted tool. Read alongside the LoRA paper to understand when to prefer DoRA over vanilla LoRA. The key signal: if your eval shows regex_negative passing and regex_positive failing with Q+V-only LoRA, DoRA on FFN layers is the likely fix.

---

**Zhang, Q. et al. (2023). *AdaLoRA: Adaptive Budget Allocation for Parameter-Efficient Fine-Tuning.* arXiv:2303.10512.**
https://arxiv.org/abs/2303.10512

*Read:* Section 3 (SVD-based rank pruning), Section 5 (ablation on rank allocation across layers).

*FDE question it answers:* How do I find out empirically which layers actually need high rank versus low rank for my specific task?

AdaLoRA starts with a high-rank budget and prunes singular values during training — the adapter learns which directions matter and zeros out the rest. The result is a distribution of effective ranks across layers rather than a uniform rank everywhere. Section 5 shows that not all weight matrices benefit from the same rank, and that attention layers and FFN layers have different effective ranks for the same task. Use this as a diagnostic tool at v0.2 of any adapter: if you want to know whether r=16 is over- or under-parameterized in specific layers, run AdaLoRA and look at the surviving singular values.

---

## 4. Evaluation and Statistics

### Papers

**Hall, P. & Wilson, S. R. (1991). *Two Guidelines for Bootstrap Hypothesis Testing.* Biometrics, 47(2), 757–762.**
https://www.semanticscholar.org/paper/Two-guidelines-for-bootstrap-hypothesis-testing-Hall-Wilson/2f58848d50441502a03ac7d4a98d9d5c4e8034d0

*Read:* Full paper (4 pages). Read Guideline 1 twice.

*FDE question it answers:* Is the bootstrap p-value in my evaluation script actually a p-value?

Guideline 1: resampling must be done in a way that reflects the null hypothesis, even when the true hypothesis is distant from the null. Violation can produce p-values that are spectacularly wrong — in practice, far smaller than the truth. The failure mode is computing `mean(lift <= 0 for lift in bootstrap_lifts)` from data sampled at the true lift, which never gives the null a fair chance. Guideline 2: bootstrap methods are excellent for confidence intervals but require null-centering for hypothesis tests. Four pages. Every FDE running LLM evaluations should read this before writing an evaluation script.

---

**Fagerland, M. W., Lydersen, S. & Laake, P. (2013). *The McNemar test for binary matched-pairs data.* BMC Medical Research Methodology, 13:91.**
https://pmc.ncbi.nlm.nih.gov/articles/PMC3716987/

*Read:* Sections 2–3 (test formulation and null distribution), Section 5 (small-sample performance).

*FDE question it answers:* What is the correct significance test when evaluating two systems on the same binary task set?

If you have N tasks, each producing pass/fail outcomes for both a baseline and a trained system, McNemar's test is the canonical procedure. It conditions on discordant pairs only — tasks where the two systems disagreed — and models them as Binomial(b+c, 0.5) under H₀. Concordant pairs (both pass, both fail) carry zero information about which system is better and are correctly excluded. Section 5 shows the test performs well at small sample sizes (n ≈ 47 is in the tested range). Implementation: `scipy.stats.binomtest(k=c, n=b+c, p=0.5, alternative='greater')` — one line, no approximation needed.

---

**Dror, R. et al. (2017). *Replicability Analysis for Natural Language Processing.* TACL.**
https://aclanthology.org/Q17-1033/

*Read:* Section 3 (paired bootstrap and permutation tests), Section 4 (conditions for paired vs unpaired inference).

*FDE question it answers:* When does pairing matter for bootstrap confidence intervals, and when is the variance reduction negligible?

Paired bootstrap is correct by design when both systems are evaluated on the same tasks (within-subject). The variance reduction from pairing depends on the correlation between the two score vectors. When that correlation is near-zero (both systems' scores are essentially uncorrelated), the paired and unpaired CI widths are nearly identical. This paper gives the conditions under which pairing provides meaningful vs negligible precision gains. Useful for defending CI methodology to reviewers.

---

**Efron, B. & Hastie, T. (2016). *Computer Age Statistical Inference.* Cambridge University Press.**
https://hastie.su.domains/CASI/

*Read:* Chapter 11 (bootstrap confidence intervals for paired designs).

*FDE question it answers:* What is the authoritative treatment of bootstrap CI validity for paired evaluation designs?

Chapter 11 is the foundational reference for when bootstrap CI construction is valid and what assumptions it rests on. The key distinction: bootstrapping from observed data is valid for confidence intervals (it captures sampling variability around the true lift) but invalid for hypothesis testing (it does not impose the null). Use McNemar for the p-value; use bootstrap for the CI. Both are correct; they answer different questions.

---

## 5. Engineering Patterns and Tools

### Patterns

**Three-Gate Model for agentic tool routing**

Source: FrugalGPT (Chen et al., 2023) + ReAct (Yao et al., 2023), applied in Day 2 explainer.

Every system that routes between cheap and expensive tools needs three distinct decision points:
- **Gate 1 (pre-query):** Based on input signals, which tool should I try first?
- **Gate 2 (post-query):** Did the chosen tool actually produce usable output? (Measure this from the output, not from the input signal.)
- **Gate 3 (escalation):** Given Gate 2's evidence, accept or escalate?

Cascade collapse — always routing to the expensive tool — is always caused by Gate 2 being absent or reading a pre-query signal as if it were post-query evidence. Apply to: document extraction pipelines, LLM model cascades, retrieval backends, any tiered tool system. The `routing_verdict` field (`correct_primary` / `correct_escalation` / `suspected_collapse`) is the machine-readable audit entry that distinguishes these cases in the ledger.

---

**Prefill/decode cost decomposition for LLM API calls**

Source: Kwon et al. (2023), Yuan et al. (2024), applied in Day 1 explainer.

Before optimizing an LLM pipeline's cost, decompose any representative call into prefill cost and decode cost separately using real provider pricing (not inherited defaults from a different provider):

```python
prefill_cost  = input_tokens  * provider_input_price_per_token
decode_cost   = output_tokens * provider_output_price_per_token
decode_dominated = decode_cost > prefill_cost
```

The optimization target depends on which phase dominates:
- If decode-dominated: constrain output length or use speculative decoding.
- If prefill-dominated: exploit prefix caching for repeated system prompts, or reduce context window.

Apply this analysis before writing a cost optimization ticket. Verify provider pricing directly — ratio assumptions inherited from other providers are frequently wrong.

---

**LoRA configuration: target modules before rank**

Source: Hu et al. (2021), Aghajanyan et al. (2020), applied in Day 3 explainer.

When configuring a LoRA adapter for a behavioral SFT task, answer these questions in order:

1. **What kind of behavioral change does the task require?**
   - Suppression (the model should stop producing X): attention routing change → Q+V sufficient.
   - Generative production (the model should reliably produce specific phrases Y): per-token transformation → FFN layers required (`gate_proj`, `up_proj`, `down_proj`).
   - Both: Q+V + FFN.

2. **What rank is defensible for this dataset size?**
   - For suppression tasks: r=4–8 is often sufficient (Hu et al. rank plateau finding). r=16 is the safe zone.
   - For generative tasks with FFN targets: start at r=8 (FFN matrices are larger, so lower rank still gives substantial parameter counts).
   - r=32+ requires monitoring validation loss carefully; r=64 is over-parameterized for datasets under ~500 pairs.

The diagnostic signal that you chose the wrong target modules: `regex_negative` (suppression) passes and `regex_positive` (generative production) fails consistently. This is a target module problem, not a data problem and not a rank problem.

---

**McNemar's exact test as the default for paired binary evaluation**

Source: Hall & Wilson (1991), Fagerland et al. (2013), applied in Day 4 explainer.

Replace `mean(lift <= 0 for lift in bootstrap_lifts)` with:

```python
from scipy.stats import binomtest

b = sum(1 for base, trained in pairs if base == 1 and trained == 0)
c = sum(1 for base, trained in pairs if base == 0 and trained == 1)
result = binomtest(k=c, n=b + c, p=0.5, alternative='greater')
p_value = result.pvalue
```

Keep the bootstrap CI — it is valid for confidence intervals. Replace only the p-value computation. The bootstrap CI and the McNemar p-value answer different questions and both belong in the final report.

---

### Scripts (Runnable Demonstrations)

**`demo_router_audit.py`** (Day 2)
Self-contained Python 3.8+ script. No external dependencies. Demonstrates BlindRouter (no post-extraction gate) vs ObservationRouter (Three-Gate Model) on three synthetic document profiles matching a real extraction corpus. Shows: routing verdict classification, cost per document, and 50% cost reduction from correct Gate 2 measurement. Run with `python demo_router_audit.py`.

**`demo_lora_rank.py`** (Day 3)
numpy-only Python 3.8+ script. Computes trainable parameter counts at ranks 1–64 for Qwen2.5-1.5B attention projections, shows the inverse alpha/r scale factor relationship, simulates the SVD of a LoRA update at different ranks to demonstrate energy concentration, and produces a dataset-size analysis comparing parameter-per-example ratios at 221 vs 600 training pairs. Run with `python demo_lora_rank.py`.

**`demo_paired_binary_test.py`** (Day 4)
numpy + scipy Python 3.8+ script. Implements all three approaches on synthetic paired binary evaluation data (47 tasks): bootstrap from observed data (invalid, for contrast), bootstrap under the null (valid), and McNemar's exact test (canonical). Produces the comparison table showing the 13× p-value inflation from the invalid bootstrap relative to McNemar's calibrated result. Run with `pip install numpy scipy && python demo_paired_binary_test.py`.

---

## Priority Order for a New FDE

If you read nothing else from this list, read these four in order:

1. **Kwon et al. (2023) — PagedAttention, §2.2** — understand prefill vs decode before touching any cost calculation.
2. **Hu et al. (2021) — LoRA, §4.1 + §7.2** — understand rank and target modules before configuring any adapter.
3. **Chen et al. (2023) — FrugalGPT, §2** — understand the Three-Gate Model before building any tool-routing system.
4. **Hall & Wilson (1991) — Bootstrap Guidelines, full paper** — read before writing any evaluation significance test.
