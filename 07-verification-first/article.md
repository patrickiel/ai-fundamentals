# Verification First: Giving a Coding Agent a Way to Check Its Own Work

> **Core idea in one sentence:** The most effective single improvement to an agentic coding workflow is to give the agent a check it can run (tests, a build, a browser, a linter), so it works inside a feedback loop and stops when the check passes, not when the work merely looks done.

## TL;DR

- Boris Cherny, who created Claude Code, calls verification "probably the most important thing to get great results": with a feedback loop, he estimates, the agent will "2-3x the quality of the final result" [1].
- Anthropic's guide gives the mechanism: "Claude stops when the work looks done. Without a check it can run, 'looks done' is the only signal available, and you become the verification loop" [2].
- Verification looks different in each domain: a bash command, a test suite, a browser, a phone simulator [1]. Pick the most direct evidence for the work.
- Decide how hard the check gates the stop: in the prompt, as a session goal, as a deterministic **Stop hook**, or as an independent **reviewer subagent** [2][4][5].
- Plan before executing, and grant autonomy through explicit permission rules rather than blanket bypasses, except inside an isolated sandbox [1][3].
- Simple, mostly built-in features carry most of the value. Cherny calls his own setup "surprisingly vanilla" [1].

## Why verification is the highest-leverage move

People often hand an agent a task, see small errors or an unfinished result, and decide the tool doesn't work. Human developers rarely write correct code in one pass either. They write it, run it, see what breaks and fix it. What makes them productive is the loop.

An agent without a check can only return its best guess. An agent that can run the tests, open the page or read the compiler output can see its own mistakes and fix them before handing back the work. Anthropic's best-practices guide describes the difference as "the difference between a session you watch and one you walk away from" [2]. Without a check, every mistake waits for you to notice it.

Andrej Karpathy describes the same lever from the user's side: "LLMs are exceptionally good at looping until they meet specific goals... Don't tell it what to do, give it success criteria and watch it go. Get it to write tests first and then pass them." [7]

Cherny's figure of 2-3x is his estimate from his own work, not a measured benchmark [1]. The direction is consistent across sources, though: a checkable target changes what the agent can do on its own.

## Turning a task into a checkable task

Anthropic's guide shows the change as before/after prompts [2]:

| Strategy | Before | After |
|---|---|---|
| Provide verification criteria | "implement a function that validates email addresses" | "write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing" |
| Verify UI changes visually | "make the dashboard look better" | "[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them" |
| Address root causes | "the build is failing" | "the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error" |

Three habits follow:

- **Test first where you can.** For bugs, "write a failing test that reproduces the issue, then fix it" [2]. For features, tests written before the code act as both the specification and the check (topic 08 builds a workflow around this).
- **Review the check, not just the code.** A common pattern is to ask the agent which tests it would write and review that list before implementation starts. You check intent, which is faster than reading every line.
- **Ask for evidence, not assertions.** Have the agent show "the test output, the command it ran and what it returned, or a screenshot of the result." Reviewing evidence is faster than re-running the verification yourself [2].

A standing rule in the project's instruction file (topic 06) makes this the default:

```markdown
## Verification
- Before starting a task, state how you will verify it (tests, commands, or UI checks).
- Don't report a task as done until that check has passed; show the output.
```

## Verification depends on the domain

Cherny: "Verification looks different for each domain. It might be as simple as running a bash command, or running a test suite, or testing the app in a browser or phone simulator. Make sure to invest in making this rock-solid." [1]

| Work | Evidence the agent can read |
|---|---|
| Backend, scripts, libraries | Test suite results, a command's exit code and output, a diff against a fixture |
| Build health | Compiler and type-checker output |
| Web UI | A driven browser that clicks through the flow and takes screenshots |
| Mobile | A phone simulator the agent can operate |
| Style and CI readiness | Linters and formatters |

For the UI case, Cherny says Claude tests every change he lands to claude.ai/code with the Claude Chrome extension: "It opens a browser, tests the UI, and iterates until the code works and the UX feels good." [1]

## How hard should the check gate the stop?

Once a check exists, Anthropic's guide lists four ways to enforce it, each trading setup for attention [2]:

