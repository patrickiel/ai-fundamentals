---
theme: ../theme
chapter: '06'
header: 'AI Fundamentals · 06 · Coding guidelines'
layout: cover
---

# Instruction Files That Change Behaviour

Write only what **corrects the agent's defaults**, keep it **short and scoped**, and **enforce** hard rules outside the file.

---

## Why the file matters, and its limits

- `CLAUDE.md` / `AGENTS.md` loads **every session**, so every line costs context
- Anthropic's test per line: *"Would removing this cause Claude to make mistakes?"*
- "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"
- But it's **context, not enforced configuration**: no guarantee of strict compliance
- Karpathy: failure modes persist "despite a few simple attempts to fix it via instructions in CLAUDE.md"

<!-- A good file reduces how often you correct the agent. It doesn't replace reviewing the diff. Sources: Anthropic best practices and memory docs; Karpathy's January 2026 post. -->

---

## What agents get wrong by default (Karpathy, Jan 2026)

- "Make wrong assumptions on your behalf and just run along with them"
- "Don't manage their confusion… don't present tradeoffs… don't push back"
- "Overcomplicate code and APIs… bloat abstractions… don't clean up dead code"
- Change code "orthogonal to the task at hand" as side effects
- The lever: "give it **success criteria** and watch it go"

<!-- His example: a 1,000-line construction cut to 100 lines once he asked whether a simpler approach would work. -->

---

## Four rules: the community CLAUDE.md

| Rule | Instruction |
|---|---|
| **Think before coding** | "Don't assume. Don't hide confusion. Surface tradeoffs." List readings, ask |
| **Simplicity first** | No speculative features or single-use abstractions; "200 lines… could be 50, rewrite it" |
| **Surgical changes** | "Every changed line should trace directly to the user's request" |
| **Goal-driven execution** | "Define success criteria. Loop until verified." |

<!-- From the andrej-karpathy-skills repository, derived from Karpathy's post, not written by him. It states its own trade-off: it biases toward caution over speed, so use judgement on trivial tasks. -->

---

## Rules 3 and 4 in practice

**Surgical changes**
- Don't "improve" adjacent code, comments or formatting; match existing style
- Mention unrelated dead code, **don't delete it**; remove only what *your* change orphaned

**Goal-driven execution**
- "Add validation" → write tests for invalid inputs, then make them pass
- "Fix the bug" → write a test that reproduces it, then make it pass
- UI: give the agent a **browser** it can drive and screenshot

<!-- Anthropic: "Claude stops when the work looks done." A check gives it an objective stopping point. Topic 07 goes deeper. -->

---

## What to include, what to leave out

| Include | Exclude |
|---|---|
| Commands Claude can't guess (`pnpm`, not `npm`) | Anything it can infer from the code |
| Style rules that differ from defaults | Standard conventions |
| Test runners, repo etiquette | Detailed API docs (link instead) |
| Project-specific architecture, env quirks, gotchas | Tutorials, file-by-file tours, "write clean code" |

Write instructions **concrete enough to verify**: "Run `npm test` before committing", not "test your changes".

<!-- From Anthropic's best-practices table. /init gives a starting point; then add what Claude wouldn't discover on its own. -->

---

## Safety rules: enforce, don't just ask

- Instruction-file text is **advisory**
- Hooks are **deterministic**: a PreToolUse hook can block an action whatever the model decides
- Permission **deny** rules block in every mode; **ask** rules force a prompt
- Split:
  - **Settings:** deny/ask for force-push, hard reset, recursive delete, deploys
  - **Instruction file:** the intent, plus "if unsure whether it's reversible, ask"

<!-- This corrects a common habit of writing "never run X" in CLAUDE.md and assuming it will hold. -->

---

## Grow the file from real mistakes

- Claude Code team: "Anytime we see Claude do something incorrectly we add it to the CLAUDE.md"
- Checked into git; the team adds to it several times a week
- Lessons added during **code review** by tagging Claude on the PR
- Add a line when: same mistake twice, review catches it, you repeat a correction
- **Prune** like code: test whether behaviour actually shifts

<!-- Source: Boris Cherny's January 2026 thread and Anthropic's memory docs. As of 2026-10 Claude Code also keeps an auto memory of corrections. -->

---

## Scope: load rules only where they apply

- Target **under 200 lines** per CLAUDE.md (Anthropic)
- `.claude/rules/*.md` with `paths:` globs load only for matching files
- Monorepo: nested files per package; AGENTS.md "closest one takes precedence"
- `@imports` organise a file but **don't reduce** its context cost
- Contradictions → the agent "may pick one arbitrarily"

```markdown
---
paths: ["src/api/**/*.ts"]
---
- All endpoints validate input and use the standard error format.
```

<!-- Emphasis: mark one repeatedly-skipped line IMPORTANT. Emphasise everything and nothing stands out. -->

---

## A skeleton

```markdown
# Acme Notes
Shared-notes app. Next.js (apps/web), Node API (apps/api), Postgres.
## Commands
- pnpm (not npm). Tests: `pnpm test`. Types: `pnpm typecheck`. Use `gh`.
## How to work
- State assumptions; if ambiguous, list readings and ask
- Minimum code; change only what the task needs
- Turn each task into a check and iterate until it passes
## Gotchas
- Migrations are generated, never hand-edited
```

<!-- Illustrative example. Destructive-command limits live in .claude/settings.json, not in this file. -->

---

## What lasts vs. what will date

**Lasts**
- Correct defaults; don't repeat what the agent can infer
- Clarify, keep changes small, make "done" checkable
- Prose is advisory: **enforce** must-never rules deterministically
- Scope rules; grow from mistakes; prune like code

**Will date (as of 2026-10)**
- File names and loading rules, the 200-line target, auto memory, AGENTS.md fallback

<!-- The same advice works for onboarding a new colleague: what's unusual here, what never to do, how we check work. -->

---

## Try it yourself

1. **Prune:** ask "would removing this cause mistakes?" for every line
2. **A/B the four rules** on the same ambiguous request; compare questions and diff size
3. **Move one "never" rule** into a deny/ask permission rule and test it
4. **Scope one area** into a path-scoped rule file
5. **Log corrections** for a week, then add and prune

<!-- Exercise 3 shows the difference between advisory and enforced rules directly. -->

---
layout: end
---

# Key takeaways

- **Think, simplify, change only what's needed, define done as checkable**
- Keep the file **short and scoped**; list only what the agent can't infer
- Instructions are advisory: **enforce** safety with permissions and hooks

Karpathy's notes: https://x.com/karpathy/status/2015883857489522876\
Anthropic best practices: https://code.claude.com/docs/en/best-practices\
CLAUDE.md docs: https://code.claude.com/docs/en/memory
