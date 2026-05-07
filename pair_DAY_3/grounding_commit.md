# Grounding Commit — Day 3

**Asker:** Atnabon Deressa
**Day:** 3
**Topic:** Training & post-training — LoRA rank vs adapter expressivity
**Date:** 2026-05-07

---

## Status

**Complete — explainer received from Yohannes Dereje (2026-05-07, 10:30 PM).**

Mechanism conclusion: rank `r` controls the **dimensionality of the update
subspace** via the `ΔW = B @ A` decomposition, not the magnitude of updates
(magnitude is governed by `alpha/r` independently). Aghajanyan et al. 2020
shows LLMs have low intrinsic dimensionality (~200 effective dimensions), so
r=16 gives the 221-pair task sufficient expressivity without overfitting.
Hu et al. 2021 §7.2 rank ablation confirms r=4–8 is usually sufficient with
diminishing returns at r=32+. r=16 is the safe choice for this task size.
The real lever for the 3 residual SOC failures is target modules, not rank —
which is also the finding from Yohannes's complementary Day 3 question.

## Concrete edit

- **Repository:** `sales-eval-bench`
- **Files:** `training/train.py:L52` and `methodology.md §LoRA-config`

```diff
# training/train.py:L52

  peft_config = LoraConfig(
-     r=16,                # accepted from Unsloth default — no mechanistic justification
-     lora_alpha=32,       # effective scaling alpha/r = 2 — also unjustified
+     r=16,                # ΔW = B@A; rank = update subspace dimensionality, not magnitude;
+                          # r=16 safe for 221 pairs given ~200-dim LLM intrinsic dimensionality
+                          # (Aghajanyan 2020); magnitude controlled separately by alpha/r = 2
+     lora_alpha=32,       # alpha/r = 2 — standard effective scaling; decouples from rank choice
      target_modules=["q_proj", "v_proj"],
      lora_dropout=0.05,
      bias="none",
      task_type="CAUSAL_LM",
  )
```

**Addition to `methodology.md §LoRA-config`:**

> **Rank choice (r=16):** LoRA decomposes the weight update as ΔW = B @ A,
> where B ∈ R^{d×r} and A ∈ R^{r×k}. The rank `r` controls the dimensionality
> of the subspace the adapter can move base weights through — not the magnitude
> of those moves, which is governed by the `alpha/r` scaling factor separately.
> r=16 was chosen for the 221-pair SFT task based on two grounds: (1) Aghajanyan
> et al. 2020 showed that large language models have low intrinsic dimensionality
> (~200 effective dimensions), so r=16 provides ample expressivity for this task
> without allowing the adapter to overfit to noise; (2) Hu et al. 2021 §7.2 rank
> ablation found r=4–8 sufficient across a range of NLP tasks with diminishing
> returns at r=32+. r=16 sits in the safe range. The `alpha/r = 2` ratio
> (lora_alpha=32) keeps effective update scaling stable across any future rank
> changes. This choice will be re-evaluated at v0.2 (~600 pairs) — at that
> dataset size r=32 becomes more defensible, but should be validated empirically
> against held-out traces rather than assumed.

## Was this a wording fix or a mechanism fix?

- [ ] Wording fix
- [x] Mechanism fix
- [ ] Both

The inline comment changed from no-justification to a mechanistic defense
grounded in the SVD-decomposition view and the Aghajanyan 2020 intrinsic
dimensionality result. The `methodology.md` addition is new mechanistic
content that did not exist before the explainer.