1. **In one prompt:** ask the agent to run the check and iterate, as in the table above. Works on any task today.
2. **Across a session:** set the check as a `/goal` condition. A separate evaluator re-checks it after every turn (as of 2026-10).
3. **As a deterministic gate:** a **Stop hook** runs your check as a script and blocks the turn from ending until it passes [2][4].
4. **By a second opinion:** a verification subagent reviews the result in a fresh context, "so the agent doing the work isn't the one grading it" [2].

Hooks are the deterministic option. Claude Code's hooks guide describes them as giving "deterministic control: certain actions always happen rather than relying on the LLM to choose to run them." An agent-based hook can, for example, "verify that tests pass before allowing Claude to stop." Stop hooks need a guard against endless loops. Claude Code overrides a Stop hook after it blocks eight times in a row with no tool call in between, and scripts should check the `stop_hook_active` field (as of 2026-10) [4].

The independent reviewer matters more the longer the agent runs unattended. A reviewer subagent "sees only the diff and the criteria you give it, not the reasoning that produced the change" [2]. Subagents run in their own context window with their own prompt and tool access [5]. Anthropic adds a caution: a reviewer asked to find gaps will usually report some, and chasing all of them leads to over-engineering. Tell it to flag only gaps that affect correctness or the stated requirements [2].

## Long-running tasks

For very long tasks, Cherny lists three options: "(a) prompt Claude to verify its work with a background agent when it's done, (b) use an agent Stop hook to do that more deterministically, or (c) use the ralph-wiggum plugin" [1].

The third is a loop pattern. Geoffrey Huntley, who named it, describes it as "a Bash loop" that feeds the same prompt to the agent again and again, with tests as backpressure [6]. Anthropic's plugin implements it with a Stop hook that blocks exit and feeds the prompt back. The README recommends it for "well-defined tasks with clear success criteria" and "tasks with automatic verification (tests, linters)", and not for tasks needing human judgement or with unclear success criteria. It says to "always use `--max-iterations` as a safety net" [8]. Huntley himself frames the technique as suited to greenfield work under expert supervision [6].

All three patterns remove the human from the inner loop. None of them removes the check.

## Plan first, then execute with bounded permissions

Cherny: "Most sessions start in Plan mode... I will use Plan mode, and go back and forth with Claude until I like its plan. From there, I switch into auto-accept edits mode and Claude can usually 1-shot it. A good plan is really important!" [1]

In plan mode Claude "reads files, runs shell commands to explore, and writes a plan, but does not edit your source" [3]. Anthropic's guide recommends explore → plan → implement → commit for uncertain or multi-file work, and skipping the plan when "you could describe the diff in one sentence" [2]. Planning moves the cost of misunderstanding to the start, where it's cheapest.

On permissions, Cherny writes: "I don't use --dangerously-skip-permissions. Instead, I use /permissions to pre-allow common bash commands that I know are safe in my environment." The allowlist is checked into `.claude/settings.json` and shared with the team [1]. For long unattended runs he makes one exception: he uses a don't-ask mode or skips permissions **in a sandbox** so the agent isn't blocked waiting for him [1]. Claude Code's docs match this. `bypassPermissions` is for "isolated containers and VMs only", and deny rules block in every mode [3]. As of 2026-10, Claude Code also offers an auto mode in which a classifier model reviews actions instead of you [3].

## Formatting and other cheap checks

Verification covers form as well as function. Cherny's team uses a `PostToolUse` hook to format Claude's code: "Claude usually generates well-formatted code out of the box, and the hook handles the last 10% to avoid formatting errors in CI later." [1] The hooks guide has a ready-made example that runs Prettier on every file Claude edits [4]. Catching it locally is cheaper than a failed CI run.

## Keep the setup simple

Cherny describes his setup as "surprisingly vanilla! Claude Code works great out of the box, so I personally don't customize it much." [1] The pieces he does use are small:

- **A shared instruction file grown from mistakes.** "Anytime we see Claude do something incorrectly we add it to the CLAUDE.md." Lessons are also added during code review through the Claude Code GitHub Action [1] (topic 06).
- **Slash commands** for "inner loop" workflows repeated many times a day, checked into `.claude/commands/` [1].
- **A few subagents** that automate common steps for most PRs, for example a `code-simplifier` and a `verify-app` agent "with detailed instructions for testing Claude Code end to end" [1]. Several of them exist to check work.
- **Parallel sessions:** he runs five in his terminal with numbered tabs and system notifications, plus more on the web [1] (topic 27).
- **Model choice:** at the time, the largest model with thinking (Opus 4.5), because "since you have to steer it less and it's better at tool use, it is almost always faster than using a smaller model in the end" [1]. The model will date. The principle of measuring time to a *correct* result won't.

