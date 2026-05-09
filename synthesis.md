# Week 12 Synthesis — Knowledge Gap Formulation for Compounding

**Yohannes Dereje | 10 Academy | Week 12**

---

## Overview

Four days. Eight gaps. Every gap connected to a real artifact. This synthesis records the eight knowledge gaps closed across Week 12 — four I named from my own work, four I researched and explained for a partner — the most surprising thing the week uncovered, and the canonical reading list I am carrying forward.

---

## Part 1 — Eight Gaps Closed

### Gaps I Named (Asker Role)

**Day 1 — Why does a LoRA adapter generate outputs 3.2× faster than baseline?**

My `ablations/ablation_results.json` showed inference latency dropping from 10,652 ms (baseline) to 3,266 ms (trained LoRA adapter). I had been writing "3.2× faster" in my README and standup deck without being able to explain the mechanism. My assumption was that adding a LoRA adapter would *increase* overhead.

Gap closed: the speedup is almost entirely a reduction in output token count. The LoRA adapter learned to emit the EOS token earlier by encoding the task-specific output distribution in its weights, rather than relying on in-context steering from a system prompt. The mechanism underneath: decode latency scales linearly with output token count because each token requires one full memory-bandwidth-limited forward pass. The adapter does not make each token cheaper — it produces fewer tokens. This distinction changes the optimization framing: if I want faster inference without a LoRA adapter, the correct lever is constraining output length in the prompt, not adjusting the model.

**Day 2 — Why does an identical constraint fail in the system prompt but succeed as a follow-up user turn?**

In `agent_core.py` Phase 5 (~lines 461–503), SOC-01 proved that embedding the honesty constraint in the system prompt was not sufficient. The model generated a prohibited phrase despite the explicit instruction not to. The fix — injecting a follow-up user turn mid-conversation naming the specific violation — passed consistently.

Gap closed: instruction-tuned models assign attention weight to the most recent instruction-shaped text. System prompt constraints are processed once during prefill and compete with all other input tokens across the growing context. A late-arriving user turn reintroduces the constraint at the current decoding position, where it receives strong attention weight in the generation of the very next tokens. This means the follow-up user turn injection is not a workaround — it is the architecturally correct fix for a model that has already produced a violation and needs to be steered before the next token is sampled.

**Day 3 — Why does Q+V-only LoRA suppress banned phrases but fail to generate required hedged phrases?**

The 3 residual SOC failures in `ablations/held_out_traces.jsonl` all shared the same pattern: `regex_negative` passed (suppression of banned velocity phrases) but `regex_positive` failed (production of required hedging phrases: "curious whether," "haven't seen," "if your team"). This looked like noise. I could not explain why the failure pattern was so consistent.

Gap closed: the pattern is a diagnostic signature, not noise. Attention projections (Q, K, V) perform information routing — they determine which tokens attend to which other tokens and what values propagate forward. A LoRA adapter on Q and V can teach the model to route attention away from patterns that trigger assertive language — a routing change — but it cannot teach the FFN to compose new output phrases. The FFN is the per-token transformation engine where specific token sequences are generated from the residual stream. Suppression is a routing task; generative production of required phrases is a transformation task. Adding `gate_proj`, `up_proj`, and `down_proj` to `LORA_TARGETS` is the correct remediation for the 3 residual failures — not increasing rank on Q and V.

**Day 4 — What makes paired bootstrapping necessary, and what would an unpaired bootstrap have reported?**

My `paired_bootstrap` function in `ablations/run_ablations.py` resamples a single set of task indices and applies it to both the trained LoRA scores and the baseline scores. I knew this was the correct procedure for a within-subject design, but I could not explain the mathematical property that makes pairing strictly necessary or quantify what would have changed with an unpaired approach.

Gap closed: pairing is correct because the experimental design evaluated both systems on the exact same 48 held-out tasks. This is a within-subject design by construction — the correct justification is the design, not the formula. The variance reduction from pairing depends on the covariance between the two score vectors. In my data, that covariance is near-zero (r = 0.167), which means the paired CI ([+35.4, +68.8] pp) and an unpaired CI ([+35.4, +66.7] pp) are essentially identical. My gap was real — I could not explain the mechanism — but the conclusion turns out to be robust to the pairing assumption in this specific dataset.

---

