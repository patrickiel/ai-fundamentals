# Agent Loops: Goal, Action, Check, Stop

> **Core idea in one sentence:** Every agent loop has the same skeleton (a goal, an action, a check and a stop condition), and loops differ mainly in *what does the checking* and *what survives between rounds*, so pick the loop whose check you can actually trust.

## TL;DR

- An **agent** is, in Anthropic's words, "typically just LLMs using tools based on environmental feedback in a loop" [1]. The ReAct paper showed the core move: interleave reasoning with actions so each observation updates the plan [2].
- A loop is only as good as its **check**. Agents need "ground truth" from the environment at each step, such as test results or command output, and a **stopping condition** such as a maximum number of iterations [1].
- Anthropic classifies Claude Code loops by **trigger**: turn-based (you prompt), goal-based (`/goal`), time-based (`/loop`, `/schedule`) and proactive (routines that run without you) [5].
- Comparing loops by **check and state** gives five useful patterns: the single-task loop, the learning loop, the multi-critic review loop, the build-and-score loop and the process-improvement loop.
- Rule of thumb: start with the cheapest loop whose check you trust, and only add agents or rounds when that check can't tell good work from bad [1][5].

## What a loop is

In ordinary prompting, *you* are the control system: "Every prompt you send starts a manual loop with you directing each turn" [5]. Loop engineering moves that job into the system. The agent acts, observes the result and keeps going until a condition says it is finished.

The idea predates any product. ReAct (2022) had the model produce "reasoning traces and task-specific actions in an interleaved manner", so that reasoning can "induce, track, and update action plans" while actions fetch new information [2]. "Building effective agents" adds two requirements: the agent must "gain 'ground truth' from the environment at each step (such as tool call results or code execution) to assess its progress", and "it's also common to include stopping conditions (such as a maximum number of iterations) to maintain control" [1]. Most practical loops mix a fixed outer **workflow** (build, check, repeat) with an **agent** making decisions inside each step [1].

### The shared anatomy

| Part | Question it answers | Examples |
|---|---|---|
| **Goal** | What does "done" mean? | "All tests in `test/auth` pass and lint is clean" |
| **Action** | What does the agent do each round? | Edit code, revise a skill, apply review fixes |
| **Check** | How is the result judged? | Test suite, build, a separate model, a rubric |
| **Stop condition** | When does the loop end? | Check passes, turn or time cap, judged impossible |

Two more properties separate the variants: **state** (does anything, such as a progress file or journal, persist between rounds?) and **who checks** (the same agent, a separate model, several critics or a deterministic tool?).

### Two ways to classify loops

Anthropic's guide sorts loops by what starts each cycle [5]:

| Type | Trigger | Stops when | Claude Code primitive (as of 2026-10) |
|---|---|---|---|
| Turn-based | Your prompt | Claude judges it done or needs input | Ordinary session |
| Goal-based | A prompt with a condition | Goal met, or a turn cap | `/goal` |
| Time-based | An interval | You cancel, or the work completes | `/loop`, `/schedule` |
| Proactive | An event or schedule, no human present | Each task's goal is met | Routines + `/goal` + skills |

That axis tells you *when* a loop runs. The five patterns below sort loops by *how they judge their own work*, which is what decides whether a loop converges or burns tokens. The guide's own advice points the same way: "The more quantitative the checks are, the easier it is for Claude to self-verify" [5].

## 1. The single-task loop

The simplest loop does one job, checks it and stops. It carries little or nothing forward in the model's context. Two well-known forms:

**Ralph.** Geoffrey Huntley's technique is, "in its purest form, … a Bash loop": `while :; do cat PROMPT.md | claude-code ; done` [3]. Each pass starts with a fresh context window and does "only one thing". Ralph is stateless only in its *context*: the state lives in files such as specs, a `fix_plan.md` the agent updates, and an `AGENT.md` on how to build and run the project. Its check is "back pressure" from tests, type checkers and static analysis that "reject invalid code generation". Huntley positions it for greenfield work and says it still needs senior engineers to guide it [3].

**`/goal` in Claude Code.** You state a completion condition. After each turn, a separate "small fast model" judges it *not yet met*, *met* or *impossible*, so "completion is decided by a fresh model rather than the one doing the work" [4]. The evaluator "doesn't run commands or read files independently", so the condition must be something Claude's own output can show, such as a test run in the transcript. Give it "one measurable end state", "a stated check", the constraints that must hold, and a cap such as "or stop after 20 turns" [4]. As of 2026-10, `/goal` wraps a session-scoped prompt-based Stop hook [4].

