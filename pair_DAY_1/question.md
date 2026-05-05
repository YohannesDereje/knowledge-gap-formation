# Day 1 — Question

**Topic:** Inference-time mechanics

**My Question:**

Why Does a Fine-Tuned LoRA Adapter Generate Outputs 3.2× Faster Than the Baseline?

In my Tenacious-Bench ablation results (`ablations/ablation_results.json`, visible in the Inference Speed chart on the Ablation Results page), the average inference latency drops from 10,652ms (baseline) to 8,421ms (prompt engineering only) to 3,266ms (trained LoRA adapter) — a 3.2× speedup. I assumed that adding a system prompt and then a LoRA adapter would add computational overhead and make inference slower, since both add more "work" to the pipeline.

What is the specific mechanism inside autoregressive text generation — particularly the relationship between the prefill and decode phases — that explains why a more constrained or fine-tuned model generates outputs faster than an unconstrained baseline on identical hardware, and why does this matter when choosing between prompt engineering and fine-tuning as a production deployment strategy?

**Connection to existing artifact:**

Closing this gap would let me correctly explain the Inference Speed chart in my ablation results page, revise the latency claim in my README ("3.2× faster than baseline at inference") with the actual mechanism behind it rather than just asserting the number, and defend the production recommendation on Slide 8 of my standup presentation.
