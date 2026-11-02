---
theme: ../theme
chapter: '08'
header: 'AI Fundamentals · 08 · Spec-driven development'
layout: cover
---

# Spec-Driven Development

Agree on **what and why** before the agent decides **how**, and review each artifact before moving on.

---

## Why prompting alone fails

- Vibe coding: fine for prototypes, unreliable for serious or existing codebases
- "The issue isn't the coding agent's coding ability, but our approach." (GitHub)
- Models "are exceptional at pattern completion, but not at mind reading"
- Unstated requirements → guesses → compounding drift
- SDD: a written spec anchors plan, tasks, tests and code

<!-- From GitHub's Spec Kit announcement (Sept 2025). Karpathy's "wrong assumptions… run along with them" (topic 06) is the same failure seen from daily use. -->

---

## The core workflow

| Phase | Question | Output |
|---|---|---|
| **Specify** | What and why? | `spec.md`: stories, acceptance criteria, edge cases |
| **Plan** | How? | `plan.md` + research, data model, contracts |
| **Tasks** | Which small steps? | `tasks.md`, ordered, `[P]` = parallel-safe |
| **Implement** | Build it | Code and tests |

**Checkpoint each phase:** "you don't move to the next one until the current task is fully validated"

<!-- GitHub calls this separating the stable "what" from the flexible "how": change the stack without rewriting requirements. -->

---

## Spec Kit commands (as of 2026-10)

- `constitution`: project principles, once per project
- `specify` → `clarify` (up to 5 questions) → `plan` → `checklist`
- `tasks` → `analyze` (read-only consistency check)
- `implement` (phase by phase) → `converge` (append missed work as tasks)
- Repeat **implement → converge** until "Converged"

> Run each step "one at a time, and review the result before continuing."

<!-- Invoked in the agent's chat as /speckit.plan, /speckit-plan etc. depending on the agent. Installed with uv tool install specify-cli, then specify init. v1.2.0 released 2026-10-13. -->

---

## Templates constrain the model

- Spec: "Focus on **WHAT** users need and **WHY**"; "Avoid **HOW**"
- "Mark all ambiguities… **Don't guess**"

```text
[NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]
```

- Checklist: "No [NEEDS CLARIFICATION] markers remain", "Requirements are testable"

<!-- The most transferable idea: ambiguity is marked and resolved by a human, not silently filled in. /speckit.clarify walks through it. -->

---

## More guardrails in the templates

- **Complexity gates:** "No future-proofing?", "Using framework directly?"; failures must be justified in writing
- **Tests before code:** contracts → contract, integration, e2e, unit tests → source to make them pass
- **No speculative features:** every feature traces to a user story

<!-- These target the same over-engineering habit as topic 06's "simplicity first", and make "done" checkable as in topic 07. -->

---

## Scripts for bookkeeping

- `specify`: next feature number, semantic **branch**, `specs/<branch>/` from template
- Repo scripts: `create-new-feature`, `setup-plan`, `check-prerequisites`
- One right answer → a **script** does it every time; model does the judgement

```
specs/003-chat-system/
  spec.md  plan.md  research.md  data-model.md
  contracts/  quickstart.md  tasks.md
```

<!-- Layout from Spec Kit's methodology example; exact files vary by version. Specs become versioned artifacts reviewed and merged like code. -->

---

## A second implementation: Kiro (AWS)

| Kiro file | Role |
|---|---|
| `requirements.md` | User stories + acceptance criteria |
| `design.md` | Architecture, sequence diagrams |
| `tasks.md` | Discrete, trackable tasks |

- Run all tasks in dependency-ordered **waves**; separate **bugfix specs**
- Same shape as Spec Kit: **what → how → tasks**, reviewed in turn

<!-- Two independent tools converging on the same structure suggests the pattern matters more than the tool. As of 2026-10. -->

---

## How much spec? Three levels

| Level | Meaning |
|---|---|
| **Spec-first** | Write a spec, use it for this task |
| **Spec-anchored** | Keep the spec to guide later maintenance |
| **Spec-as-source** | Humans edit only the spec, never the code |

Most tools today are mainly **spec-first** (Böckeler, Thoughtworks)

<!-- Spec Kit's methodology argues for the strong version: "code serves specifications". Böckeler compares spec-as-source to model-driven development, which "never took off for business applications". -->

---

## Costs and open questions

- Artifacts "repetitive… with each other, and with the code"; "I'd rather review code than all these markdown files"
- Agent "not follow all the instructions", or "way overboard" on the constitution
- Small bug: "like using a sledgehammer to crack a nut"
- Lighter option: agent **interviews you** → self-contained `SPEC.md` → fresh session

<!-- Böckeler: spec-first "is definitely valuable in many situations", but the term "isn't very well defined yet". GitHub calls Spec Kit "our experiment" and lists walls of text as an open problem. The SPEC.md interview pattern is from Anthropic's Claude Code best practices. -->

---

## What lasts vs. what will date

**Lasts**
- What/why before how, in separate documents
- Mark ambiguity; a human resolves it
- Review each step; tests before code
- Scripts for bookkeeping, model for judgement
- Size the process to the change

**Will date (as of 2026-10)**
- Spec Kit command names, converge/clarify/analyze, extensions, agent list
- Kiro spec types and task waves

<!-- The tools are moving fast; the discipline underneath them isn't new. -->

---

## Try it yourself

1. Write a **technology-free spec**; mark open questions `[NEEDS CLARIFICATION]`
2. Ask an agent for **≤5 ambiguity questions**; resolve before planning
3. Write the **plan separately**; could you swap the stack without editing the spec?
4. Generate **tasks**, tests first, `[P]` on independent ones; check they really are
5. **Converge by hand:** a fresh session lists missed requirements
6. Replace one prompted bookkeeping step with a **script**

<!-- Exercise 3 is the quickest test of whether your spec really stays at the "what" level. -->

---
layout: end
---

# Key takeaways

- **Specify → plan → tasks → implement**, with a review at each step
- **Mark ambiguity** instead of letting the agent guess; **tests before code**
- Powerful for features in real systems, **overkill for small fixes**

GitHub's announcement: https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/\
Spec Kit: https://github.com/github/spec-kit\
Böckeler on SDD tools: https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
