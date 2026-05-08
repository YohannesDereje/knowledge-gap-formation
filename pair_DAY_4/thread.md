# Day 4 — Thread

**Platform:** LinkedIn and Medium

**LinkedIn post link:** https://www.linkedin.com/posts/yohannes-dereje17_ai-artificialintelligence-activity-7458634297990848514-Y8kO?utm_source=social_share_send&utm_medium=android_app&rcm=ACoAAD1aB58B1jPY2ERiFBy4PMx3Rcq9L3Ppg2E&utm_campaign=copy_link

**Medium post link:** https://medium.com/@yohannesdereje1221/your-bootstrap-p-value-is-not-a-p-value-heres-what-to-use-instead-a72d99f8f372

---

## LinkedIn Post Content

My evaluation script reported p=0.0001 for my LoRA adapter's lift.

It was wrong. Not because the lift wasn't real — but because I was measuring the wrong thing entirely.

Here's what I did:

```python
p_value = mean(lift <= 0 for lift in bootstrap_lifts)
```

I bootstrapped 1,000 resamples from my observed data and asked how often the lift fell below zero. Tiny number. Strong evidence. Reject the null.

The problem: my bootstrap distribution was centered at the **true lift** — not at zero. I never imposed the null hypothesis. I was asking "how often does a +52pp effect look like zero?" Not "how often would we see this result if the null were true?"

Hall & Wilson (1991) called this out 30 years ago. Resampling must reflect the null hypothesis — not the observed data. Violating this can make your p-value spectacularly wrong.

The correct test for 47 paired binary task outcomes: **McNemar's exact test.**

```python
from scipy.stats import binomtest
result = binomtest(k=c, n=b+c, p=0.5, alternative='greater')
# p = 0.0013 — not 0.0001
```

Under the null, discordant pairs (tasks where the systems disagreed) split 50/50. The correct p-value was 0.0013 — still significant, but 13× less dramatic than my invalid answer.

The lesson: **a bootstrap CI and a bootstrap p-value are not the same thing.** Keep the CI. Replace the p-value with McNemar.

*Sources: Hall & Wilson (1991) | Fagerland et al. (2013)*