### Make the check objective

A model's opinion that the work is done is the weakest check. The fix is to **write the tests before the feature**, then set the goal as "make these tests pass; don't modify the tests". The stop condition becomes evidence, and tests for *other* features act as guardrails: if the agent breaks something it shouldn't touch, a test fails. Anthropic's harness research found the same failure without such checks: Claude tended to declare victory on the whole project too early and to "mark a feature as complete without proper testing" [6].

### Checkpoint working states

Unattended runs need a way back. In Anthropic's long-running harness, each session commits its work with a descriptive message, so the agent can "use git to revert bad code changes and recover working states of the code base" [6]. A one-line instruction in your agent's project file ("commit after each passing test run") gives the single-task loop the same safety net.

**Fits:** well-specified work with an objective check (tests, a build, a schema validator, an empty queue).

## 2. The learning loop

This loop's job is to **improve something you will reuse**, such as a skill, a prompt or a workflow, and to write down what it learned so later rounds and later users benefit.

The research root is Reflexion (Shinn et al., 2023). Agents "verbally reflect on task feedback signals, then maintain their own reflective text in an episodic memory buffer" that guides later attempts, improving behaviour without changing model weights [7]. Ralph's `AGENT.md` is a practical version: the loop updates it with what it learns about building and running the project [3].

A useful design has two parts:

- **A baseline comparison.** Run the same task *with* and *without* the asset and compare. Without a baseline you can't tell whether a skill helps at all (topic 18 covers evals in depth).
- **A journal that lives with the asset**, for example a `learning.md` next to the skill, recording what was tried, the measured result and the lesson. It stops the loop repeating failed experiments and tells future runs what caused problems before.

**Fits:** assets you will run many times, where small improvements add up.

## 3. The multi-critic review loop

One reviewer asked to judge everything tends to miss things. A common pattern splits review across **specialised critics** (for example factual accuracy, relevance, safety and style), each with a narrow brief and the tools that brief needs, such as web search for the fact-checker.

The root pattern is what "Building effective agents" calls **parallelization by voting**: "running the same task multiple times to get diverse outputs", for example several prompts that each review code for vulnerabilities [1]. Andrej Karpathy's *llm-council* is a related design: several models answer, rank each other's answers anonymously, and a "chairman" model writes the final reply. He calls it a "fun Saturday hack" he won't support, so treat it as inspiration [8].

A central orchestrator that runs the rounds keeps the history of earlier rounds in one place. Critics that message each other give more independent views at more cost. Topic 23 compares the two.

**Fits:** output judged on several separate qualities, where no single test can say "correct".

## 4. The build-and-score loop

Here **one agent builds and another scores**, and the builder's job is to raise the score against a fixed rubric. This is the **evaluator-optimizer** workflow: "one LLM call generates a response while another provides evaluation and feedback in a loop". Anthropic says it is "particularly effective when we have clear evaluation criteria" and when feedback demonstrably improves the result [1].

Three design choices make the score trustworthy:

- **Separate the judge from the builder.** Anthropic's guide notes that "a reviewer with fresh context is less biased and not influenced by the main agent's reasoning" [5], the same reason `/goal` uses a separate evaluator [4].
- **Give the scorer no write access.** A reviewer that can't edit can't quietly fix things or grade its own changes. This is general practice, easy to enforce with a read-only tool list.
- **Persist scores between rounds**, for example in a JSON file, so you can see whether the loop is converging or stuck.

Breadth costs money: save reviewers that fan out to many sub-agents for large, finished codebases, and use one reviewer with a clear rubric for everyday work.

**Fits:** you can state a quality metric and want work to converge on it.

## 5. The process-improvement loop

None of the loops above improves **the loop itself**. This one adds a step after each round: review how the round went and change the *process* (prompts, rubric, step order, checks), not just the output.

Anthropic's guide states the principle: when a result misses the standard, encode the fix into the system rather than only patching that one result [5]. Huntley describes the manual version: each time Ralph goes wrong, you add a "sign" to the prompt [3]. A process-improvement loop automates part of that tuning with a separate agent that reads the round's transcript and proposes changes. Keep a human approving them, because a loop that rewrites its own rubric can drift toward whatever is easiest to score (topic 29).

**Fits:** processes you will run many times, where a better process pays off across many future runs.

## Choosing a loop

| If you need to... | Use | Because the check is... |
|---|---|---|
| Finish a well-specified task | Single-task | Objective (tests, build, validator) |
| Improve a reusable skill or prompt | Learning | A with/without comparison, logged over time |
| Review output on many dimensions | Multi-critic review | Several specialised critics |
| Push quality toward a target | Build-and-score | A separate, read-only scorer with a rubric |
| Improve how you build, not just what | Process-improvement | A score plus a reviewed process critique |

