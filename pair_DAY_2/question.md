# Day 2 — Question

**Topic:** Agent and tool-use internals

**My Question:**

Why Does a Constraint in the System Prompt Fail While the Same Constraint in a Follow-Up User Turn Succeeds?

In `agent_core.py` Phase 5 (~lines 461-503), SOC-01 proved that embedding the honesty constraint in the system prompt was not sufficient — the model generated "you are rapidly scaling your team" despite an explicit instruction not to. The fix was to inject a follow-up user turn mid-conversation naming the specific violation and requesting a rewrite, and that second attempt passed.

At the token level, what is different about a constraint carried in the system prompt versus the same constraint arriving as a new user turn after the initial generation? Is this recency, role-tag weighting, or something about how instruction-tuned models process competing signals during decoding?

**Connection to existing artifact:**

Closing this gap would let me revise `probes/probe_library.md` SOC-01 from "prompt-only constraint proved insufficient" — which is just an observation — to an actual mechanistic explanation of the failure mode, and let me defend why the follow-up user turn injection is not a workaround but the architecturally correct fix given how instruction-tuned models weight competing signals during decoding.
