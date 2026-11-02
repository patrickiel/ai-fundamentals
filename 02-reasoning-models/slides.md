---
theme: ../theme
chapter: '02'
header: 'AI Fundamentals · 02 · Reasoning models'
layout: cover
---

# Prompting Reasoning Models

**They already think step by step, so give them short, structured prompts with clear constraints and goals, and use them to plan and judge**

---

## What's different about a reasoning model

- Generates **internal reasoning tokens** before the answer
- Better at planning, multi-step problems, tool use
- Reasoning tokens are **billed** and **take up context**, even when hidden
- OpenAI suggests reserving **~25,000 tokens** for reasoning + output when starting out
- Roots: **chain-of-thought** prompting (Wei et al., 2022); **reinforcement learning** for reasoning (DeepSeek-R1, 2025)

<!-- Sources: OpenAI's reasoning models guide; Wei et al.; the DeepSeek-R1 paper. -->

---

## 1. Simple and direct

- The models "excel at understanding and responding to **brief, clear instructions**" (OpenAI)
- Cut filler, pleasantries and padding
- Every extra sentence is something else to interpret
- OpenAI: reasoning models do well with **high-level guidance**; GPT models need precise instructions

<!-- OpenAI's prompt engineering guide compares a GPT model to a junior coworker who needs explicit instructions. -->

---

## 2. Don't tell it how to think

- "Think step by step" is **redundant**; OpenAI says it "can sometimes hinder" performance
- Anthropic: a general "think thoroughly" often beats a **hand-written step-by-step plan**
- Some current Claude models may **decline** requests to write reasoning into the answer; rely on built-in thinking (as of 2026-10)

<!-- Chain-of-thought prompting helped older models; reasoning models do this internally. -->

---

## 3. Structure with delimiters

```markdown
## Context
<background, audience, facts>

## Examples
<optional; only if zero-shot missed>

## Instructions
<task, constraints, goal>
```

- Markdown, XML tags or section titles
- Removes guessing about **what each block is for**

<!-- OpenAI's guide recommends delimiters to clearly indicate distinct parts of the input. -->

---

## 4. Zero-shot first?

| Source | Advice on examples |
|---|---|
| OpenAI | Try **zero-shot first**; examples must match instructions closely |
| DeepSeek-R1 paper | Few-shot **consistently degraded** R1; use zero-shot |
| Anthropic | Multishot examples **do help** Claude's thinking |

**Start without, add if needed, keep consistent, measure on your model**

<!-- The vendors disagree here, so treat it as something to test rather than a rule. -->

---

## 5. Constraints and the end goal

- **Specific guidelines:** "propose a solution with a budget under $500"
- **Specific end goal:** clear criteria for success; keep reasoning until met
- Add the **why**: "…for a beginner who may upgrade later"
- For agents: define what **"done"** means and **how to check** it

<!-- Both tips are in OpenAI's best-practices guide; the definition-of-done advice is in its reasoning models guide. -->

---

## 6. Self-checks, with care

- Anthropic: "Before you finish, verify your answer against [test criteria]"
- Catches errors reliably for **code and maths**
- But some newer models verify on their own; inherited checks can cause **over-verification**
- **Re-test prompts** when you change models

<!-- Anthropic's docs flag over-verification on some newer models as of 2026-10. -->

---

## API notes (as of 2026-10)

| Point | Detail |
|---|---|
| Developer messages | Replace system messages for OpenAI reasoning models (from o1-2024-12-17) |
| Markdown | Off by default; first line `Formatting re-enabled` turns it on |
| Effort knobs | OpenAI `reasoning.effort` · Anthropic `effort` + adaptive thinking · Google `thinking_level` |
| Output caps | Include thinking, so a low cap can cut the answer |

"Treat `reasoning.effort` as a **tuning knob**, not the primary way to recover quality" (OpenAI)

<!-- Google suggests low thinking for simple retrieval, default for moderate tasks, highest for hard maths, coding and planning. -->

---

## Planners and workhorses

- **Reasoning models = planners:** strategy, ambiguous inputs, decisions
- **GPT models = workhorses:** low latency and cost, straightforward execution

```
Fast model(s)    → gather and triage data        (volume)
Reasoning model  → plan, decide, or check        (where errors cost most)
Fast model(s)    → carry out routine steps       (volume)
```

- OpenAI's example: GPT-4o / 4o mini triage a return request → **o3-mini decides**
- OpenAI cookbook: **o1 turns policy articles into routines** for a cheaper agent

<!-- Put the careful model where mistakes are costly. Topics 24 and 26 build this into full harnesses. -->

---

## When to reach for a reasoning model

1. **Ambiguous** tasks (asks before guessing)
2. **Needle in a haystack** (a clause in a footnote)
3. **Relationships** across hundreds of pages
4. **Multi-step agentic planning**
5. **Visual reasoning** (reading drawings and legends)
6. **Code review and debugging** (often in the background)
7. **Evaluating** other models' outputs

Simple, high-volume tasks: a fast model or **low effort** usually wins

<!-- These are the seven patterns OpenAI lists in its best-practices guide; it says the list isn't exhaustive. -->

---

## What lasts vs. what will date

**Lasts**
- Don't prescribe how to think; give inputs, constraints, goal
- Delimit the parts of a prompt
- Examples: add when needed, keep consistent, measure
- Careful models plan and judge; fast models execute

**Will date**
- Model names and which ones think by default
- Effort parameters, output caps, `Formatting re-enabled`
- Vendor differences on examples

<!-- Re-check API details against current docs before relying on them. -->

---

## Try it yourself

1. Same prompt with and without "think step by step"
2. Restructure a messy prompt into Context / Examples / Instructions
3. Zero-shot, then add two matching examples
4. Add the goal behind a constraint
5. Run easy and hard tasks at low vs. high effort

<!-- Exercise 5 shows where extra thinking pays off and where it only adds latency. -->

---
layout: end
---

# Key takeaways

- **Short, structured prompts** with explicit constraints and a clear goal
- **Don't tell it how to think**; tune effort only after the prompt is right
- **Planners and workhorses**: spend expensive reasoning where errors cost most

developers.openai.com/api/docs/guides/reasoning-best-practices\
platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices\
arxiv.org/abs/2501.12948
