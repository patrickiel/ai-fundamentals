---
theme: ../theme
chapter: '26'
header: 'AI Fundamentals · 26 · Agent harness design'
layout: cover
---

# Agent Harness Design

**Plan → Generate → Evaluate, and never let the doer be the judge.**

---

## The problem

- Long tasks need **scaffolding** around the model: roles, plans, checks, files
- Scaffolding built for older models can become **overhead**
- Extra tokens and steps, and constraints that block what the model can now do
- Which parts still help? **Measure, don't guess**

<!-- Anthropic's earlier advice for agentic systems: find the simplest solution possible and increase complexity only when needed. -->

---

## Components are assumptions

> "Every component in a harness encodes an assumption about what the model can't do on its own."

- Assumptions can be **wrong**, and they **go stale** as models improve
- Method: **ablate**. Remove one part, run, compare, keep it only if results drop
- Radical simplification failed; **one-at-a-time** removal worked
- The useful design space "doesn't shrink as models improve. Instead, it moves"

<!-- Quotes from Prithvi Rajasekaran, "Harness design for long-running application development" (Anthropic, March 2026). -->

---

## Planning: product level, not implementation

- Planner expands a 1–4 sentence prompt into a **product spec**
- Prompted to be **ambitious about scope**
- Stays at product context and high-level design
- Why: errors in a detailed spec **cascade** into everything built on it
- Removing the planner → the generator **under-scoped**

<!-- The earlier long-running harness used a feature list of 200+ user-facing features, all initially failing, worked through one at a time. -->

---

## Separate the doer from the judge

- Agents "reliably **skew positive** when grading their own work"
- Worst on subjective work like **UI design**
- A skeptical standalone evaluator is **far more tractable** to tune than a self-critical generator
- Claude Code docs: a fresh context reviews better, "not biased toward code it just wrote"

<!-- Separation doesn't remove leniency on its own: the evaluator is still a model inclined to be generous. It does give you a component you can tune. -->

---

## The three roles

| Role | Does |
|---|---|
| **Planner** | Writes a product-level spec to a file |
| **Generator** | Builds feature by feature, commits to git, self-checks, hands off |
| **Evaluator** | Reads the spec, **uses the running app** (Playwright MCP), grades, sends feedback |

- Supporting habits: progress file, `init.sh`, start-of-session smoke test

<!-- The supporting habits come from Anthropic's "Effective harnesses for long-running agents" (Nov 2025). -->

---

## Tuning the evaluator

- "Out of the box, Claude is a **poor QA agent**"
- Found real issues, then **approved anyway**
- Tested **superficially**, missed edge cases
- Fix: read its logs, compare with human judgement, revise the prompt
- Took **several rounds** before grading was reasonable

<!-- The evaluator is a component like any other: it needs its own iteration loop. -->

---

## Graded evaluation: define "good"

| Criterion | Checks | Weight |
|---|---|---|
| **Design quality** | A coherent whole with a distinct mood | Higher |
| **Originality** | Deliberate choices, not defaults or AI patterns | Higher |
| **Craft** | Typography, spacing, colour, contrast | Lower |
| **Functionality** | Users find and understand primary actions | Lower |

- Claude already did well on craft and functionality, so weight went **where it was weak**

<!-- The evaluator was calibrated with few-shot examples showing score breakdowns. The post calls out purple gradients over white cards as a typical AI default. -->

---

## Why scores beat pass/fail

- A score per criterion shows **where** the work is weak
- Weights say **what matters most**
- **Hard thresholds**: any criterion below its bar fails the chunk
- Pass/fail against the plan can approve work that is on-spec but poor
- This is the **evaluator-optimizer** workflow: clear criteria, measurable gains

<!-- Rubric grading is also offered in Claude Managed Agents (beta, as of 2026-10). -->

---

## Sprint contracts: model-dependent

- Before each chunk: generator **proposes** what it will build and how to test it
- Evaluator reviews; they iterate until they **agree**, via files
- Turns a product spec into **checkable steps**
- With Opus 4.6: **sprint construct removed**
- Planner and evaluator kept; evaluation moved to **one pass at the end**

<!-- The evaluator "is worth the cost when the task sits beyond what the current model does reliably solo." -->

---

## Context anxiety and resets

- **Context anxiety**: wrapping up early near the perceived context limit
- **Compaction**: summarise in place. Continuity, but anxiety can persist
- **Reset**: fresh agent + structured handoff in files. Clean slate, more overhead
- Sonnet 4.5 needed resets; Opus 4.5 "**largely removed** that behavior"
- Keep the **diagnostic**: watch for premature "done"

<!-- The earlier harness saw later sessions "declare the job done" after seeing that some progress had been made. -->

---

## What it costs

| Run | Time | Cost |
|---|---|---|
| Retro game maker, solo (Opus 4.5) | 20 min | $9 |
| Retro game maker, full harness | 6 hr | ~$200 |
| Audio workstation, simplified harness (Opus 4.6) | ~4 hr | ~$125 |

- Solo version looked fine, but **nothing responded to input**
- QA rounds cost a few dollars each and still caught real gaps

<!-- Figures from the harness-design post. Worth it only when the task is beyond what the model does reliably alone. -->

---

## What lasts vs. what will date

**Lasts**
- Components are assumptions: **ablate** them, again with each new model
- Separate doing from judging; tune a **skeptical** evaluator
- Plan outcomes, not keystrokes; make "good" **explicit**

**Will date**
- Which model needs which part (Sonnet 4.5, Opus 4.5, 4.6)
- Costs, stack, Playwright/Puppeteer MCP, Managed Agents rubrics

<!-- All model and product details as of 2026-10. -->

---

## Try it yourself

1. **Ablate** one component of your agent setup and compare results
2. Same-session self-rating vs. fresh skeptical evaluator with a rubric
3. Write a **four-criterion rubric** with weights and thresholds
4. Rewrite a technical task list as **product-level** user stories
5. Watch for premature "done"; add a feature list and progress file

<!-- Exercise 2 usually shows the leniency gap within minutes. -->

---
layout: end
---

# Key takeaways

- Every harness part is an **assumption**: test it by removing it
- **Separate the doer from the judge**, and grade against **weighted criteria**
- Plan the **product**; keep durable state in **files and git**

anthropic.com/engineering/harness-design-long-running-apps · anthropic.com/engineering/effective-harnesses-for-long-running-agents
