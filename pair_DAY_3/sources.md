# Day 3 — Sources

**Canonical sources read (minimum 2):**

1. **LoRA: Low-Rank Adaptation of Large Language Models** — Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. (2021) — https://arxiv.org/abs/2106.09685
   - Section 4.1 defines the ΔW = BA decomposition and establishes that rank controls the dimensionality of the update subspace while alpha/r controls the scaling magnitude — the core geometric distinction the explainer is built on. Section 7.2 provides the rank ablation on GPT-3 (ranks 1, 2, 4, 8, 64) showing performance plateaus at r=4–8 for most tasks, which grounds the verdict that r=16 is in the safe zone but not uniquely optimal. Section 7.3 shows the amplification factor relationship — lower rank produces higher amplification per direction — used to explain why a narrowly targeted low-rank adapter can outperform a high-rank one for suppression tasks. Section 4.2 explicitly applies LoRA to Wq and Wv only, noting MLP adaptation is left to future work, directly sourcing the default your peer inherited.

2. **Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning** — Aghajanyan, A., Zettlemoyer, L., & Gupta, S. (2020) — https://arxiv.org/abs/2012.13255
   - Shows empirically that common pre-trained models have very low intrinsic dimension — optimizing only 200 trainable parameters randomly projected back into the full space achieves 90% of full fine-tuning performance on RoBERTa classification tasks. Establishes the theoretical foundation for why low-rank adapters work and why r=16 is not under-parameterized for a narrow suppression task: the task's intrinsic dimensionality is far below 16, so the adapter has more directions available than the task needs. Also shows that pre-training implicitly minimizes intrinsic dimension, explaining why Qwen2.5-1.5B requires only a tiny subspace update to learn SOC suppression behavior.

---

**Tool or pattern used hands-on:**

- **`demo_lora_rank.py`** — A self-contained Python script (numpy only, no ML framework required, runs on Python 3.8+) that computes trainable parameter counts at ranks 1–64 for Qwen2.5-1.5B attention projections (d_model=1536, 28 layers, q_proj + v_proj), shows the inverse relationship between rank and the alpha/r scale factor, simulates the SVD of a LoRA update at different ranks to demonstrate energy concentration, and produces a dataset-size analysis comparing parameter-per-example ratios at 221 pairs vs 600 pairs. Key output: at r=16 with 221 pairs, the params-per-example ratio is ~12,454 — large in absolute terms but defensible given the low intrinsic dimensionality of suppression tasks. At r=32, the ratio doubles to ~24,909 with no performance benefit predicted by the Hu et al. rank plateau finding. All output shown inline in the explainer.

---

**Additional references:**

- **DoRA: Weight-Decomposed Low-Rank Adaptation** — Liu, S., Zhu, C., Wan, W., Wu, X., Xiao, W., Hamid, R., & Yin, Z. (2024) — https://arxiv.org/abs/2402.09353
  - Decomposes the weight update into magnitude and direction components separately, giving the adapter more expressive freedom within the same rank budget. Cited in the explainer as the adjacent concept explaining why DoRA on FFN layers would likely close regex_positive failures better than increasing rank on Q+V — the direction-magnitude decomposition addresses the generative production gap that Q+V routing cannot reach.

- **AdaLoRA: Adaptive Budget Allocation for Parameter-Efficient Fine-Tuning** — Zhang, Q., Chen, M., Bukharin, A., He, P., Cheng, Y., Chen, W., & Zhao, T. (2023) — https://arxiv.org/abs/2303.10512
  - Starts with a high-rank budget and prunes singular values during training, learning which directions actually matter and zeroing out the rest. Cited in the explainer as the wider landscape context — the tool that would give empirical evidence of which matrices actually needed rank-16 versus rank-4 in your peer's setup, and the natural next step for v0.2 if uniform rank across all layers proves suboptimal.
