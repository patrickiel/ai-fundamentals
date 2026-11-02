---
theme: ../theme
chapter: '07'
header: 'AI Fundamentals · 07 · Verification first'
layout: cover
---

# Verification First

Give the agent **a check it can run**, so it stops when the check passes, not when the work **looks done**.

---

## The problem

> "Claude stops when the work looks done. Without a check it can run, 'looks done' is the only signal available, and you become the verification loop." (Anthropic)

- Humans don't get it right first time either. They **run, observe, fix**
- With a check, the agent finds and fixes its own mistakes
- Boris Cherny (created Claude Code): "probably the most important thing"; his estimate: **2-3x** quality

<!-- The 2-3x figure is his estimate from his own work, not a benchmark. Karpathy makes the same point: give it success criteria and watch it go. -->

---

## Make the task checkable

| Before | After |
|---|---|
| "implement email validation" | "write validateEmail… test cases: … run the tests after implementing" |
| "make the dashboard look better" | "[screenshot] implement this… screenshot the result, compare, fix differences" |
| "the build is failing" | "[error] fix it, verify the build succeeds, address the root cause" |

- Bugs: **write a failing test first**, then fix
- Ask for **evidence**: test output, command + result, screenshot

<!-- From Anthropic's best-practices guide. Reviewing evidence is faster than re-running checks yourself. -->

---

## Verification depends on the domain

| Work | Evidence |
|---|---|
| Backend / scripts | Test suite, exit codes, diff against a fixture |
| Build health | Compiler and type-checker output |
| Web UI | Driven browser + screenshots |
| Mobile | Phone simulator |
| Style / CI | Linters and formatters |

> "Make sure to invest in making this rock-solid." (Cherny)

<!-- Cherny: Claude tests every change he lands to claude.ai/code in a browser via the Chrome extension and iterates until the UX feels good. -->

---

## A standing rule

```markdown
## Verification
- Before starting a task, state how you will verify it.
- Don't report done until that check has passed; show the output.
```

- Put it in the instruction file (topic 06)
- Review the **test list** before implementation: you check intent, not every line

<!-- Reviewing the test list first is a common pattern rather than a quoted source recommendation. -->

---

## How hard should the check gate the stop?

1. **In the prompt:** run the check and iterate
2. **Session goal:** `/goal` condition re-checked after every turn
3. **Deterministic gate:** a **Stop hook** blocks the turn until the check passes
4. **Second opinion:** a reviewer **subagent** in a fresh context

Each step trades **setup** for **attention**.

<!-- From Anthropic's guide, as of 2026-10. The reviewer "sees only the diff and the criteria", so the agent doing the work isn't grading itself. Tell it to flag only correctness gaps, or it will push you into over-engineering. -->

---

## Hooks: deterministic control

- Hooks run "rather than relying on the LLM to choose to run them"
- **Agent-based Stop hook:** verify all unit tests pass before allowing a stop
- **PostToolUse hook:** format every edited file (e.g. Prettier)
- Guard against loops: check `stop_hook_active`; Claude Code overrides after **8** consecutive blocks

<!-- Cherny's team formats with a PostToolUse hook: the model writes well-formatted code, the hook handles "the last 10%" before CI. -->

---

## Long-running tasks

| Option (Cherny) | How |
|---|---|
| Background verifier | Prompt Claude to verify with a background agent when done |
| Stop hook | Run the verification deterministically on stop |
| Ralph loop | Re-feed the same prompt until checks pass |

- Ralph: "a Bash loop" (Huntley); plugin uses a Stop hook
- Use for **clear success criteria + automatic verification**; always set `--max-iterations`

<!-- The plugin README says not to use it for tasks needing human judgement or with unclear success criteria. Huntley frames it as greenfield work under expert supervision. -->

---

## Plan first, then execute

- "Most sessions start in Plan mode… go back and forth with Claude until I like its plan"
- Then **auto-accept edits**: "Claude can usually 1-shot it"
- Plan mode reads and explores but **doesn't edit source**
- Skip the plan if "you could describe the diff in one sentence"

> Misunderstandings are cheapest to fix before any code exists.

<!-- Anthropic's recommended flow: explore, plan, implement, commit. -->

---

## Bounded autonomy

- "I don't use --dangerously-skip-permissions": pre-allow **safe** commands instead
- Allowlist checked into `.claude/settings.json`, shared with the team
- Long unattended runs: skip prompts **only in a sandbox**
- Docs: `bypassPermissions` is for "isolated containers and VMs only"; **deny** rules block in every mode

<!-- As of 2026-10 Claude Code also has auto mode, where a classifier model reviews actions instead of you. -->

---

## Keep the setup simple

- "Surprisingly vanilla": Claude Code works out of the box
- Shared CLAUDE.md grown from mistakes, also during code review
- Slash commands for repeated "inner loop" workflows
- A few subagents: `code-simplifier`, `verify-app`, several of them checkers
- Five parallel terminal sessions, numbered tabs
- Model chosen for **time to a correct result**, not speed per reply

<!-- He used Opus 4.5 with thinking at the time. The model will date; the principle won't. -->

---

## What lasts vs. what will date

| Durable | Will date (as of 2026-10) |
|---|---|
| Feedback loops beat guessing | `/goal`, auto mode, hook loop limits |
| Domain-appropriate evidence | Chrome extension, ralph-wiggum plugin |
| Evidence over assertions | Opus 4.5 |
| Escalate enforcement with autonomy | Mode names and flags |
| Plan before execute; bypass only in isolation | 2-3x, five sessions (one person, Jan 2026) |

<!-- Everything on the right reflects specific products or one person's setup. -->

---

## Try it yourself

1. Rewrite three recent prompts with **test cases, screenshots or root-cause checks**
2. Review the **test list** before any code
3. Add a **Stop hook** running lint + fast tests (with a loop guard)
4. Add a **reviewer subagent** reporting only correctness gaps
5. Commit an **allow / deny** permission config

<!-- Exercise 3 is where most people first see how often "done" was premature. -->

---
layout: end
---

# Key takeaways

- **Give the agent a check it can run**, and ask for evidence
- **Escalate enforcement** as autonomy grows: prompt → hook → reviewer
- **Plan first**; bound autonomy with permissions, bypass only in a sandbox

Cherny's thread: https://x.com/bcherny/status/2007179832300581177\
Anthropic best practices: https://code.claude.com/docs/en/best-practices\
Hooks guide: https://code.claude.com/docs/en/hooks-guide
