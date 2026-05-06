# Day 2 — Thread

**Platform:** LinkedIn and Medium

**LinkedIn post link:** https://www.linkedin.com/posts/yohannes-dereje17_ai-artificialintelligence-share-7457920171479977984-P4Qs?utm_source=social_share_send&utm_medium=android_app&rcm=ACoAAD1aB58B1jPY2ERiFBy4PMx3Rcq9L3Ppg2E&utm_campaign=copy_link

**Medium post link:** https://medium.com/@yohannesdereje1221/when-every-document-goes-to-the-expensive-tool-cascade-collapse-in-agentic-pipelines-54e3def6178e

---

## LinkedIn Post Content

My document extraction router sent every document to the most expensive tool. Every class. Every time.

I couldn't tell if this was correct escalation or silent failure. Both look identical from the outside.

The bug was one line in all three extractors:

```python
extraction_confidence = profile.triage_confidence  # never changes
```

The escalation guard was checking a pre-extraction signal as if it were post-extraction evidence. It had no idea what the extractor actually found.

FrugalGPT calls this **cascade collapse** — a tiered system that always defaults to the expensive tier because it has no post-query evidence to accept cheaper options.

The fix: stop looking only before extraction. Look after too.

Before → After:

```
CBE_Annual_Report → Vision  ($0.04, reason unknown)
CBE_Annual_Report → FastText ($0.00, correct_primary) ✓
```

50% cost reduction. And now I can tell the difference between a correct decision and a broken default.

Full breakdown with code and paper citations in the comments 👇

*Sources: FrugalGPT (Chen et al., 2023) | ReAct (Yao et al., 2023)*