Cost scales with agents × rounds. Anthropic's guidance is to match the primitive and the model to the job, use scripts for deterministic steps, and "pilot before a large run", noting that model and effort level are "among the biggest levers on what a loop costs" [5]. "Building effective agents" puts it more generally: aim for "the simplest solution possible, and only increasing complexity when needed" [1].

## What lasts vs. what will date

- **Durable principles:**
  - A loop is only as good as its **check**. Invest in tests, rubrics and baselines before investing in autonomy.
  - Prefer **objective stop conditions** to a model's own judgement, and always have a cap.
  - **Separate the judge from the builder**, and give scorers no write access.
  - **Measure against a baseline** before claiming an asset helps.
  - **Persist state in files** (plans, progress logs, journals, git history), not in the context window.
  - **Checkpoint working states** so autonomous runs can roll back.
- **Time-sensitive details:**
  - Claude Code's `/goal`, `/loop`, `/schedule` and routines, the evaluator model and its behaviour (as of 2026-10).
  - Anthropic's four-way trigger taxonomy (June 2026 guide).
  - The Ralph technique's specific files and its 2025 framing.

## Try it yourself

1. **Tests-first goal.** Write three to five failing tests for a small function. Run `/goal all tests in <file> pass without modifying the test file, or stop after 10 turns` (or the equivalent in your tool). Note rounds and tokens.
2. **Annotate the anatomy.** For one workflow you already use, write down its goal, action, check, stop condition and state. If you can't name the check, fix that first.
3. **With/without baseline.** Run a task five times with a skill or prompt template and five times without. Start a `learning.md` with what actually differed.
4. **Split the reviewer.** Review one document with separate prompts for accuracy, safety and style, then with a single "review this" prompt. Which issues did only one critic catch?
5. **Build and score.** Write a 100-point rubric for a small app. One session builds, a separate read-only session scores. Run three rounds and check whether the score rises.

## Key terms

- **Agent loop**: a system in which an agent repeatedly acts, observes the result and decides whether to continue, without a human writing each prompt.
- **Stop condition**: the rule that ends a loop: a passed check, a cap on turns or time, or a judgement that the goal is impossible.
- **Back pressure**: Huntley's term for automated checks (tests, type checkers, linters) that reject bad output and push the loop back to work.
- **Evaluator-optimizer**: a workflow where one model generates and another evaluates and gives feedback, in a loop.
- **Baseline comparison**: running the same task with and without an asset to measure its real effect.
- **Episodic memory (Reflexion)**: reflective text an agent writes after feedback and reads on later attempts.
- **Rubric**: an explicit, weighted set of criteria used to score work consistently across rounds.

## Sources

1. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Erik Schluntz and Barry Zhang, Anthropic, 2024-12-19). Workflows vs. agents, agents as tool use in a loop, ground truth at each step, stopping conditions, parallelization by voting, the evaluator-optimizer pattern, and "start simple".
2. [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) (Yao et al., arXiv 2022-10, rev. 2023-03). The interleaved reason-act-observe loop that underlies modern agents.
3. [Ralph Wiggum as a "software engineer"](https://ghuntley.com/ralph/) (Geoffrey Huntley, 2025-07-14). The Bash-loop technique, one item per loop, fresh context with state in files, back pressure from tests, and tuning by adding signs.
4. [Keep Claude working toward a goal](https://code.claude.com/docs/en/goal) (Claude Code docs, Anthropic, accessed 2026-10-14). How `/goal` evaluates a condition with a separate small model, how to write conditions, turn caps, stall protection, and the Stop-hook mechanism.
5. [Loop engineering: Getting started with loops](https://claude.com/blog/getting-started-with-loops) (Delba de Oliveira and Michael Segner, Anthropic, 2026-06-30). The turn/goal/time/proactive taxonomy, quantitative checks, fresh-context reviewers, encoding fixes into the system, and cost levers.
6. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Justin Young, Anthropic, 2025-11-26). Progress files, feature lists with pass flags, git commits for recovery, and the "declares victory too early" failure mode.
7. [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) (Shinn et al., arXiv 2023-03, rev. 2023-10). Agents that learn across attempts by writing reflections to an episodic memory.
8. [llm-council](https://github.com/karpathy/llm-council) (Andrej Karpathy, GitHub). Several models answer, anonymously rank each other, and a chairman model synthesises; an unsupported weekend project.