## What lasts vs. what will date

- **Durable principles:**
  - An agent with a feedback loop beats one without. Make "how will this be verified?" part of every task.
  - Choose evidence that fits the domain: tests, build output, browser checks, simulators, linters.
  - Ask for evidence, not claims of success.
  - Escalate enforcement as autonomy grows: prompt, then deterministic gate, then independent reviewer.
  - Resolve ambiguity in a planning step, before execution.
  - Grant autonomy through explicit permissions, and bypass checks only inside isolation.
- **Time-sensitive details (as of 2026-10):** Claude Code's mode names, `/goal`, auto mode, hook event names and loop limits, the Chrome extension, the ralph-wiggum plugin, Opus 4.5, and the specific numbers in one person's January 2026 setup (2-3x, five sessions).

## Try it yourself

1. **Rewrite three prompts** from your recent history using the before/after table: add test cases, a screenshot comparison, or "fix the root cause and verify the build."
2. **Review the test list first.** For one feature, ask only for the tests the agent would write. Edit the list, then allow implementation.
3. **Add a Stop hook** that runs your linter and fast tests, with a `stop_hook_active` guard. Count how often it blocks a premature "done."
4. **Add a reviewer.** After a task, have a subagent review the diff against your plan and report only correctness gaps.
5. **Write a permission allowlist** for your safe commands, add deny rules for destructive ones, and commit it.

## Key terms

- **Verification loop:** the agent runs a check, reads the result and iterates until it passes.
- **Check:** anything that returns a signal the agent can read: test results, exit codes, linter output, screenshots.
- **Plan mode:** a Claude Code mode that explores and writes a plan without editing source files.
- **Stop hook:** a script that runs when the agent tries to end its turn and can block it until a condition holds.
- **PostToolUse hook:** a script that runs after a tool call, for example formatting a file after each edit.
- **Reviewer subagent:** a separate agent with a fresh context that checks the diff against stated criteria.
- **Ralph loop:** re-running an agent on the same prompt until checks pass, with an iteration cap.
- **Permission rules:** allow, ask and deny lists for agent actions, shared through committed settings.

## Sources

1. [How I use Claude Code (thread)](https://x.com/bcherny/status/2007179832300581177) (Boris Cherny, X, 2026-01-02). The verification tip and 2-3x estimate, domain-specific verification, plan mode then auto-accept, permissions and sandboxed long runs, the PostToolUse formatter, subagents, slash commands, the shared CLAUDE.md and parallel sessions.
2. [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (Anthropic, accessed 2026-10-14; the earlier engineering post "Claude Code: Best practices for agentic coding" now redirects here). "Looks done" as the failure mode, before/after verification prompts, the four gating levels, evidence over assertion, explore-plan-implement, and the adversarial review step.
3. [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) (Anthropic, accessed 2026-10-14). What plan mode does, acceptEdits and auto mode, bypassPermissions for isolated environments only, and deny rules applying in every mode.
4. [Automate actions with hooks](https://code.claude.com/docs/en/hooks-guide) (Anthropic, accessed 2026-10-14). Hooks as deterministic control, the Prettier PostToolUse example, agent-based Stop hooks that verify tests pass, and the loop limit with `stop_hook_active`.
5. [Create custom subagents](https://code.claude.com/docs/en/sub-agents) (Anthropic, accessed 2026-10-14). Subagents' separate context, prompts and tool access, the basis for independent reviewers.
6. [Ralph Wiggum as a "software engineer"](https://ghuntley.com/ralph/) (Geoffrey Huntley, 2025-07-14). The original loop technique, tests as backpressure, and its limits (greenfield work, expert operator).
7. [A few random notes from claude coding quite a bit last few weeks](https://x.com/karpathy/status/2015883857489522876) (Andrej Karpathy, X, 2026-01-26). Success criteria and test-first as the main lever for agent loops.
8. [ralph-wiggum plugin README](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) (Anthropic, accessed 2026-10-14). The Stop-hook implementation of the loop, when to use and not use it, and the `--max-iterations` safety net.
