# Sign-off — Day 3

**Asker (signing off):** Yohannes Dereje
**Explainer (received from):** Atnabon Deressa
**Topic:** Routing vs transformation under Q+V-only LoRA
**Date:** 2026-05-07

---

## Judgement

- [x] **Closed** — gap is closed; I can now defend the mechanism unaided.
- [ ] **Partially closed**
- [ ] **Not closed**

## What I understand now that I did not before

My gap is closed because I now understand that the 3 residual SOC failures in
`ablations/held_out_traces.jsonl` were not a data problem — they were a
configuration problem with a precise mechanistic explanation. Before this
explainer, I was reading those failures as noise: "the model still has some SOC
bugs despite 180 training pairs." I could not explain why all 3 failures shared
the exact same pattern: `regex_negative` passing and `regex_positive` failing.
That pattern felt like a coincidence. I now understand it is a diagnostic
signature.

The concept that closed the gap was the **routing vs transformation cut**
between the attention layers and the FFN layers. Attention (Q, K, V, O
projections) performs information routing — it decides which tokens get attended
to and what values flow forward through the layer. The FFN performs information
transformation — it operates as a per-token key-value memory (Geva et al. 2021)
that composes specific token sequences into next-token logits. These are two
different computational jobs, and LoRA on only Q and V can only update the
routing job. It can teach the model which signals to attend to less (suppression
of banned phrases — a routing behavior). It cannot teach the FFN to compose new
required output phrases like `"curious whether"` or `"haven't seen"` that the
base model did not already produce reliably — that is a transformation behavior.

This makes the `regex_negative` / `regex_positive` split in my rubric read as
diagnostic, not noisy. `regex_negative` measures routing success: did the model
stop attending to and propagating banned velocity patterns? Q+V LoRA can learn
this, and it did — all 3 failing tasks pass `regex_negative`. `regex_positive`
measures transformation success: did the FFN compose the required hedged phrases
at the right positions? Q+V LoRA cannot learn this, because it never touches the
FFN key-value memory. That is why all 3 failing tasks fail `regex_positive`. The
failure pattern is mechanistically predicted by the configuration choice.

The second thing this explainer corrected was my trust in the Hu et al. (2021)
Q+V default. I had treated it as a general recommendation. I now understand it
was derived from GLUE benchmarks — classification and ranking tasks that are
suppression-flavored by design. They did not test generative production of
required output substrings against a regex positive-pattern check. The Q+V
default is the right choice for routing tasks. It is the wrong choice for
generative tasks that require new phrase composition. My task — producing
specific hedged acknowledgment phrases conditional on `signal_confidence=Low` —
is the second kind.

The actionable fix is to add `gate_proj`, `up_proj`, and `down_proj` to
`LORA_TARGETS` in `training/train.py`. The grounding commit reflects this
understanding: the comment on line 37 now names the routing vs transformation
distinction and flags FFN targets as the correct remediation for the 3 residual
SOC failures, and `methodology.md §1` now defends the target module choice —
or honestly acknowledges where it was under-specified — with reference to the
mechanism.

## Residual gap

Once FFN targets are added, does the rank choice (`r=16`) need to scale with
the larger adapter parameter count, or does rank-vs-target-modules decouple in
practice? That follow-up is exactly Atnabon's Day 3 question on rank — the two
questions are deliberately complementary.
