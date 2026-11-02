---
theme: ../theme
chapter: '28'
header: 'AI Fundamentals · 28 · Agentic development lifecycle'
layout: cover
---

# A Lifecycle for Agentic Systems

Agents **vary from run to run**, so building them needs extra steps: justify the agent, assign responsibility, prove value, evaluate continuously, and keep watching after launch.

---

## Why the classic lifecycle strains

- Classic SDLC assumes **determinism**: same input, same output, pass/fail tests
- "Agent behavior varies between runs" (Anthropic)
- Behaviour comes from **model + instructions + tools + knowledge + memory**
- Lab results can differ from **real-world** results; drift continues after launch (NIST)

<!-- The phases still map loosely onto requirements, design, build, test, deploy, maintain. What changes is the work inside each one. -->

---

## About the name "ADLC"

- Several vendors and consultancies use "ADLC", each with **different phases**
- **No standards body** defines it; it is not a coding-tool feature
- This topic assembles the lifecycle from Anthropic, OpenAI, Microsoft, NIST and Google guidance
- Scope: the lifecycle of an **agentic system you build**, not how you use a coding agent (topics 06–08)

<!-- Treat the seven phases as a teaching structure, not a standard. -->

---

## Seven phases at a glance

| # | Phase | What's added |
|---|---|---|
| 1 | Frame the problem | "Do we need an agent?"; testable hypotheses |
| 2 | Scope & responsibility | Baseline KPIs; human–agent split |
| 3 | Design | Workflow vs. agent; models, tools, cost |
| 4 | Prove value | Pilot on real data; **go/no-go** |
| 5 | Build with evals | Prompts versioned like code; evals in CI |
| 6 | Test as a system | Repeated trials, outcome grading, red teaming |
| 7 | Deploy & operate | Staged rollout, monitoring, feedback loop |

<!-- Each row maps roughly onto a classic phase; phase 4 is the one classic projects rarely make explicit. -->

---

## 1 · Frame the problem

- Anthropic: find "the simplest solution possible", which may mean **no agent**
- OpenAI: agents fit **complex judgement**, **brittle rule sets**, **unstructured data**
- Microsoft: predictable task → **code**; static Q&A → **classic RAG**
- If an agent fits, write **testable hypotheses** with a metric that could disprove each

<!-- A planning mode in a coding agent can help map behaviour, failure modes and assumptions, as long as you keep it away from code at this stage. -->

---

## 2 · Scope & responsibility

- Set KPIs and **record today's baseline** before development (Microsoft)
- NIST: human–AI configurations range from **fully autonomous to fully manual**; define roles
- OpenAI's escalation triggers: **failure thresholds** and **high-risk actions**
- Microsoft's **agent charter**: responsibilities, roles, prohibited actions

Mark each decision: **agent alone** · **agent proposes, human approves** · **human only**

<!-- Irreversible or high-stakes actions start as "human approves" or "human only" until the agent has a track record. -->

---

## 3 · Design

- **Workflow** (predefined code paths) vs. **agent** (model directs its own steps)
- Patterns: chaining, routing, parallelisation, orchestrator–workers, evaluator–optimizer
- **One agent first**; add more only when needed (topic 23)
- Prototype with the most capable model for a **baseline**, then try smaller ones
- Invest in **tool design**; define success before code

<!-- Microsoft recommends deterministic workflows for critical business logic. Anthropic: agents trade latency and cost for task performance. -->

---

## 4 · Prove value

- **Pilot the hardest steps** on real data (Microsoft)
- Business metrics as **decision gates**: continue, pivot or stop
- NIST: an initial **go/no-go** decision before design, development or deployment
- Start the eval set: **20–50 tasks from real failures** (Anthropic)

> Decide on "the data, not assumptions or optimism."

<!-- The eval set built here becomes the regression suite later. Topic 18 covers how to build it. -->

---

## 5 · Build with evals in the loop

- Behaviour lives in **code, prompts, tool definitions, model choice**
- Keep instructions in **source control**: review, audit trail, rollback
- Run **evals in CI/CD** before deployment
- Test tools in a **sandbox**, including API failures and odd payloads
- Manage context deliberately (topics 03–05)

<!-- Re-run the eval set after each meaningful change; a one-word prompt edit can shift behaviour. -->

---

## 6 · Test as a system

| Classic testing | Agentic testing |
|---|---|
| One run, pass/fail | Several trials: **pass@k** and **pass^k** |
| Check exact code path | **Grade the outcome** |
| One test suite | **Capability** + **regression** suites |
| Functional checks | Plus **red teaming** (prompt injection) |

At 75% per trial, pass^3 ≈ **42%**. Track **cost per task** too.

<!-- NIST asks for performance demonstrated under conditions similar to deployment. -->

---

## 7 · Deploy in stages

- **Canary**: "a partial and time-limited deployment of a change … and its evaluation" (Google SRE)
- Compare canary vs. control; proceed or **roll back**
- Include **behavioural** metrics: task success, escalations, user corrections
- Human intervention matters most **early in deployment** (OpenAI)

<!-- Microsoft: trace decision points and set baselines and alerts for latency, cost per interaction and success rate. -->

---

## Operate: close the loop

- NIST MANAGE: post-deployment monitoring with **user input, override, incident response, change management**
- Layer signals: evals, monitoring, A/B tests, user feedback, transcript review
- "No single evaluation layer catches every issue" (Anthropic)
- New production failures → **new eval cases**; re-run regressions before **model upgrades**
- Review KPIs on a schedule: refine, retire or reinvest

<!-- Microsoft frames the last point as comparing live performance with the original target KPIs. -->

---

## What lasts vs. what will date

| Durable | Will date (as of 2026-10) |
|---|---|
| Simplest design that meets the goal | The "ADLC" label and phase counts |
| Baselines and responsibility split up front | Vendor platform names |
| Evidence-based go/no-go gates | "20–50 tasks" starting heuristic |
| Version all behaviour-shaping artefacts | Which model is "most capable" |
| Staged rollout and continuous evaluation | |

<!-- The ideas come from risk management and SRE practice and don't depend on the label. -->

---

## Try it yourself

1. Run the **"do we need an agent?"** test on a real task
2. Write **3 hypotheses**, each with today's baseline and a success threshold
3. Draft a **responsibility table** for every decision in the workflow
4. Build a **20-case eval set**, run it 5 times, compare pass@1 and pass^5
5. Plan a **canary**: first group, 3 behavioural metrics, rollback thresholds

<!-- Exercise 4 makes run-to-run variation concrete. -->

---
layout: end
---

# Key takeaways

- **Justify the agent** and **fix responsibility** before you build
- **Gate on evidence**: pilot on real data, measure distributions, not single runs
- **Launch starts operations**: staged rollout, monitoring, feedback into evals

anthropic.com/engineering/building-effective-agents\
learn.microsoft.com/azure/cloud-adoption-framework/ai-agents/business-strategy-plan\
doi.org/10.6028/NIST.AI.100-1
