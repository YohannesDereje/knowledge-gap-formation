# Day 1 — Evening Call Summary

**Participants:** Yohannes Dereje and partner
**Duration:** ≥45 minutes

Yohannes walked Mamaru through the output token pricing explainer and Mamaru confirmed the gap was closed, giving feedback that the call-shape cost analysis section was the most valuable part because it connected the abstract hardware mechanism directly to a concrete error in their production code. Mamaru then walked Yohannes through the 3.2× inference speedup explainer, explaining that the speedup is driven entirely by reduced output token count — the LoRA adapter learned to emit the EOS token earlier by encoding task-specific output distribution in the weights rather than relying on in-context steering from a system prompt. Yohannes confirmed his gap was closed and gave feedback that the DistilGPT-2 profiling demonstration was the strongest part of the explainer because it made the linear decode scaling relationship empirically visible rather than just theoretically asserted. No revisions were required by either partner — both explainers were accepted as written.
