# From Loops to Graphs: Why Every Node Needs Verification

> **Core idea in one sentence:** Splitting a task across several agents arranged as a graph buys speed and lets you pick a model per step, but an unchecked error in one node flows into everything downstream, so each node needs its own check, run by a reviewer that did not do the work.

## TL;DR

- A single agent works in a **loop**: gather context, act, verify, repeat [1]. Each step waits for the one before it, even when the two are unrelated.
- A **graph** splits the task into steps (nodes) connected by rules for passing results (edges). Independent nodes can run at the same time. Anthropic's catalogue of workflow shapes (chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) [2] and frameworks such as LangGraph [6] describe the same structures.
- Graphs are faster and let you put **cheaper models on easy steps** [2][3], but they **use more tokens in total** and make failures harder to trace [3].
- The fix is **verification inside the graph**: a gate after each node, checks against the original requirements, and reviews from several angles that are merged before anything is fixed [1][2][4].
- **The builder is a poor judge of its own work.** Use a fresh context for review [4][7], and tell the reviewer to report only findings that matter, because a reviewer asked to find problems will find some [4].
- Verification checks can run **standalone**, **embedded** in a workflow, **chained** after other skills, or **on every PR** [1].

## Loops, and why they turn into graphs

The Claude Code team describes an agent as a loop: gather context, take action, verify results [1]. Without the verify step the agent stops when the work "looks done", and you become the check [4]. A loop is also sequential: step two waits for step one even when it doesn't depend on it. A graph removes that wait by giving each part of the task its own agent and connecting the parts so each result reaches the agent that needs it. Anthropic's "Building effective agents" names the common shapes [2]:

| Shape | What it does | When it fits |
|---|---|---|
| **Prompt chaining** | Steps in sequence, with a programmatic "gate" checking intermediate output | The task splits cleanly into fixed subtasks |
| **Routing** | Classify the input, send it to a specialised follow-up | Distinct categories are better handled separately, e.g. easy questions to a small model |
| **Parallelization** | Independent subtasks at once (*sectioning*), or the same task several times (*voting*) | You want speed, or several perspectives for more confidence |
| **Orchestrator-workers** | A central model splits the task, delegates, and synthesises | Subtasks cannot be predicted in advance, e.g. multi-file code changes |
| **Evaluator-optimizer** | One model generates, another evaluates and gives feedback, in a loop | Criteria are clear and iteration measurably helps |

The same paper warns that these systems "often trade latency and cost for better task performance", that autonomy brings "the potential for compounding errors", and that you should start with the simplest design and add structure only when it demonstrably helps [2].

## What a graph is made of

LangGraph, one widely used graph framework, makes the structure explicit [6]:

| Part | What it is | In LangGraph |
|---|---|---|
| **Node** | One unit of work: a model call, a tool call, or a whole agent | A function that reads shared state and returns an update |
| **Edge** | The rule for what runs next | `add_edge` for a fixed transition, `add_conditional_edges` for a decision such as "pass or retry" |
| **State** | What flows between nodes | A typed object, with reducers that merge updates from parallel branches |

In LangGraph's own examples, a parallel workflow fans out from the start to three writer nodes, all of which feed one aggregator node. A gate function sends a draft either to the end or back for improvement [6].

### Two shapes worth knowing

**Diamond (fan out, then synthesise).** One task splits into parallel workers, and one agent merges their results into a single answer. This is orchestrator-workers [2].

```
        task
      /  |  \
    A    B    C      (run in parallel)
      \  |  /
     synthesiser
```

**Fan-in at a barrier.** The same problem goes to several agents, each with a different lens. A **barrier** holds everything until every branch has reported, and only then do fixes start. This is parallelization by voting or sectioning [2]. Claude Code's dynamic workflows expose the barrier directly: `parallel()` "runs a set of agent tasks at the same time and waits for all of them" [3]. The barrier stops one reviewer's fix from landing before the other angles have been heard.

A Claude Code **dynamic workflow** (as of 2026-10) is a graph in practice: a script Claude writes that orchestrates many subagents and can have "independent agents adversarially review each other's findings before they're reported" [3].

## The costs of going wide

**Tokens.** A smaller model on easy stages lowers the cost per agent [2][3], not the total. Claude Code's documentation says a workflow "can use meaningfully more tokens than working through the same task in conversation", flags runs over 25 agents or 1.5 million projected tokens, and recommends trying a small slice first (as of 2026-10) [3].

