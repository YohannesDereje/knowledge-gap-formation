# Day 1 — Morning Call Summary

**Participants:** Yohannes Dereje and partner
**Duration:** ≥20 minutes

Yohannes's original question asked broadly why fine-tuning produces faster inference, without naming the specific artifact or the three latency values — the call sharpened it to cite `ablations/ablation_results.json` explicitly and anchor the mechanism question to the decode phase and its relationship to output token count. Mamaru's original question about output token pricing did not name the file containing the wrong formula, so the call sharpened it to `agent/llm/client.py:L91` and added the concrete bug consequence — ClaudeClient inheriting DeepSeek's 2× ratio instead of Claude's actual 5× — so the grounding commit target was unambiguous before research began. Both questions were confirmed by both partners as passing the four-property rubric before the call ended.
