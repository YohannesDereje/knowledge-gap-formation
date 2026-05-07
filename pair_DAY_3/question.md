# Day 3 — Question

**Topic:** Training and post-training mechanics

**My Question:**

Why Does Targeting Only q_proj and v_proj Successfully Suppress Banned Phrases But Fail to Generate Required Hedged Phrases?

In `training/train.py` (line 37), I configured LoRA to target only `q_proj` and `v_proj` with rank r=16, accepted as the Unsloth default without mechanistic justification. All 3 residual SOC failures in `ablations/held_out_traces.jsonl` show an identical pattern: the `regex_negative` check passes (weight 0.6 — the model stopped producing banned velocity phrases) but the `regex_positive` check fails (weight 0.4 — the model could not reliably produce the required hedged phrases: `"curious whether"`, `"if your team"`, `"haven't seen"`, `"we saw only"`). SOC has the most training pairs of any dimension (180 pairs, 15.9%, per `methodology.md §14`), yet these 3 tasks still fail. The suppression behavior was learned; the generative production behavior was not.

What is the mechanistic difference between what updating `q_proj` and `v_proj` can change in a transformer's output token distribution versus what updating `k_proj`, `o_proj`, or the FFN layers would change — and is there a principled reason why targeting only the query and value projections would successfully suppress absence-of-pattern failures (regex_negative) while leaving the model's probability of generating specific required output phrases (regex_positive) largely unchanged?

**Connection to existing artifact:**

Closing this gap would let me revise the LoRA configuration comment in `training/train.py` line 37 from `# attention projections only — LoRA-only config` to a mechanistically defended choice, explain in `methodology.md §1` why r=16 and q+v only was or was not the right call for a generative behavioral task, and determine whether the 3 residual SOC failures are a solvable training problem (add k_proj or FFN targets) or a fundamental limit of the current approach.
