# Signoff — Day 4

**Asker:** Natnael Alemseged
**Explainer by:** Yohannes Dereje
**Gap-closure judgment:** Closed

---

## What I understand now that I did not before

I knew my bootstrap script produced a number that felt like a p-value but I could not articulate what was actually wrong with it. Yohannes's explainer gave me the precise name for the error: my bootstrap resampled from the observed data, which is centered at the true lift (+52 pp), never at zero. A p-value must be computed under the null hypothesis — it asks "how often would I see this result if there were no real advantage?" My script instead answered "how often does a +52 pp effect randomly look like zero?" — an uninteresting question with a trivially small answer.

The concrete consequence is stark: the invalid bootstrap reports p ≈ 0.0001 while McNemar's exact test gives p = 0.0013 — a 13× inflation driven entirely by never imposing the null, not by stronger evidence.

The correct procedure is McNemar's exact test. The insight I had been missing is that the concordant pairs (both pass or both fail) carry zero information about which system is better — only the discordant pairs matter, and under the null those are a fair coin flip: Binomial(b+c, 0.5). That is the null distribution I should have been building. One line replaces the invalid computation:

```python
from scipy.stats import binomtest
result = binomtest(k=c, n=b+c, p=0.5, alternative='greater')
```

I also now understand that the bootstrap CI I was already computing is valid — the bootstrap correctly captures sampling variability around the observed lift. The error was only in using the bootstrap to generate a p-value without centering the distribution at the null. Both tools belong in the script; they answer different questions.

The grounding commit pointer is in `grounding_commit.md`.
