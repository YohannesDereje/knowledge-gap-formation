# Grounding Commit.md

## Pointer to actual edit

I grounded my peer’s explainer in my Document Intelligence Refinery repository through this seven-commit patch series:

1. [docs: peer explainer documentation](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/5a631f8e8bb4c0c63a1ccd3d200a2fc23a099bf7)
2. [feat(utils): add extractor quality control after peer explainer suggestion](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/ac51454c3865b47e64ce2b0238e3dc0ef21e2b70)
3. [feat(strategies): update vision extractor with peer explainer modification](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/64b7e84803c02907251fd8970b48526841e9a246)
4. [feat(strategies): update layout extractor with peer explainer modification](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/b0e37e8fc206d32fc7d54578a28d6c2ada3a8dd0)
5. [feat(strategies): update fast text extractor with peer explainer modification](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/3d8a2a0a4140f138f4843be9d5a4f7a778c187b3)
6. [feat(models): update pydantic models with peer explainer modification](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/0f4430498248417c05b950c9a0b2e3d0b68ab663)
7. [config(rubric): update extraction rules with peer explainer modification](https://github.com/nuhaminae/Document-Intelligence-Refinery/commit/b7e43c3a851a62427bb28ac46061f75352a0dec5)

## What changed and why

My peer’s explainer showed that the extraction router could not distinguish **correct escalation** from **tool-selection collapse** because the pipeline was relying too heavily on pre-extraction triage confidence instead of measuring whether each extractor actually produced usable output. In response, I updated the Document Intelligence Refinery so extraction quality is now treated as a post-extraction signal. I added a quality-control utility, updated the FastText, Layout-Aware, and Vision extractors to compute actual output quality, extended the Pydantic models to carry richer extraction-quality evidence, and moved relevant quality thresholds into the extraction rules configuration. This grounds the explainer in code by turning the abstract three-gate idea into an implementation direction. The router should not just record that a document ended up in `VisionAugmented`, it should be able to prove whether cheaper tools failed, whether escalation was justified, and what validation signal caused the final tool choice.
