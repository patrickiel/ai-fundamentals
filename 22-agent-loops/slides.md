---
theme: ../theme
chapter: '22'
header: 'AI Fundamentals · 22 · Agent loops'
layout: cover
---

# Agent Loops

Every loop is **goal → action → check → stop**. Pick the loop whose **check** you can trust.

---

## What a loop is

- Agents are "LLMs using tools based on environmental feedback **in a loop**" (Anthropic)
- ReAct: interleave **reasoning** and **actions**; each observation updates the plan
- Each step needs **ground truth** from the environment: test results, command output
- Always add a **stopping condition**, such as a maximum number of iterations
- Loop engineering moves "what next?" from **you** into the **system**

<!-- Roots: "Building effective agents" (Dec 2024) and the ReAct paper (2022). In a turn-based session you are the control system; a loop hands that job to the agent plus a check. -->

---

## The shared anatomy

| Part | Question | Example |
|---|---|---|
| **Goal** | What is "done"? | All tests in `test/auth` pass |
| **Action** | What happens each round? | Edit code, revise a skill |
| **Check** | How is it judged? | Tests, separate model, rubric |
| **Stop** | When does it end? | Check passes, turn cap, impossible |

Plus two properties: **state** (what persists between rounds) and **who checks**

<!-- If you can't name the check, you don't have a loop yet, just an agent that runs until it feels finished. -->

---

## Two ways to classify loops

**By trigger** (Anthropic's guide, as of 2026-10):
turn-based (you) · goal-based (`/goal`) · time-based (`/loop`, `/schedule`) · proactive (routines)

**By check and state** (this course's teaching framework):
single-task · learning · multi-critic review · build-and-score · process-improvement

> "The more quantitative the checks are, the easier it is for Claude to self-verify."

<!-- The trigger axis is Anthropic's official framing. The five patterns are a way to organise the root patterns (evaluator-optimizer, voting, Reflexion, Ralph), not an official taxonomy. -->

---

## 1. Single-task loop: Ralph and `/goal`

- **Ralph** (Huntley): `while :; do cat PROMPT.md | claude-code ; done`
  - Fresh context each pass, **one item per loop**, state in files (`fix_plan.md`, `AGENT.md`)
  - "Back pressure" from tests and type checkers rejects bad output
- **`/goal`** (Claude Code): after each turn a **separate small model** judges your condition: not met / met / impossible
  - The evaluator reads only the conversation; it runs no commands

<!-- Ralph is stateless in context, not in the repo. /goal is a wrapper around a session-scoped prompt-based Stop hook (as of 2026-10). Because the evaluator can't run tests itself, write conditions Claude's own output can demonstrate. -->

---

## Make the check objective

- A model's opinion that it's done is the **weakest** check
- **Write tests first**, then: "make these pass; don't modify the tests"
- Tests for other features act as **guardrails**
- Without this, agents declare victory too early (Anthropic harness research)
- **Checkpoint**: commit after each passing run so the loop can roll back

<!-- Good /goal conditions: one measurable end state, a stated check, constraints that must not change, and a turn cap such as "or stop after 20 turns". -->

---

## 2. Learning loop

- Goal: **improve a reusable asset** (skill, prompt, workflow)
- **Baseline**: run the task *with* and *without* the asset
- **Journal** next to the asset (`learning.md`): tried, result, lesson
- Root: **Reflexion**, agents write reflections to an episodic memory and read them later
- Ralph's `AGENT.md` collects build-and-run learnings the same way

<!-- Without a baseline you can't tell whether a skill helps. Topic 18 covers evals. The journal stops the loop repeating failed experiments. -->

---

## 3. Multi-critic review loop

- One reviewer covering everything **misses things**
- Split review into **specialised critics**: facts, relevance, safety, style
- Root: **parallelization by voting** (several prompts review the same code)
- Related: Karpathy's *llm-council*: answer, anonymous peer ranking, chairman
- Orchestrator keeps round history in one place; peer-to-peer gives more independent views

<!-- llm-council is an unsupported weekend project, useful as inspiration. Topic 23 compares orchestrator-led subagents with agent teams that message each other. -->

---

## 4. Build-and-score loop

- One agent **builds**, another **scores** against a rubric
- Root: **evaluator-optimizer**, best "when we have clear evaluation criteria"
- **Separate the judge**: a fresh-context reviewer is less biased
- **No write access** for the scorer
- Persist scores (e.g. JSON) to see convergence or stalling

<!-- Heavy fan-out reviewers are worth it on large finished codebases. For everyday work one reviewer with a clear rubric keeps the structure at much lower cost. -->

---

## 5. Process-improvement loop

- Adds a step: **critique the process**, not just the output
- Change prompts, rubric, step order, checks
- Anthropic: when a result misses the standard, **encode the fix into the system**
- Huntley: each failure gets a new "sign" in the prompt; Ralph "gets tuned - like a guitar"
- Keep a **human approving** process changes

<!-- A loop that rewrites its own rubric can drift toward what is easy to score. Topic 29 covers reward hacking. -->

---

## Choosing a loop

| Need | Loop | Check |
|---|---|---|
| Finish a specified task | Single-task | Tests, build, validator |
| Improve a reusable asset | Learning | With/without, logged |
| Judge many qualities | Multi-critic | Specialised critics |
| Push toward a target | Build-and-score | Read-only scorer + rubric |
| Improve the process | Process-improvement | Score + reviewed critique |

**Start with the cheapest loop whose check you trust.**

<!-- Cost scales with agents times rounds. Anthropic: pilot before a large run; model and effort level are among the biggest cost levers. -->

---

## What lasts vs. what will date

**Lasts**
- A loop is only as good as its **check**; always have a cap
- Separate judge from builder; scorers can't edit
- State in **files and git**, not the context window

**Will date**
- `/goal`, `/loop`, `/schedule`, routines and their behaviour (as of 2026-10)
- The four-way trigger taxonomy; Ralph's specific files

<!-- Principles go back to ReAct (2022) and the evaluator-optimizer pattern (2024); the commands are 2026 product details. -->

---

## Try it yourself

1. Tests-first `/goal` with a turn cap; note rounds and tokens
2. Write down goal, action, check, stop, state for one workflow
3. Run a skill 5× with and 5× without; start a `learning.md`
4. Review one document with 3 critics vs. 1 generic prompt
5. Build with one session, score with a separate read-only session

<!-- Exercise 2 is the fastest win: if you can't name the check, fix that before adding autonomy. -->

---
layout: end
---

# Key takeaways

- **Invest in the check** before investing in autonomy
- **Separate builder and judge**; prefer objective stop conditions
- **Persist state in files**, checkpoint with git, and cap every loop

anthropic.com/engineering/building-effective-agents\
claude.com/blog/getting-started-with-loops\
ghuntley.com/ralph
