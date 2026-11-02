---
theme: ../theme
chapter: '24'
header: 'AI Fundamentals · 24 · Advisor strategy'
layout: cover
---

# The Advisor Strategy

A **cheaper executor** does the work and consults a **stronger advisor** only at the decisions that matter.

---

## The problem

- Strong models reason better but cost more and run slower
- Cheaper models handle **routine steps** well, but can go wrong on hard calls
- In a long task, **most turns are routine**; a few decide the outcome
- Claude Code's cost guide: Sonnet for most coding, reserve Opus for hard reasoning

<!-- The question is how to apply "reserve Opus for hard decisions" inside a single task rather than per session. -->

---

## Three ways to combine models

| Approach | Who decides | Strong model produces |
|---|---|---|
| **Routing** | A router, before the task | The full answer |
| **Cascade** | A check on the cheap answer | The full answer, if needed |
| **Advisor** | The executor, mid-task | **Guidance only** |

<!-- Routing: "Building effective agents" and RouteLLM (learned routers, >2x cost cut in some cases). Cascade: FrugalGPT (up to 98% cost reduction matching the best single model on its benchmarks). The advisor keeps the cheap model in charge throughout. -->

---

## How it works

- **Executor** (e.g. Sonnet, Haiku): tools, edits, final output
- Hits a hard decision, a recurring error, or "am I done?" → calls the **advisor**
- **Advisor** (e.g. Opus) reads the full transcript, returns "a plan, a correction, or a stop signal"
- The advisor "never calls tools or produces user-facing output"
- Inverts the usual pattern of a strong orchestrator over cheap workers

<!-- Anthropic's multi-agent research system is the opposite shape: an Opus lead delegating to Sonnet subagents. -->

---

## In the API (as of 2026-10)

- Beta header `advisor-tool-2026-03-01`; tool `type: "advisor_20260301"`, `name: "advisor"`, `model`
- Executor sends an **empty** call: it "signals timing, and the server supplies context"
- Advisor runs **without tools**; only advice text returns; one `/v1/messages` request
- `max_uses` per **request**; `max_tokens` ≥ 1024 per call
- Advisor tokens billed at advisor rates, reported in `usage.iterations`

<!-- Advice is typically 400-700 text tokens. "The cost savings come from the advisor not generating your full final output." The advisor must be at least as capable as the executor. -->

---

## In Claude Code (as of 2026-10)

- `/advisor opus`, the `advisorModel` setting, or `claude --advisor opus`
- **Experimental**; Anthropic API only (not Bedrock, Google Cloud, Foundry)
- Claude decides when: before committing to an approach, on recurring errors, before "done"
- No setting to force or cap calls; **ask** in your prompt
- Each call re-reads the full transcript; the advisor's read isn't cached

<!-- If evidence contradicts the advice, Claude surfaces the conflict instead of following blindly. Related options: opusplan, subagents with their own model, /model. -->

---

## What the evidence says

- SWE-bench Multilingual: Sonnet + Opus advisor **+2.7 points** vs. Sonnet alone, **−11.9%** cost per task
- BrowseComp: Haiku + Opus advisor **41.2%** vs. 19.7% alone
- Sonnet at **medium effort** + Opus advisor ≈ Sonnet at default effort, at lower cost
- "Results are task-dependent. Evaluate on your own workload."

<!-- Vendor benchmarks, Opus 4.6 as advisor, April 2026. Anthropic recommends comparing three setups on your own evals: cheap alone, cheap + advisor, strong alone. Topic 18 covers evals. -->

---

## When it fits

- Long, multi-step tasks where **plan quality decides the outcome**
- Debugging with a **recurring error**; a diagnosis unlocks mechanical fixes
- Work you want **independently checked** before "done"

**Poor fits:** single-turn tasks · every turn needs the top model · users already choose their own model

<!-- From the Claude Platform and Claude Code advisor docs. -->

---

## Limit 1: escalation

- The executor **decides when to ask**; timing is "model-driven rather than rule-based"
- It may judge a hard task routine and **skip the advisor**
- Suggested prompt: call "BEFORE substantive work", when stuck, before declaring done
- Nudges help unevenly: Haiku **+~7 points**, Sonnet no effect, Opus slightly worse

<!-- Write explicit triggers: "consult the advisor before changing the data model, before a new feature, and after two failed attempts." -->

---

## Limit 2: decisions, not execution

- The advisor has **no tools** and writes **no code**
- Speed, thoroughness and work-splitting still depend on the **executor**
- If the hard part is lots of interdependent work, not a few decisions,
  a **strong main model** may finish sooner with fewer wrong turns

<!-- This follows from the design: guidance improves the plan, the executor still carries it out at its own level. -->

---

## What lasts vs. what will date

**Lasts**
- Spend expensive reasoning where it **changes the outcome**
- Separate deciding from doing; escalation is only as good as its **trigger**
- Routing, cascades, advisors: pick by **when** and **what** the strong model contributes

**Will date**
- Tool name, header, parameters, pairing rules; `/advisor` status (as of 2026-10)
- Model pairings and benchmark numbers (April 2026)

---

## Try it yourself

1. Mark each turn of a session as routine or judgement-heavy
2. Manual advisor: after two failures, ask a stronger model for a plan only
3. Compare cheap / cheap + advisor / strong on one task (`/usage`)
4. Add explicit escalation triggers and rerun
5. Find a task where advice didn't save the executor: decision or execution?

<!-- Exercise 3 mirrors Anthropic's own recommended evaluation. -->

---
layout: end
---

# Key takeaways

- **Cheap executor, strong advisor**: the advisor guides, the executor does the work
- **Escalation is the weak spot**: write explicit triggers
- **Measure on your workload**: cheap vs. cheap + advisor vs. strong

claude.com/blog/the-advisor-strategy\
platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool\
code.claude.com/docs/en/advisor
