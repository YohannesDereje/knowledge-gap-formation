# Day 4 — Evening Call Summary

**Participants:** Yohannes Dereje and Natnael Alemseged
**Duration:** ≥45 minutes

---

## Yohannes's explainer → Natnael's gap

Yohannes walked Natnael through the bootstrap p-value explainer, covering Hall & Wilson's Guideline 1 — that resampling
must reflect the null hypothesis, not the observed data — the 2×2 contingency table structure of paired binary outcomes,
and why McNemar's exact test using Binomial(b+c, 0.5) on discordant pairs only is the correct null distribution,
producing a calibrated p=0.0013 versus the invalid bootstrap's p=0.0001 which is 13× smaller. Natnael confirmed the gap
was closed and gave feedback that the courtroom analogy in Part 1 and the thermometer analogy in Part 6 were the
clearest moments because they made the "under the null" requirement feel inevitable rather than technical. No revisions
were required.

**Gap closure: Closed**

---

## Natnael's explainer → Yohannes's gap

Natnael walked Yohannes through the paired bootstrap explainer, explaining that pairing is justified by experimental
design — the same 48 tasks evaluated under both systems — and that the variance reduction from pairing depends on the
covariance between the two score vectors, which in Yohannes's data is near-zero (r=0.167), making the paired and
unpaired CIs essentially identical at [+35.4, +68.8] pp and [+35.4, +66.7] pp respectively — both with lower bounds far
above zero. Yohannes confirmed his gap was closed and gave feedback that the empirical simulation comparing paired and
unpaired bootstrap output directly on the held-out traces was the most actionable section because it gave a concrete
number to present to a reviewer. No revisions were required.

**Gap closure: Closed**

---

**Confirmed by:** Natnael Alemseged

The above summary accurately reflects the call on both sides. On my gap: the courtroom and thermometer analogies were
the sections I found clearest — they made the "under the null" requirement feel like a logical necessity rather than a
technicality, and I can now name exactly what was wrong with the original line, why McNemar's test is the correct
replacement, and defend both to a reviewer. On Yohannes's gap: the feedback I gave was that the empirical simulation was
the strongest section because it put a concrete number on the paired vs. unpaired difference, and I confirmed that the
design justification section correctly came before the variance-reduction formula — pairing is correct by design before
any formula applies — which was the core distinction Yohannes needed to close his gap, as reflected in his signoff.
