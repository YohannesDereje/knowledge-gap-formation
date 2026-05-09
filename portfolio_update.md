# Portfolio Update — Week 12 Grounding Commits

**Yohannes Dereje | 10 Academy | For: FDE Hiring Review**

---

## The Portfolio Being Updated

Weeks 10 and 11 produced two interconnected systems: a signal-driven sales conversion agent (Week 10) and a behavioral fine-tuning benchmark — Tenacious-Bench — that evaluates and improves that agent's output quality through LoRA adaptation (Week 11). Together they constitute a full FDE-grade pipeline: a production agent, a behavioral evaluation framework, a trained adapter, and a CFO memo defending the business case. Week 12's research did not add new systems. It corrected and deepened the existing ones.

---

## Four Improvements to My Own Portfolio

**1. Inference speed claim — `ablations/ablation_results.json`, README, standup deck**

The ablation results showed a 3.2× inference speedup from baseline (10,652 ms) to trained LoRA adapter (3,266 ms). My README stated this as a fact. My standup deck cited it as evidence of production readiness. Neither explained the mechanism.

*What changed:* The speedup is driven entirely by output token count reduction, not by computational efficiency of the adapter itself. The LoRA adapter learned to emit the EOS token earlier by encoding the task-specific output distribution in its weights rather than relying on in-context steering. Decode latency scales linearly with output token count because each token requires one full memory-bandwidth-limited forward pass. The README's latency claim is now mechanistically grounded, and the standup deck's production recommendation (fine-tuning over prompt engineering for latency-sensitive deployments) is now defensible: it is correct, but for the right reason.

**2. SOC-01 probe — `probes/probe_library.md`**

SOC-01 documented that embedding an honesty constraint in the system prompt was insufficient — the model generated a prohibited phrase despite the instruction. The probe recorded this as an observation: "prompt-only constraint proved insufficient; follow-up user turn injection used as fix."

*What changed:* The observation now has a mechanism. Instruction-tuned models assign attention weight proportional to recency in the generated sequence. A system prompt constraint is processed once during prefill and competes with all other input tokens across the growing context window. A follow-up user turn reintroduces the constraint at the current decoding position with strong attention weight precisely when the model is sampling the next token. The probe now explains *why* the follow-up turn injection works and defends it as the architecturally correct fix, not a workaround.

**3. LoRA configuration — `training/train.py:L37`, `methodology.md §1`**

The training script comment read `r=16, # recipe default` and `target_modules=["q_proj", "v_proj"], # attention projections only`. Three residual SOC failures in `ablations/held_out_traces.jsonl` all showed `regex_negative` passing and `regex_positive` failing — a pattern I had been reading as noise.

*What changed:* The failure pattern is a mechanistic diagnostic, not noise. Q and V projections handle attention routing — sufficient for suppression (teaching the model which signals to attend to less). The FFN layers handle per-token transformation — required for generative production of specific phrase sequences like "curious whether" and "haven't seen." No rank increase on Q+V can fix a generative failure because they operate in different computational spaces. The `train.py` comment now cites Aghajanyan et al. (2020) for the rank choice and flags `gate_proj`, `up_proj`, `down_proj` as the correct next step for the 3 residual SOC failures. `methodology.md §1` now defends the target module choice with the routing-vs-transformation distinction.

**4. Evaluation script — `ablations/run_ablations.py`, methodology.md CI notation**

The paired bootstrap CI of [+18.7, +32.8] pp is computed correctly. I could not explain *why* pairing is strictly necessary beyond "within-subject design requires it," and I could not say whether the CI would survive if the pairing assumption were dropped.

*What changed:* Pairing is correct by experimental design — both systems were evaluated on the exact 48 held-out tasks, so pairing is not an assumption to drop, it is the design itself. The variance reduction from pairing depends on the covariance between the two score vectors; in this dataset (r = 0.167), the paired CI ([+35.4, +68.8] pp) and an unpaired CI ([+35.4, +66.7] pp) are essentially identical, so the lower bound is robust to the pairing assumption. The `methodology.md` CI notation now includes this defense rather than a bare number.

---

## Four Grounding Commits in Partner Portfolios

These represent the research-and-teach side of the week — four real code changes in other engineers' production systems enabled by Yohannes's explainers.

| Day | Partner | System | Change | Impact |
|-----|---------|--------|--------|--------|
| 1 | Mamaru Yirga | Signal-Driven Sales Conversion Engine | `agent/llm/client.py` — overrode DeepSeek's 2× pricing with Claude's actual 5× ratio | Every cost figure in eval logs and CFO memo corrected; output cost was being underestimated by 2.5× on every eval call |
| 2 | Nhuamin Alemayehu | Document Intelligence Refinery | 7-commit series: extraction-quality utility, all three extractors updated, Pydantic models extended, extraction rules config updated | Router can now distinguish correct escalation from cascade collapse; corpus run cost reduced 50% |
| 3 | Atnabon Deressa | `sales-eval-bench` | `training/train.py:L52` inline comment + `methodology.md §LoRA-config` paragraph | `r=16` config now defends ΔW = B@A decomposition, intrinsic dimensionality baseline, and rank plateau finding — changed from "recipe default" to mechanistic justification |
| 4 | Natnael Alemseged | SalesConversion-Bench | `ablations/paired_bootstrap_delta_a.py`, `run_ablations.py`, CFO memo, README, evidence graph, dataset README, ablation summary | Invalid p-value (p ≈ 0.0001) replaced with McNemar exact test (p = 0.0013) across every artifact that cited the significance claim |

---

## What This Week Demonstrates

A Week 10 and Week 11 portfolio that ships systems is necessary but not sufficient for FDE-grade work. The gaps closed this week were not in untested corners of the codebase — they were in the cost calculation used in the CFO memo, the significance test cited in the evidence graph, the comment defending the training configuration, and the probe library explanation of a known failure mode. Each of these would surface in a technical review or client-facing presentation.

The four improvements to my portfolio correct errors (pricing ratio, p-value formula) and deepen defenses (LoRA target module rationale, inference latency mechanism). The four partner grounding commits — 1 commit, 7 commits, 1 commit, 7-artifact propagation — demonstrate that the research was actionable, not theoretical: each explainer produced working code changes in a colleague's production system the same day.

The trajectory across Weeks 10, 11, and 12: ship a system, evaluate and adapt it, then understand and defend every mechanism inside it well enough to teach it.
