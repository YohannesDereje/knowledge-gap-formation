# Day 1 — Thread

**Platform:** LinkedIn and medium
**Link to full post (Medium):** https://medium.com/@yohannesdereje1221/why-output-tokens-cost-twice-as-much-the-hardware-reality-behind-a-pricing-decision-584614b17881

**LinkedIn post link:** https://www.linkedin.com/posts/yohannes-dereje17_ai-artificialintelligence-softwarengineering-share-7457541517759451136-FK1t?utm_source=share&utm_medium=member_android&rcm=ACoAAD1aB58B1jPY2ERiFBy4PMx3Rcq9L3Ppg2
---

## Thread

---

**1/**

I had a bug in my AI agent's cost formula and didn't know it.

`cost = (input × $0.0000014) + (output × $0.0000028)`

Copied the 2× ratio from the pricing page. Never questioned it.

For Claude, the ratio isn't 2×. It's 5×.

Every cost figure in my CFO memo was wrong. Here's why 🧵

---

**2/**

Every LLM call runs two completely different phases.

Prefill: all input tokens processed in parallel. GPU at full capacity.

Decode: output tokens generated one at a time. Each word depends on every word before it — can't be parallelised.

Same GPU. Two completely different efficiency regimes.

---

**3/**

Researchers measured this directly for LLaMA-2-7B.

Same attention layer. Same GPU.

Prefill: 1,024 ops/byte
Decode: 1 op/byte

A 1,024× collapse in efficiency — just because the phase changed.

The GPU isn't computing. It's waiting for memory transfers. That's the hardware reality behind the pricing ratio.

---

**4/**

My pipeline: ~700 input tokens, ~175 output tokens per call.

Claude pricing — input $0.000003, output $0.000015:

Input cost: $0.00210
Output cost: $0.002625

4× more input tokens. Output still wins on cost.

Because 5× price per token beats 4× volume. My call is decode-dominated.

---

**5/**

The fix: one override in ClaudeClient with correct constants + a comment explaining the 5× ratio and why.

So the next person who reads that line doesn't repeat the gap.

---

**6/**

Output length is your real cost lever — not prompt length.

20% shorter output saves more than 20% shorter prompt when output tokens cost 5× each.

And if your system prompt is shared across calls, prefix caching makes those tokens free after the first request.

Full breakdown with paper citations 👇

---

**Medium post:**
https://medium.com/@yohannesdereje1221/why-output-tokens-cost-twice-as-much-the-hardware-reality-behind-a-pricing-decision-584614b17881

---

**Twitter thread link:** https://x.com/i/status/2051774058027479074


