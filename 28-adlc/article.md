# A Lifecycle for Agentic Systems: Building Software That Doesn't Behave the Same Way Twice

> **Core idea in one sentence:** An agent's behaviour comes from a model, prompts, tools and context together and varies from run to run, so the lifecycle for building one adds work that classic software rarely needed: decide whether an agent is warranted at all, assign responsibility between people and the agent, prove value on real data before the full build, evaluate continuously, and treat launch as the start of monitoring.

## TL;DR

- **Start by asking whether you need an agent.** Anthropic, OpenAI and Microsoft all advise using the simplest thing that works; deterministic code or a single LLM call often does [1][2][3].
- **Write the decision rules down early:** testable hypotheses, baseline KPIs, and which decisions the agent may take alone versus with human approval [3][4][6].
- **Gate the build on evidence.** Pilot the hardest steps on real data and make an explicit go/no-go decision [3][6].
- **Behaviour lives in more than code.** Version prompts and tool definitions like code, and run evals in CI [4].
- **Measure distributions, not single runs.** Run several trials per case and track cost per task [5].
- **Launch is the start of operations.** Roll out in stages, monitor behaviour, capture user feedback, and keep re-evaluating as models and data change [2][6][7].

## About the name "ADLC"

"Agent development lifecycle" or "agentic development lifecycle" (ADLC) describes the process of building and operating agentic systems. Vendors use different phase definitions; there is no single standard. The seven phases below cover problem framing, responsibility, design, validation, implementation, testing and operations.

There are also two meanings to keep apart. This topic is about the lifecycle of an **agentic system you are building**: software whose decisions come from a model. Using a coding agent to write that software is a separate matter, covered in topics 06–08.

## Why the classic lifecycle strains

A traditional software lifecycle runs requirements, design, build, test, deploy and maintain. It quietly assumes **determinism**: the same input gives the same output, so behaviour can be specified up front and checked with pass/fail tests.

Agentic systems weaken that assumption in three ways:

- **Run-to-run variation.** Anthropic's eval guide states plainly that "agent behavior varies between runs", so a single result is hard to interpret [5]. Agents also act over many turns, so early errors can compound [1][5].
- **Behaviour spread across components.** OpenAI describes an agent as three parts: model, tools and instructions [2]. Microsoft's build guidance also treats knowledge sources and memory as separate design decisions [4]. Changing any one of them can change what the system does.
- **Change after launch.** NIST's AI Risk Management Framework warns that risks measured "in a laboratory or a controlled environment" may differ from those in "operational, real-world settings", and that AI systems may need corrective maintenance due to data, model or concept drift [6].

The phases still map loosely onto the classic ones. What changes is the work inside each.

## The seven phases

| # | Phase | Classic counterpart | What's added |
|---|---|---|---|
| 1 | Frame the problem | Planning | "Do we need an agent?" test; testable hypotheses |
| 2 | Scope and responsibility | Requirements | Baseline KPIs; human–agent responsibility split |
| 3 | Design | Design | Workflow vs. agent; model, tool and cost choices |
| 4 | Prove value | (rarely explicit) | Pilot on real data; go/no-go gate |
| 5 | Build with evals | Development | Prompts and tools versioned like code; evals in CI |
| 6 | Test as a system | Testing | Repeated trials, outcome grading, red teaming |
| 7 | Deploy and operate | Deploy + maintain | Staged rollout, behavioural monitoring, feedback loop |

### Phase 1: Frame the problem

The first question is whether an agent is the right tool. Anthropic recommends "finding the simplest solution possible" and notes this "might mean not building agentic systems at all", because agents "trade latency and cost for better task performance" [1]. OpenAI lists where agents earn their keep: complex judgement calls with exceptions, rule sets that have become hard to maintain, and heavy reliance on unstructured data. Otherwise, "a deterministic solution may suffice" [2]. Microsoft's guidance adds two clear "no" cases: structured, predictable tasks (use regular code) and static question answering over fixed documents (use classic RAG) [3].

If an agent does look justified, find out how the work is done today and where it breaks, then write **testable hypotheses**: "an agent can draft first replies for billing tickets that agents accept without edits at least 70% of the time." Each hypothesis needs a metric that could prove it wrong.

A coding agent's planning mode can help here, provided you ask it about the *behaviour* of the system (users, decisions, failure modes, assumptions) and not yet about code.

### Phase 2: Scope and responsibility