### Gaps I Researched and Explained (Explainer Role)

**Day 1 — Why do output tokens cost 5× input tokens? (for Mamaru)**

The Conversion Engine's `ClaudeClient` was inheriting DeepSeek pricing constants, treating output as 2× input. Every cost figure in the eval logs and CFO memo was wrong by a factor of 2.5×.

What I researched: the pricing ratio reflects hardware reality. Prefill is compute-bound — all input tokens are processed simultaneously via matrix-matrix multiplication, achieving ~1024 operations per byte of memory access. Decode is memory-bound — each output token requires one sequential matrix-vector pass, dropping arithmetic intensity to ~1 operation per byte. The 5× ratio for Claude Sonnet reflects that decode GPU time is genuinely less productive per token. The grounding commit fixed ClaudeClient's `calculate_cost` method to use Claude's actual pricing and changed the optimization target for the Conversion Engine from generic "reduce costs" to specific "constrain output length or exploit prefix caching for the shared system prompt."

**Day 2 — How to distinguish correct escalation from cascade collapse in agentic tool routing (for Nhuamin)**

Every document in the Document Intelligence Refinery's corpus was being routed to VisionAugmented — the most expensive extraction tool. The pipeline could not distinguish correct escalation from silent failure because both produced identical ledger entries.

What I researched: FrugalGPT (Chen et al., 2023) formalizes why cascaded systems degenerate to always using the expensive tier — the post-query quality scorer is missing. The Refinery's extractors were copying triage confidence unchanged (`extraction_confidence = profile.triage_confidence`), never re-measuring quality from actual extractor output. The Three-Gate Model I wrote into the explainer — Gate 1 (pre-extraction routing), Gate 2 (post-extraction quality measurement), Gate 3 (evidence-based escalation) — gave the Refinery a structural fix. Seven grounding commits implemented extraction-quality utilities, updated all three extractors to compute real output quality, and added the `routing_verdict` field to the ledger.

**Day 3 — What LoRA rank actually controls, and was r=16 the right choice for 221 training pairs? (for Atnabon)**

The training script comment read `r=16 # recipe default` — no mechanistic justification.

What I researched: rank controls the dimensionality of the update subspace ΔW = B × A, not the magnitude. Magnitude is controlled separately by alpha/r (alpha=32, r=16 gives scale = 2.0). These are independent knobs. Aghajanyan et al. (2020) showed that pre-trained LLMs have intrinsic dimensionality of roughly 200 effective dimensions — optimizing only 200 parameters achieves 90% of full fine-tuning on RoBERTa. This makes r=16 safe for a 221-pair suppression task: the task lives in a low-dimensional subspace, so the extra adapter directions simply go unused rather than causing harm. The Hu et al. (2021) rank ablation confirms performance plateaus at r=4–8 for most tasks; r=16 sits in the safe zone, and r=32+ produces diminishing returns while shrinking the per-direction amplification factor.

**Day 4 — Why the bootstrap p-value is not a valid p-value, and what to use instead (for Natnael)**

The evaluation script reported p ≈ 0.0001 for the trained judge's +52 pp lift using `mean(lift <= 0 for lift in bootstrap_lifts)`.

What I researched: this is not a p-value. It violates Hall & Wilson's (1991) Guideline 1 — resampling must reflect the null hypothesis. Bootstrapping from observed data builds a distribution centered at the true lift, not at zero. Asking how often that distribution falls below zero answers "how stable is the observed effect?" not "how often would we see this effect if the null were true?" McNemar's exact test is the correct procedure for paired binary outcomes: condition on discordant pairs only (the tasks where the two systems disagreed), model them as Binomial(b+c, 0.5) under H₀, compute one-sided p. The corrected result was p = 0.0013 — still highly significant, but 13× less dramatic than the invalid bootstrap. The grounding commit updated the evaluation script, CFO memo, README, and evidence graph to use the McNemar result.

---

## Part 2 — Most Surprising Finding

The most surprising thing the week uncovered was how close to the surface real production bugs were hiding behind vague language in my own artifacts.

Day 1's explainer for Mamaru found an actual pricing bug — a ClaudeClient inheriting DeepSeek's 2× ratio instead of Claude's 5× ratio — that had been distorting every cost figure in my eval logs and CFO memo. The bug was discoverable from first principles once you understood that different providers have different pricing ratios because those ratios reflect different hardware economics, not arbitrary business decisions. I had been treating the pricing ratio as a lookup-table value. Understanding the mechanism made the bug obvious.

