# The Advisor Strategy: A Cheaper Executor That Consults a Stronger Model When It Matters

> **Core idea in one sentence:** Let a faster, cheaper model run the whole task and consult a stronger model only at the decisions that shape the outcome, so you get much of the strong model's judgement while most tokens are generated at the cheaper rate, provided the executor recognises when to ask.

## TL;DR

- In the **advisor strategy**, a cheaper model (Sonnet or Haiku in Anthropic's examples) is the **executor**: it calls tools, edits and writes the answer. A stronger model (Opus) is the **advisor**: it reads the shared context and returns "a plan, a correction, or a stop signal" [1].
- The advisor "never calls tools or produces user-facing output" [1]. It changes decisions; the executor still does all the work.
- Anthropic reports that Sonnet with an Opus advisor scored 2.7 points higher on SWE-bench Multilingual than Sonnet alone while cost per task fell 11.9% [1]. Its docs add the caveat: "Results are task-dependent. Evaluate on your own workload." [2]
- As of 2026-10, it exists as a beta **API tool** (`advisor_20260301`) and as an experimental **Claude Code feature** (`/advisor`) [2][3].
- The weak spot is the **escalation decision**. In Claude Code, "the timing is model-driven rather than rule-based" [3]. Tell the executor when to consult if it under-calls.
- It belongs to an older family of cost-quality techniques: **routing** and **cascades** [4][5][6].

## The problem: one model is a compromise

A strong model reasons better but costs more per token and is slower. A cheaper model handles routine steps well but can go wrong on hard decisions. Claude Code's cost guide states the usual compromise: "Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning" [7]. The question is how to apply that inside a single long task, where most turns are routine and a few decide the outcome.

## Three ways to combine a cheap and a strong model

The advisor strategy is easier to understand next to two earlier ideas.

| Approach | Who decides | When the strong model runs | What the strong model produces |
|---|---|---|---|
| **Routing** | A classifier or router, before the task starts | For the whole request, if routed there | The full answer |
| **Cascade** | A check on the cheap model's answer | After the cheap answer is judged not good enough | The full answer |
| **Advisor** | The executor itself, mid-task | At chosen decision points | Guidance only, no tools, no final output |

- **Routing.** "Building effective agents" describes sending "easy/common questions to smaller, cost-efficient models" and harder ones to more capable models [4]. RouteLLM trains routers that "dynamically select between a stronger and a weaker LLM during inference", using human preference data, and reports cost reductions of over 2× in some cases without losing response quality [6].
- **Cascade.** FrugalGPT queries models in order of cost and stops when an answer is judged reliable. The authors report matching the best single model with "up to 98% cost reduction" on their benchmarks [5].
- **Advisor.** Routing and cascades pick *which model answers*. The advisor strategy keeps the cheap model in charge throughout and buys the strong model's judgement in small amounts, at the points where it changes the plan.

It also inverts the common orchestrator pattern. In Anthropic's multi-agent research system, an Opus lead delegates to Sonnet subagents [8]. The advisor strategy puts the smaller model in the driver's seat and uses frontier reasoning only when needed, "without decomposition, a worker pool, or orchestration logic" [1].

## How it works

The flow inside one task:

```
task ──> executor (cheaper model) works: reads, edits, runs tools
              │
              └─ reaches a hard decision, a recurring error, or "done?"
                       │
                       ▼
              advisor (stronger model) reads the full transcript,
              returns a plan / correction / stop signal (no tools)
                       │
                       ▼
              executor applies the guidance and continues
```

### In the API (as of 2026-10)

The Claude Platform exposes this as a server tool [2]:

- Add the beta header `advisor-tool-2026-03-01` and a tool with `type: "advisor_20260301"`, `name: "advisor"` and the advisor's `model` ID.
- The executor emits an advisor call with an **empty input**. "The executor signals timing, and the server supplies context." The advisor receives the executor's full transcript: system prompt, tool definitions, prior turns and tool results.
- The advisor "runs without tools and without context management". Its thinking is dropped and "only the advice text reaches the executor". All of this happens inside one `/v1/messages` request.
- `max_uses` caps advisor calls **per request**, not per conversation. `max_tokens` (minimum 1024) caps advisor output per call.
- Advisor tokens are billed at the advisor's rates and reported separately in `usage.iterations`, so you can track spend per tier.
- The advisor must be at least as capable as the executor. Equal-capability models can advise each other.

Typical advice is short: about 400 to 700 text tokens [1][2]. That is the source of the savings: "The cost savings come from the advisor not generating your full final output" [2].

### In Claude Code (as of 2026-10)

Claude Code wraps the same tool [3]:

- Set it with `/advisor opus`, the `advisorModel` setting, or `claude --advisor opus` for one session. `/advisor off` turns it off.
- It is **experimental** and requires the Anthropic API. It isn't available on Amazon Bedrock, Google Cloud, Microsoft Foundry or Claude Platform on AWS.
- Claude decides when to call it. It "tends to consult before committing to an approach, when an error keeps recurring, and before declaring a task done". There is no setting to force or cap calls; you steer it with instructions such as "consult the advisor before you continue".
- If its own evidence contradicts the advice, for example a recommended step fails, Claude surfaces the conflict rather than following the advice blindly.
- Each advisor call processes the full transcript anew. The advisor's read isn't cached between calls.

The docs compare it with related features: `opusplan` (Opus during plan mode, Sonnet for execution), subagents with their own `model`, and switching with `/model` [3].

## What the evidence says

Anthropic's announcement reports, with Opus 4.6 as the advisor in all runs [1]:

- **SWE-bench Multilingual:** Sonnet + Opus advisor scored 2.7 percentage points above Sonnet alone, with cost per agentic task down 11.9%.
- **BrowseComp:** Haiku + Opus advisor scored 41.2%, against 19.7% for Haiku alone.
- On BrowseComp and Terminal-Bench 2.0, Sonnet + Opus advisor scored higher than Sonnet alone at lower cost per task.

The API docs add two practical findings [2]. First, "for coding tasks, pairing a Sonnet executor at medium effort with an Opus advisor achieves intelligence comparable to Sonnet at default effort, at lower cost." Second, the right setup depends on your starting point. If you run Sonnet on complex tasks, an Opus advisor "keeps total cost similar or lower". If you run Haiku and want more intelligence, expect "higher cost than Haiku alone, but lower than switching the executor to a larger model."

These are vendor benchmarks. Anthropic's own recommendation is to run your existing evals on three setups: the cheaper model alone, the cheaper model with the advisor, and the strong model alone [1]. Topic 18 covers how to build those evals.

## When it fits, and when it doesn't

**Good fits** [2][3]:

- Long, multi-step tasks where most turns are routine but **plan quality decides the outcome**: large refactors, implementing a spec.
- Debugging where an error **keeps recurring** and a diagnosis unlocks mechanical fixes.
- Tasks you want **independently checked** before the agent declares them done.

**Poor fits** [2][3]:

- Short or single-turn tasks with nothing to plan.
- Work where **every turn** needs the strongest model. Switch the main model instead.
- Products where users already pick their own cost-quality trade-off.

Two limits follow from how the pattern works:

- **The advice is only as good as the escalation.** The executor decides when to call. Anthropic's suggested prompt tells it to call "BEFORE substantive work", when stuck, and when it believes the task is complete, and on longer tasks "at least once before committing to an approach and once before declaring done" [2]. Nudging helps some executors more than others. In Anthropic's tests a mid-conversation nudge raised Haiku's pass rate by about 7 points, had no measurable effect on Sonnet, and slightly lowered Opus's [2].
- **The advisor improves decisions, not execution.** It has no tools and writes no code. Speed, thoroughness and how well the work is split up still depend on the executor. If the hard part of a task is the sheer amount of interdependent work rather than a few key decisions, running the strong model as the main agent may finish sooner and with fewer wrong turns.

A practical habit: write explicit escalation triggers into your instructions, for example "consult the advisor before any change that touches the data model, before starting a new feature, and after two failed attempts at the same fix."

## What lasts vs. what will date

- **Durable principles:**
  - Spend expensive reasoning where it changes the outcome. Most agent turns are routine.
  - Separating *deciding* from *doing* lets a cheaper model do the doing.
  - Escalation is only as good as its trigger. If the worker decides when to ask for help, it will sometimes not ask.
  - Routing, cascades and advisors are three answers to the same cost-quality question. Pick by when the strong model is needed and what it should produce.
  - Measure on your own workload before switching.
- **Time-sensitive details:**
  - The tool name `advisor_20260301`, the beta header, parameters and pairing rules (as of 2026-10).
  - Claude Code's `/advisor` command, its experimental status and provider limits (as of 2026-10).
  - The specific model pairings and benchmark numbers (Opus 4.6 as advisor, April 2026).

## Try it yourself

1. **Audit a session.** Mark each turn of a recent agent session as routine or judgement-heavy. Estimate what share needed your strongest model.
2. **Manual advisor.** With a cheaper model, when it fails twice on the same problem, paste the context into a stronger model and ask only for a diagnosis and plan, no code. Hand the plan back.
3. **Three-way comparison.** Run one medium task with the cheap model alone, with the cheap model plus advisor (`/advisor opus` in Claude Code), and with the strong model alone. Compare tokens (`/usage`), time and bugs found afterwards.
4. **Escalation rules.** Add explicit triggers to your instructions and rerun. Did the executor consult more reliably, and did results change?
5. **Find the ceiling.** Pick a task with many interacting parts and note where the executor went wrong despite the advice. Was it a decision or an execution problem?

## Key terms

- **Executor:** the cheaper main model that calls tools, makes changes and writes all user-facing output.
- **Advisor:** a stronger model consulted mid-task. It reads the transcript and returns guidance, never tool calls or final output.
- **Advisor strategy:** cheap executor plus on-demand strong advisor, the reverse of a strong orchestrator delegating to cheap workers.
- **Escalation trigger:** the condition under which the executor consults the advisor, such as a new approach, a recurring error or "done".
- **Model routing:** choosing which model handles a request before it runs.
- **LLM cascade:** trying cheaper models first and escalating only when the answer isn't judged good enough.
- **Server tool:** a tool executed on Anthropic's side within the same API request, such as the advisor.

## Sources

1. [The advisor strategy: Give agents an intelligence boost](https://claude.com/blog/the-advisor-strategy) (Anthropic, 2026-04-09). The executor/advisor roles, the inversion of the orchestrator pattern, benchmark and cost results, and the recommendation to compare three setups on your own evals.
2. [Advisor tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) (Claude Platform docs, Anthropic, accessed 2026-10-14). The API mechanics, parameters, billing, pairing rules, suggested timing prompt, nudging results, good and poor fits.
3. [Escalate hard decisions with the advisor tool](https://code.claude.com/docs/en/advisor) (Claude Code docs, Anthropic, accessed 2026-10-14). The `/advisor` command, when Claude consults, cost and caching behaviour, requirements, and comparison with opusplan and subagents.
4. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Erik Schluntz and Barry Zhang, Anthropic, 2024-12-19). The routing workflow, including sending easy questions to smaller models.
5. [FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance](https://arxiv.org/abs/2305.05176) (Chen, Zaharia and Zou, arXiv 2023-05). The LLM cascade and other cost-reduction strategies.
6. [RouteLLM: Learning to Route LLMs with Preference Data](https://arxiv.org/abs/2406.18665) (Ong et al., arXiv 2024-06, rev. 2025-02). Learned routers that choose between a strong and a weak model per query.
7. [Manage costs effectively](https://code.claude.com/docs/en/costs) (Claude Code docs, Anthropic, accessed 2026-10-14). Model-choice guidance for Sonnet vs. Opus.
8. [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (Hadfield et al., Anthropic, 2025-06-13). The opposite pattern: a strong lead model delegating to cheaper subagents.
