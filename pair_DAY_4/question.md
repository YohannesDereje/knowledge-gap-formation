# Day 4 — Question

**Topic:** Statistical evaluation and experimental design

**My Question:**

What Mathematical Property Makes Paired Bootstrapping Necessary, and What Would an Unpaired Bootstrap Have Reported?

My `paired_bootstrap` function in `ablations/run_ablations.py` (lines 271–310) resamples a single set of task indices and applies it to both the trained LoRA scores and the baseline scores, producing a 95% CI of [+18.7, +32.8] pp on the +26.4 pp lift across 48 held-out tasks. I know this is the correct procedure for a within-subject experimental design.

What I cannot explain is: what mathematical property of the experimental design makes pairing strictly necessary here, and what would the bootstrap CI have been if I had run an unpaired bootstrap on the same 48-task binary score vectors — and would the difference have been large enough to change a conclusion I present to a reviewer?

**Connection to existing artifact:**

Closing this gap would let me defend the paired bootstrap choice in `ablations/run_ablations.py` with a mechanistic justification rather than "within-subject design requires it," revise the CI notation in `methodology.md §LoRA-config` from a bare number to an annotated claim explaining why the paired interval is tighter than an unpaired one would be, and answer a reviewer who asks whether the [+18.7, +32.8] pp CI would survive if the pairing assumption were dropped.
