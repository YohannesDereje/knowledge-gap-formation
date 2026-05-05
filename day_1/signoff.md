# Day 1 — Sign-Off

**Asker:** Mamaru Yirga

**Gap closure status:** Closed

**What I understand now that I didn't before:**
Before reading this explainer, I knew the 2× ratio existed but treated it as an arbitrary business decision copied from OpenRouter's pricing page. Now I understand the load-bearing mechanism: prefill is parallel matrix-matrix multiplication (compute-bound, 1024 OPs/byte arithmetic intensity) while decode is sequential matrix-vector multiplication (memory-bound, 1 OP/byte). The pricing ratio reflects that decode GPU time is genuinely less productive per token because the GPU sits mostly idle waiting for memory bandwidth. The specific insight that closed my gap: my typical call shape (700 input, 175 output) is decode-dominated in cost (55.6% output despite 4× fewer output tokens) because Claude's actual pricing is 5× (not 2×), and the ClaudeClient bug was underestimating output cost by 2.5× on every eval call. I can now defend the $3.53/lead figure in my CFO memo with mechanism knowledge and know the optimization target: either constrain output length or exploit prefix caching for the shared system prompt. The factory analogy (10,000 workers during prefill, only 1 active during decode) made the sequential dependency concrete, and the worked example using my actual token counts showed me the bug's real impact on my system.