**Traceability.** When a graph returns a wrong answer, you see only the final output. A small error in an early node has been built on by every node after it [2]. Checking once at the end tells you *that* something failed, not *where*. Verification has to sit inside the graph.

## Verification: what is built in, and what you add

The Claude Code team's post on verification loops starts from deterministic signals the agent can read: type checkers, linters, tests, runtime errors. Writing the exact build and test commands in `CLAUDE.md` saves the agent from rediscovering them [1][4]. Built-in options listed as of 2026-10 include the `/verify` skill (build, run and observe the changed app), Code Review (a managed multi-agent PR review, research preview), GitHub Actions jobs that run a verification skill on each PR, spec validation, and rubric grading by a separate grader in Claude Managed Agents (beta) [1].

Tests catch breakage, not intent. Everything Claude can't infer becomes a manual check you keep repeating, and those repeated corrections are exactly what to turn into verification [1]. The post's example is a project rule: "Reject any migration that drops a column without a backfill step." You can generate such a skill with the skill-creator plugin or write a `SKILL.md` by hand [1].

In a graph, the most valuable check is the **requirements review**: compare a node's output with what was originally asked. Each node sees only its slice, so it cannot notice that it has drifted from the whole. Claude Code's best-practices guide gives a template: have a reviewer check the diff against `PLAN.md`, confirm every requirement is implemented and tested, and report "gaps, not style preferences" [4].

## Where checks run

The verification-loops post sorts checks by where they run [1]:

| Mode | How it runs | Good for |
|---|---|---|
| **Standalone** | You invoke it deliberately | Cross-cutting checks such as a pre-commit security scan or an accessibility audit. If you find yourself running it after every change, embed or chain it instead |
| **Embedded** | Inside the skill that produces the work, e.g. a lint step at the end of a scaffolding skill | Checks tied to one kind of output. Only works for skills you can edit |
| **Chained** | One skill calls another when it finishes; a wrapper skill can add checks to skills you can't modify | Several review angles in sequence |
| **On every PR** | The stable chain runs in CI | Team-wide gates, once the chain has stopped changing |

The Claude Code team chains `/code-review` (bugs), `/simplify` (cleanup), `/verify` (end-to-end behaviour) and a custom `/design` skill that checks UI changes against a `DESIGN.md` file [1]. That is one check per angle, each measured on its own terms. The post warns that chained loops "can increase token spend", so test a chain before deploying it widely [1].

To run several angles in parallel, put them under a fan-in: one agent per review angle, a barrier, then one merge step that ranks and de-duplicates findings for the fixing agents. Dynamic workflows document this pattern ("review every file changed in this PR... then merge the per-file findings into one ranked summary") [3].

**Visual checks, cheaply.** UI verification usually means driving a browser and taking screenshots [4], which is slow when repeated inside a graph. Chrome's **chrome-headless-shell** is a lighter headless binary suited to automated screenshots, while full Chrome in headless mode is the more faithful choice for high-accuracy end-to-end testing [8]. Use the light tool for frequent checks and the faithful one for the final pass.

## Who judges: independence and quality

**The builder should not grade itself.** Anthropic's harness experiments found that "agents reliably skew positive when grading their own work", and that tuning a separate, skeptical evaluator is "far more tractable" than making the generator self-critical [7]. Claude Code's guide says the same about sessions: "A fresh context improves code review since Claude won't be biased toward code it just wrote." It recommends a Writer/Reviewer pair of sessions, or a reviewer subagent that "sees only the diff and the criteria you give it" [4].

A simple way to get that independence is a **fresh non-interactive session**: `claude -p "<prompt>"` runs once with no prior conversation, and you can pipe in only the diff and the requirements. By default `-p` still loads the project's `CLAUDE.md`, hooks and skills; `--bare` skips them for a reproducible run (as of 2026-10) [5].

**A reviewer can be wrong in both directions.** Claude Code's guide warns: "A reviewer prompted to find gaps will usually report some, even when the work is sound", and chasing every finding leads to over-engineering [4]. Anthropic's harness author found the opposite failure too: out of the box, the QA agent "tended to test superficially" and talked itself into approving flawed work [7]. In a graph both errors multiply. False alarms send many agents to "fix" working code, and missed bugs pass through every node.

