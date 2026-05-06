# Day 2 — Sources

**Canonical sources read (minimum 2):**

1. **ReAct: Synergizing Reasoning and Acting in Language Models** — Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023) — https://arxiv.org/abs/2210.03629
   - Section 2 defines the Thought-Action-Observation loop and establishes why interleaving reasoning traces with actions is necessary for debugging and auditing agentic systems. Section 4 (ALFWorld results) provides the empirical comparison: ReAct agents achieve 71% success vs Act-only agents at 45% — a 58% relative improvement from adding reasoning traces alone. Table 2 identifies repetitive loops as the dominant failure mode of act-only agents, the direct analogue of tool-selection collapse in a routing system. Applied to the explainer as the canonical argument for why every routing decision must carry a written reasoning trace.

2. **FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance** — Chen, L., Zaharia, M., & Zou, J. (2023) — https://arxiv.org/abs/2305.05176
   - Formalizes the three required components of a working cascade system: routing policy, post-query quality scorer, and escalation policy. Identifies cascade collapse — degeneration to always using the most expensive model — as the direct result of a missing or miscalibrated post-query scorer. Demonstrates up to 98% cost reduction when the post-query scorer is present and calibrated. Applied to the explainer as the canonical source for the Three-Gate Model and the structural diagnosis of why the Document Intelligence Refinery routes every document to VisionAugmented.

---

**Tool or pattern used hands-on:**

- **`demo_router_audit.py`** — A self-contained Python script (no external dependencies, runs on Python 3.8+) demonstrating BlindRouter vs ObservationRouter on three synthetic document profiles matching the peer's actual corpus classes: a clean digital PDF (CBE Annual Report), a scanned Amharic document, and an ambiguous mixed-layout document. BlindRouter copies `triage_confidence` unchanged as `extraction_confidence` and reports `UNKNOWN` verdict for every document. ObservationRouter implements the Three-Gate Model — pre-extraction routing, post-extraction quality measurement, and evidence-based escalation — and correctly labels each decision as `correct_primary`, `correct_escalation`, or `suspected_collapse`. Total cost comparison: BlindRouter $0.08 vs ObservationRouter $0.04, demonstrating 50% cost reduction from correct gate 2 measurement alone. Code demonstrated inline in the explainer with full output shown.

---

**Additional references:**

- **Langfuse Documentation: Tracing** — Langfuse (2024) — https://langfuse.com/docs/tracing
  - Production observability pattern for agentic pipelines. Directly relevant as the peer's pipeline already uses Langfuse from Week 10. The trace schema proposed in the explainer maps onto Langfuse's span structure — each gate becomes a named span with input signals, output signals, and a verdict tag. Secondary source confirming the engineering pattern used in the explainer.

- **The Art of Abstention: Selective Prediction and Error Regularization for Natural Language Processing** — Xin, J., Tang, R., Yu, Y., & Lin, J. (2021) — https://aclanthology.org/2021.acl-long.84/
  - Foundational paper on selective prediction — when a system should accept a model's output vs abstain and escalate to a stronger model. Provides the theoretical grounding for Gate 2's quality threshold: a system that cannot measure output quality cannot make a principled abstention decision. Additional context for the FrugalGPT cascade mechanism applied in the explainer.
