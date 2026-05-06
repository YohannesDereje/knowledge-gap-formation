# Morning Call Summary

During the morning call, we each explained our Week 12 questions and clarified the gaps we were trying to investigate.

For my Question, I walked my peer Nuhamin through the SOC-01 failure in detail. She summarized her understanding back to me, and we agreed that My question is about the timing and level of instruction injection. Specifically, I wanted to understand why an honesty constraint embedded in the original system prompt was ignored during the first generation, while a later follow-up user turn that explicitly named the violation successfully triggered a corrected rewrite.

For my peer's Nuhamin's question, She explained that she found the Document Intelligence Refinery a better fit for “Agent and tool-use internals” theme because it contains a concrete multi-tool routing problem: deciding between FastText/pdfplumber, Layout-Aware/Docling, and Vision-Augmented extraction. We dicussed the key gaps:  understanding how an agentic extraction router distinguishes correct escalation from bad tool-selection collapse, and what evidence it should log to make each tool decision auditable.
