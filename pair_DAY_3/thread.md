# Day 3 — Thread

**Platform:** LinkedIn and Medium

**LinkedIn post link:** https://www.linkedin.com/posts/yohannes-dereje17_ai-artificialintelligence-softwaredevelopment-share-7458265131471257602-Ok_E?utm_source=social_share_send&utm_medium=android_app&rcm=ACoAAD1aB58B1jPY2ERiFBy4PMx3Rcq9L3Ppg2E&utm_campaign=copy_link

**Medium post link:** https://medium.com/@yohannesdereje1221/lora-rank-is-not-your-problem-what-r-16-actually-controls-and-why-my-residual-failures-were-about-98118edbc6cd

---

## LinkedIn Post Content

My LoRA config said "r=16 — recipe default." I couldn't defend it.

Here's what I learned.

LoRA rank controls how many independent directions of weight change the adapter can express — not the magnitude. That's alpha/r. Two separate knobs.

Aghajanyan et al. (2020) showed that optimizing just **200 parameters** achieves 90% of full fine-tuning performance on a pre-trained model. Pre-trained models already live in a very low-dimensional subspace. r=16 is defensible. r=32 adds directions the task doesn't need.

But the real lesson came from my held-out failures:

```
regex_negative ✓  — suppression learned (banned phrases stopped)
regex_positive ✗  — generation failed (hedging phrases never appeared)
```

That's not a rank problem. That's a target module problem.

Q+V handles attention routing — enough for suppression. FFN handles per-token transformation — required for generating specific phrases. No amount of rank increase on Q+V fixes a generative failure.

Full breakdown with code in the comments 👇

*Sources: Hu et al. (2021) — LoRA | Aghajanyan et al. (2020) — Intrinsic Dimensionality*