Microsoft advises setting success metrics "before any development begins": identify the KPIs the agent must improve and **record today's performance as a baseline** [3]. Without a baseline, you can't later tell whether the agent helped.

The new artefact in this phase is a **human–agent responsibility model**. NIST frames human–AI configurations as a spectrum "from fully autonomous to fully manual" and asks organisations to define and differentiate human roles in "decision making and overseeing AI systems" [6]. Practical guidance makes this concrete:

- OpenAI names two triggers for handing control to a person: exceeding a **failure threshold** (too many retries) and **high-risk actions** that are "sensitive, irreversible, or have high stakes", such as cancelling orders or authorising large refunds [2].
- Microsoft recommends an **agent charter** listing what the system is responsible for, each agent's role, and prohibited actions, plus human confirmation for high-impact actions like database writes or financial transactions [4].

A simple format: list every decision in the workflow and mark it *agent alone*, *agent proposes, human approves*, or *human only*.

### Phase 3: Design

Anthropic distinguishes **workflows**, where "LLMs and tools are orchestrated through predefined code paths", from **agents**, which "dynamically direct their own processes and tool usage" [1]. Its catalogue of workflow patterns (prompt chaining, routing, parallelisation, orchestrator–workers, evaluator–optimizer) covers many problems without full autonomy [1]. Microsoft goes further and recommends deterministic workflows for critical business logic [4].

Other design decisions:

- **Start with one agent.** OpenAI's advice is to "maximize a single agent's capabilities first" and add agents only when needed [2] (see topic 23).
- **Choose models against a baseline.** Prototype with the most capable model to set a performance baseline, then try smaller models where results stay acceptable [2]. Microsoft frames this as matching model tier to task complexity to control cost [4].
- **Design the tools carefully.** Anthropic suggests investing as much in the agent–computer interface as in a human interface: clear tool descriptions, examples, and formats that are hard to misuse [1].
- **Define success before code**, so the evals in phases 4–6 test something agreed.

### Phase 4: Prove value

Before the full build, test the riskiest assumptions on real data. Microsoft's advice is to "pilot the hardest steps", and to use business metrics as **decision gates**: continue, change direction or stop, based on "the data, not assumptions or optimism" [3]. NIST describes the same gate after its MAP function: enough context "to inform an initial go/no-go decision about whether to design, develop, or deploy an AI system" [6].

This phase also produces the first **evaluation set**. Anthropic suggests that "20-50 simple tasks drawn from real failures is a great start" [5]. Keep it: it becomes the regression suite later (topic 18 covers how to build one).

### Phase 5: Build with evals in the loop

Because behaviour lives in prompts, tool definitions and model choice as well as code, treat all of them as versioned artefacts. Microsoft recommends storing agent instructions in source control so changes get review and rollback, and integrating evaluations into CI/CD pipelines "to catch issues before production deployment" [4].

Two practices follow:

- **Validate as you go.** Small prompt or tool changes can shift behaviour, so re-run the eval set after each meaningful change instead of saving testing for the end.
- **Test tools in a sandbox.** Anthropic recommends "extensive testing in sandboxed environments" [1], and Microsoft adds checking how the agent handles API failures and unexpected payloads [4].

Context management belongs here too: what the agent sees at each step strongly affects quality and cost (topics 03–05).

### Phase 6: Test as a system

System testing for agents differs from unit testing:

- **Repeat trials.** Anthropic distinguishes *pass@k* (at least one of k attempts succeeds) from *pass^k* (all k succeed). At 75% per trial, pass^3 is about 42%, which is the number that matters for a customer-facing agent [5].
- **Grade the outcome**, not the exact path, so valid alternative solutions aren't penalised [5].
- **Keep two suites:** capability evals that start with low pass rates, and regression evals that should stay near 100% [5].
- **Red-team it.** Microsoft calls for AI red teaming against prompt injection and system-prompt extraction [4] (topic 13).
- **Test in deployment-like conditions.** NIST asks that performance be "demonstrated for conditions similar to deployment setting(s)" [6].

Track latency, token usage and cost per task alongside quality [5]. A change that raises the pass rate but multiplies cost may still be a regression.

### Phase 7: Deploy and operate

**Deploy in stages.** Google's SRE workbook defines canarying as "a partial and time-limited deployment of a change in a service and its evaluation": a small share of traffic gets the change, its metrics are compared with the rest, and the rollout proceeds or rolls back [7]. For agents, include behavioural metrics (task success, escalation rate, user corrections) alongside error rates. OpenAI notes that human intervention matters most "early in deployment", to find failures and edge cases [2].