Day 4 was similar: a p-value that was 13× too small sat unquestioned in an evaluation script and propagated into a CFO memo, a README, and an evidence graph, because the code pattern `mean(lift <= 0 for lift in bootstrap_lifts)` looks exactly like a p-value computation.

The pattern across both: abstraction language and plausible-looking code can carry incorrect numbers for a long time if you accept the output without understanding the mechanism that produced it.

---

## Part 3 — Canonical Reading List

**Papers**

- **Kwon et al. (2023) — PagedAttention / vLLM.** *Efficient Memory Management for Large Language Model Serving with PagedAttention.* SOSP 2023. https://arxiv.org/abs/2309.06180 — The foundational paper on LLM serving systems. Section 2.2 defines the two-phase inference model. Every FDE who cares about inference cost should read this.

- **Yuan et al. (2024) — LLM Inference Roofline.** *LLM Inference Unveiled: Survey and Roofline Model Insights.* https://arxiv.org/abs/2402.16363 — Table 1 gives the 1024× arithmetic intensity gap between prefill and decode measured on real hardware. The roofline model framework belongs in every FDE's debugging toolkit.

- **Hu et al. (2021) — LoRA.** *Low-Rank Adaptation of Large Language Models.* https://arxiv.org/abs/2106.09685 — Section 4.1 (ΔW = BA decomposition), Section 7.2 (rank ablation), Section 7.3 (amplification factor). The three sections you need to read to defend any LoRA configuration.

- **Aghajanyan et al. (2020) — Intrinsic Dimensionality.** *Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning.* https://arxiv.org/abs/2012.13255 — The theoretical grounding for why low-rank adapters work at all. ~200 effective dimensions is the number to remember.

- **Chen et al. (2023) — FrugalGPT.** *How to Use Large Language Models While Reducing Cost and Improving Performance.* https://arxiv.org/abs/2305.05176 — The canonical source on cascade collapse. Every pipeline that routes between cheap and expensive tools needs its three-component model: routing policy, post-query quality scorer, escalation policy.

- **Yao et al. (2023) — ReAct.** *Synergizing Reasoning and Acting in Language Models.* https://arxiv.org/abs/2210.03629 — The argument for why every agentic routing decision needs a written reasoning trace, not just a final action. The 71% vs 45% success rate difference comes entirely from adding reasoning.

- **Hall & Wilson (1991) — Bootstrap Hypothesis Testing.** *Two Guidelines for Bootstrap Hypothesis Testing.* Biometrics, 47(2). https://www.semanticscholar.org/paper/Two-guidelines-for-bootstrap-hypothesis-testing-Hall-Wilson/2f58848d50441502a03ac7d4a98d9d5c4e8034d0 — Guideline 1 is the rule every FDE running LLM evaluations needs to know: resampling must reflect the null hypothesis.

- **Fagerland et al. (2013) — McNemar's Test.** *The McNemar test for binary matched-pairs data.* BMC Medical Research Methodology. https://pmc.ncbi.nlm.nih.gov/articles/PMC3716987/ — The canonical reference for paired binary evaluation. If you are evaluating two systems on the same task set with pass/fail outcomes, this is the test to use.

**Engineering Patterns**

- **Three-Gate Model for agentic routing:** Pre-query routing → post-query quality scoring → evidence-based escalation. The middle gate is the one that is always missing when a cascade collapses to the expensive tier. Applies to document extraction, LLM cascades, medical triage, database query planning.

- **Call-shape cost analysis:** Decompose any LLM API call into prefill cost and decode cost separately using real provider pricing, not inherited constants. Whether a pipeline is prefill-dominated or decode-dominated determines the correct optimization target.

- **McNemar's exact test for eval scripts:** `scipy.stats.binomtest(k=c, n=b+c, p=0.5, alternative='greater')`. One line. Replace any `mean(lift <= 0 for lift in bootstrap_lifts)` with this.

- **LoRA target module selection before rank selection:** For behavioral SFT tasks, the question to answer first is routing vs transformation — what kind of behavioral change does the task require? Then choose target modules (Q+V for suppression, FFN for generative production), then choose rank.
