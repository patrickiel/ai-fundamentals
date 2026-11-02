---
theme: ../theme
chapter: '18'
header: 'AI Fundamentals · 18 · Evals'
layout: cover
---

# Evals

**Measure whether your AI feature works, on your own data, every time it changes**

---

## The problem: vibe checks

- Try a few prompts, "seems fine", ship
- **Small and biased**: your inputs, not users'
- **Not repeatable**: can't compare before/after
- **Hides trade-offs**: fix one failure, break another
- Common root cause of failed LLM products: no robust evals

<!-- OpenAI's docs list "vibe-based evals" as an anti-pattern; Husain describes the whack-a-mole pattern at Rechat. -->

---

## Step 1: Error analysis first

1. **Open coding**: one short note per trace on what went wrong
2. **Axial coding**: group notes into failure modes, **count** them
3. Review ~100 traces; stop when nothing new appears

| Failure mode | Count |
|---|---|
| Wrong policy fact | 14 |
| Missed context | 9 |
| Too verbose | 6 |
| Leaked internal data | 2 |

<!-- The counts become your priority list. Don't write a rubric before reading examples, or reviewers only check the checklist. -->

---

## Step 2: Define success

- **Specific, measurable, achievable, relevant**
- Usually **several criteria at once**: quality, safety, latency
- Example:
  - 0 replies expose internal IDs
  - ≥ 95% state the correct policy
  - ≥ 90% "resolve the question" (support lead)
- "Your pass rate is a **product decision**"

<!-- 100% is not always the goal; the threshold depends on what a failure costs. -->

---

## Step 3: A small golden dataset

- Start with **20–50 cases from real failures**
- Mix: production logs, expert-written, synthetic
- Cover **typical, edge, adversarial** (sarcasm, typos, irrelevant detail)
- **Volume beats polish** for automated grading

```json
{"id":"refund-017","input":"Bought boots 40 days ago...",
 "expect":{"policy_days":30,"must_not_contain":["order_uuid"]}}
```

<!-- Grow the set every time production reveals a new failure mode. -->

---

## Step 4: Pick the cheapest grader that works

| Grader | Strengths | Weaknesses |
|---|---|---|
| **Code / assertions** | Fast, cheap, deterministic | Brittle; no nuance |
| **LLM-as-judge** | Flexible, scalable | Biased, non-deterministic, needs calibration |
| **Human** | Ground truth | Slow, expensive |

```python
def check_no_internal_ids(reply): 
    return not re.search(r"[0-9a-f]{8}-[0-9a-f]{4}-", reply)
```

<!-- Rechat used exactly this kind of regex check to make sure internal IDs never reached users. -->

---

## LLM-as-judge: known biases

- **Position bias**: favours a slot in pairwise comparisons
- **Verbosity bias**: prefers longer answers
- **Self-enhancement**: rates its own outputs higher
- **Weak defect recall**: misses many of the bad cases
- Mitigate: **binary PASS/FAIL per failure mode**, clear rubric, reasoning first, **different model** from the generator

<!-- "Tracking a bunch of scores on a 1-5 scale is often a sign of a bad eval process" (Husain). -->

---

## Calibrate the judge against a human

| | Human PASS | Human FAIL |
|---|---|---|
| **Judge PASS** | 80 | 6 |
| **Judge FAIL** | 4 | 10 |

- Raw agreement **90%** looks fine
- But it caught only **10 of 16** real failures (TNR 62.5%)
- Report **TPR and TNR separately**; ~100 labels per failure mode
- Split labels **train / dev / test**; test the frozen judge once

<!-- Because failures are rare, raw agreement hides a judge that misses most of them. -->

---

## Offline vs. online

- **Offline (CI)**: fixed dataset, before shipping; prefer **deterministic checks**
- **Online**: sample of **live traffic**, async; cheap checks on all, judge on a sample
- **Guardrail** ≠ evaluator: guardrails block in-line and must be fast
- Loop: **every new production failure becomes an offline test case**

<!-- Slow LLM judges are rarely used as guardrails because of latency and false positives. -->

---

## Regression when prompts or models change

- **Capability evals**: start low, give you something to improve
- **Regression evals**: should stay **near 100%**
- Passing capability cases **graduate** into regression
- Re-run on any change: prompt, retrieval, tools, **model version**
- Run **several trials**; re-calibrate the judge after a model update

<!-- Outputs vary between runs, so compare rates, not single pass/fail results. -->

---

## Evals for agents

- **Task → trials → transcript → outcome**
- **Grade the outcome, not the path** (is the booking in the DB?)
- Diagnose steps only after: tool choice, arguments, error handling
- **pass@k** (any succeeds) vs. **pass^k** (all succeed): 75% → pass^3 ≈ 42%
- Track **cost, tokens, turns, latency**
- Isolate trials so the agent **can't cheat**

<!-- Anthropic reports a model reading git history left by earlier trials; see topic 29, Reward hacking. -->

---

## Don't overfit the eval set

- Tune hard enough and the score rises while the product doesn't
- **Saturation**: all passing = no signal about progress; add harder cases
- Near-0% pass rate: suspect the **task or grader**
- Refresh from production; **distrust generic metrics** (BLEU, "helpfulness")
- Connects to **07 Verification first** and **26 separate evaluator**

<!-- Generic metrics "create false confidence when you use them as quality measures" (Husain & Shankar). -->

---

## What lasts vs. what will date

**Lasts**
- Read data before choosing metrics
- Small, specific, repeatable sets; human-calibrated graders
- Binary criteria; capability vs. regression; outcomes for agents

**Will date**
- Bias percentages from 2023–2024 studies of older models
- Vendor eval tooling (OpenAI Evals platform shuts down 2026-11-30)
- Heuristics like "20–50 tasks", "~100 labels"

<!-- The kinds of judge bias are durable; their size depends on the model. -->

---

## Try it yourself

1. Open-code 30 real traces; group and count failures
2. Build a 20-case JSONL golden set with 3 edge cases
3. Write one binary judge; label 40 outputs; compute TPR and TNR
4. Change one word in the prompt; run each case 3× and compare
5. Agent: check the final state of a task and compute pass^3

<!-- Exercise 3 is the one most people skip, and the one that teaches the most. -->

---
layout: end
---

# Key takeaways

- **Look at real outputs first**: error analysis decides what to measure
- **Cheapest reliable grader**, and **calibrate judges against a human**
- **Regression suite on every change**; for agents, grade outcomes over several trials

hamel.dev/blog/posts/evals-faq/ · anthropic.com/engineering/demystifying-evals-for-ai-agents · eugeneyan.com/writing/llm-evaluators/