**Monitor behaviour, not only uptime.** Microsoft recommends tracing decision points and setting baselines and alerts for latency, cost per interaction and success rate [4]. NIST's MANAGE function calls for post-deployment monitoring plans that capture user input and include override, incident response and change management [6].

**Close the loop.** No single signal is enough. Anthropic lists automated evals, production monitoring, A/B tests, user feedback and manual transcript review as complementary layers, comparing them to the Swiss cheese model in which "no single evaluation layer catches every issue" [5]. Turn new production failures into eval cases, add guardrails as real edge cases appear [2], and re-run the regression suite before any model upgrade. Microsoft also asks teams to compare live results with the original KPIs on a schedule and decide whether to refine, retire or reinvest [3].

## What lasts vs. what will date

- **Durable principles:**
  - Use the simplest design that meets the goal; an agent is a cost you should justify.
  - Set baselines and decide human–agent responsibilities before building.
  - Gate investment on evidence from real data.
  - Version everything that shapes behaviour, and evaluate with repeated trials.
  - Roll out gradually and keep evaluating after launch, because models, data and users change.
- **Time-sensitive details (as of 2026-10):** the "ADLC" label and any particular phase count; vendor platform names in Microsoft's guidance; the "20–50 tasks" starting heuristic; which models count as "most capable" for baselines.

## Try it yourself

1. **Run the "do we need an agent?" test.** For a task you want to automate, answer OpenAI's three criteria and Microsoft's two "no" cases [2][3]. Write one paragraph on whether code, a single LLM call, a workflow or an agent fits.
2. **Write three hypotheses with baselines.** For each, record today's number (time, error rate, cost) and the threshold that would count as success.
3. **Draft a responsibility table.** List every decision in the workflow and mark it *agent alone*, *agent proposes / human approves* or *human only*, with one line of reasoning each.
4. **Build a 20-case eval set and run it five times.** Compute pass@1 and pass^5. The gap is your consistency problem.
5. **Plan a canary.** Choose the first user group, three behavioural metrics, the thresholds that trigger rollback, and who decides.

## Key terms

- **Agentic system**: software in which a model decides which steps and tools to use, rather than following a fixed code path.
- **Workflow (vs. agent)**: LLM calls and tools orchestrated through predefined code paths.
- **Human–agent responsibility model**: a documented split of which decisions the agent makes alone, which need human approval, and which stay human.
- **Decision gate (go/no-go)**: a point where measured results decide whether the project continues, changes or stops.
- **Evaluation set**: real inputs with expected outcomes, run on every change.
- **pass^k**: the probability that all k trials of a task succeed; a measure of consistency.
- **Canary release**: a partial, time-limited deployment compared against the rest of traffic before wider rollout.
- **TEVV**: NIST's term for test, evaluation, verification and validation, performed throughout the lifecycle.

## Sources

1. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Anthropic Engineering, Erik Schluntz & Barry Zhang, 2024-12-19). Workflows vs. agents, the simplest-solution principle, workflow patterns, sandboxed testing and tool (ACI) design.
2. [A practical guide to building agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) (OpenAI, 2025). When to build an agent, model/tools/instructions, baseline-first model selection, single-agent-first orchestration, guardrails and human-intervention triggers.
3. [Business plan for AI agents](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ai-agents/business-strategy-plan) (Microsoft Cloud Adoption Framework, 2026-02-13). When not to use agents, use-case scoring, piloting the hardest steps, baseline KPIs and go/no-go decision gates.
4. [Process to build agents across your organization](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ai-agents/build-secure-process) (Microsoft Cloud Adoption Framework, 2025-12-01). Agent charters, deterministic workflows, instructions under version control, sandboxed tool testing, tracing, evals in CI/CD and red teaming.
5. [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (Anthropic Engineering, 2026-01-09). Run-to-run variation, starting eval sets, capability vs. regression suites, pass@k vs. pass^k, outcome grading and layered evaluation.
6. [Artificial Intelligence Risk Management Framework (AI RMF 1.0), NIST AI 100-1](https://doi.org/10.6028/NIST.AI.100-1) (NIST, 2023-01). Lifecycle-wide TEVV, human–AI configurations and roles, go/no-go after MAP, lab vs. real-world risk, and post-deployment monitoring.
7. [Canarying releases](https://sre.google/workbook/canarying-releases/) (Google, The Site Reliability Workbook, ch. 16, 2018). Definition and practice of partial, time-limited rollouts compared against a control population.
