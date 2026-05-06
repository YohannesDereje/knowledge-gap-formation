# Day 2 — Sign-Off

**Asker:** Nhuamin ALemayehu

**Gap closure status:** Closed

**What I understand now that I didn't before:**

My gap is closed because I now understand that the extraction router problem was not simply “all documents went to `VisionAugmented`.” The deeper issue was that the pipeline could not prove whether Vision was the correct escalation or a tool-selection collapse. Before this explainer, I was thinking mostly in terms of the initial document profile: character density, whitespace ratio, layout complexity, and whether a document looked scanned or table-heavy. I now understand that those are only **Gate 1** signals: they help decide which extractor to try first, but they do not prove whether that extractor worked.

The concept that closed the gap for me was **Gate 2**: the post-extraction quality check. Gate 2 happens after an extractor runs. It asks, did this tool actually produce usable output? It should measure evidence such as content blocks returned, non-empty text ratio, bounding-box coverage, table extraction, provenance coverage, validation failures, and remeasured extraction confidence. This matters because pre-extraction triage confidence and post-extraction output quality are different things. A document may look difficult but still be handled well by FastText, or it may look simple but produce broken extraction. Without Gate 2, the router can only say which tool it used. With Gate 2, the router can explain whether the tool worked, whether escalation was justified, and whether ending up at `VisionAugmented` was correct escalation or unnecessary collapse.

The grounding commits reflect this understanding. I added extraction-quality utilities, updated the FastText, Layout-Aware, and Vision extractors to compute quality from actual output, extended the Pydantic models to carry quality evidence, and updated the extraction rules configuration. I can now explain the router as a three-gate tool-use system: **Gate 1 chooses the initial tool, Gate 2 validates the tool’s actual output, and Gate 3 accepts or escalates based on evidence.** This gives me a clearer and more defensible model of agentic tool routing in the Document Intelligence Refinery.