So judge a reviewer by whether its findings are **correct**, not by how many it produces. Scope it to correctness and stated requirements [4], calibrate it on real outputs (topic 18), and give the judging step a model strong enough for the task. A weak verdict is copied into every node that relies on it.

## What lasts vs. what will date

- **Durable principles:**
  - Running independent work in parallel trades total cost and traceability for speed [2][3].
  - Put a gate after each step that others build on. An unchecked error compounds downstream [2].
  - Check work against the *original requirements*, especially when each worker sees only a slice [4].
  - Separate the reviewer from the builder, and scope the reviewer to findings that matter [4][7].
  - Use one focused check per angle, joined at a barrier and merged once [1][3].
  - Start simple. Add nodes and checks only when they measurably help [2].
- **Time-sensitive details (as of 2026-10):** the names and behaviour of `/verify`, `/simplify`, `/code-review`, Code Review (research preview), Managed Agents rubrics (beta), dynamic workflows and their limits and warnings, the `-p` and `--bare` flags, LangGraph's API names, and chrome-headless-shell packaging.

## Try it yourself

1. **Draw your workflow.** Take a multi-step task you hand to an agent and sketch it as nodes and edges. Mark which steps actually depend on each other. Is it a chain, a diamond, or a fan-in at a barrier?
2. **Add a gate.** Pick the node whose errors hurt most downstream. Write a programmatic check for its output (a schema, a test, a grep) and route failures back for a retry.
3. **Requirements review in a fresh session.** After an agent finishes a change, run `git diff main | claude -p "Compare this diff with the requirements below. Report only gaps that affect correctness or the stated requirements. <requirements>"`. Compare the result with the original agent's self-assessment.
4. **Grade the grader.** Run the same review prompt with a small and a large model on one diff. Mark each finding correct or not. Note which model you would trust as the judge, and why.
5. **Split one review into three.** Write three narrow review prompts (correctness, simplicity, end-to-end behaviour), run them in parallel, and write a fourth that merges and ranks their findings.

## Key terms

- **Loop:** an agent cycling through gather context, act, verify until the goal is met.
- **Graph:** a workflow whose steps are nodes connected by edges, some running in parallel.
- **Node / edge / state:** a unit of work; the rule for what runs next; the data passed between nodes.
- **Gate:** a check on intermediate output that decides whether the workflow continues or retries.
- **Diamond (orchestrator-workers):** fan out to parallel workers, then synthesise in one agent.
- **Barrier:** a point where the workflow waits for every parallel branch before going on.
- **Standalone / embedded / chained check:** run by hand; run inside the producing skill; run after another skill finishes.
- **Fresh-context reviewer:** a review by a session or subagent that did not share the builder's context.

## Sources

1. [Building verification loops in Claude Code with skills](https://claude.com/blog/building-verification-loops-in-claude-code-with-skills) (Delba de Oliveira, Claude blog, 2026-07-22). The gather-act-verify loop, built-in checks, turning manual corrections into skills, standalone/embedded/chained/PR modes, the team's four-skill chain, and the token-spend warning.
2. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Erik Schluntz & Barry Zhang, Anthropic, 2024-12-19). Workflows vs. agents, the five workflow shapes with gates, routing to smaller models, compounding errors, and "start simple".
3. [Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows) (Claude Code docs, read 2026-10). Workflows as scripted graphs, `parallel()` as a barrier, adversarial cross-checking, cost warnings and thresholds, per-stage model choice.
4. [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (Claude Code docs, read 2026-10). Giving the agent a check, `CLAUDE.md` commands, Writer/Reviewer sessions, fresh-context review against a plan, and the warning about reviewers over-reporting.
5. [Run Claude Code programmatically](https://code.claude.com/docs/en/headless) (Claude Code docs, read 2026-10). The `-p` flag, piping input, `--bare`, output formats.
6. [Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents) (LangChain/LangGraph docs, read 2026-10). Nodes, edges and state; gates as conditional edges; fan-out to an aggregator; orchestrator-worker with `Send`.
7. [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Prithvi Rajasekaran, Anthropic, 2026-03-24). Self-evaluation leniency, why a separate skeptical evaluator works better, and the effort needed to tune a QA agent.
8. [chrome-headless-shell](https://developer.chrome.com/blog/chrome-headless-shell) (Chrome for Developers, read 2026-10). The lightweight headless binary for screenshots and scraping, and its trade-off against full headless Chrome.
