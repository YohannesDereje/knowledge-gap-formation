# Day 1 — Sources

**Canonical sources read (minimum 2):**

1. **Efficient Memory Management for Large Language Model Serving with PagedAttention** — Kwon, W., Li, Z., Zhuang, S., Sheng, Y., Zheng, L., Yu, C. H., Gonzalez, J., Zhang, H., & Stoica, I. (2023) — https://arxiv.org/abs/2309.06180
   - Section 2.2 defines the two-phase inference model (prompt phase and autoregressive generation phase) and explicitly identifies decode as memory-bound due to sequential KV cache reads. Section 3 quantifies KV cache size at 800 KB per token for a 13B model. Section 4.4 describes shared prefix/KV cache reuse — the mechanism behind prefix caching.

2. **LLM Inference Unveiled: Survey and Roofline Model Insights** — Yuan, Z., Shang, Y., Zhou, Y., Dong, Z., et al. (2024) — https://arxiv.org/abs/2402.16363
   - Section 2.2 introduces the Roofline Model framework for diagnosing compute-bound vs memory-bound operations. Table 1 provides direct arithmetic intensity measurements for LLaMA-2-7B on an NVIDIA A6000 GPU: prefill layers reach 1024 OPs/byte (compute-bound), decode layers collapse to 1 OP/byte (memory-bound) — a 1024× efficiency gap that directly explains why output tokens cost more than input tokens.

---

**Tool or pattern used hands-on:**

- **Call-shape cost analysis** — Applied the arithmetic intensity framework from Yuan et al. (2024) to the actual token distributions of the Conversion Engine's outreach email generation calls. Computed prefill and decode costs separately using Claude Sonnet's real pricing ($0.000003/input token, $0.000015/output token) for a typical call shape of ~700 input tokens and ~175 output tokens. Demonstrated that the call is decode-dominated in total cost despite 4× more input tokens, because the 5× output price ratio outweighs volume. Also identified the `ClaudeClient` pricing bug — inheriting DeepSeek's 2× ratio instead of Claude's actual 5× ratio — and produced the corrected code with mechanism-level documentation. All analysis and code demonstrated inline in the explainer.

---

**Additional references:**

- **LLM Inference Series: 5. Dissecting model performance** — Pierre Lienhart (2024) — https://medium.com/@plienhar/llm-inference-series-5-dissecting-model-performance-6144aa93168f
  - Practical walkthrough of the Roofline model applied to LLM layers, including the memory bandwidth ceiling and arithmetic intensity calculations for real GPU hardware. Useful secondary source for understanding how the roofline diagonal and compute ceiling interact.

- **POD-Attention: Unlocking Full Prefill-Decode Overlap for Faster LLM Inference** — Kamath, A. K., et al. (2025) — https://arxiv.org/abs/2410.18038
  - Confirms the fundamental prefill/decode phase heterogeneity from a systems perspective. Describes prefill as compute-bound and decode as memory-bandwidth-bound as the baseline problem motivating hybrid batching kernels. Additional confirmation of the mechanism described in the explainer.
