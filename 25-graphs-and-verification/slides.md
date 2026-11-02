---
theme: ../theme
chapter: '25'
header: 'AI Fundamentals · 25 · Graphs and verification'
layout: cover
---

# From Loops to Graphs

Parallel agents are faster, but **an unchecked node spoils everything downstream**, so every node needs a check by someone who didn't build it.

---

## Loop vs. graph

- A **loop**: gather context → act → verify → repeat
- Each step waits for the last, even when they're unrelated
- A **graph**: split the task into nodes; independent nodes run **at once**
- Gains: speed, and a **cheaper model on easy nodes**
- Costs: **more tokens in total**, and failures are hard to trace

<!-- The loop framing is from the Claude Code team's verification post. Routing easy work to smaller models is one of the patterns in Anthropic's "Building effective agents". -->

---

## The standard workflow shapes

| Shape | What it does |
|---|---|
| **Prompt chaining** | Steps in sequence, with a programmatic **gate** between them |
| **Routing** | Classify, then send to a specialised handler or model |
| **Parallelization** | Sectioning (split the work) or voting (same task, many times) |
| **Orchestrator-workers** | A lead splits, delegates and synthesises |
| **Evaluator-optimizer** | Generate → evaluate → feed back, in a loop |

<!-- From Anthropic's "Building effective agents" (Dec 2024). Its advice: start with the simplest design and add structure only when it demonstrably helps. -->

---

## What a graph is made of

| Part | What it is |
|---|---|
| **Node** | One unit of work: a model call, a tool, or an agent |
| **Edge** | What runs next: fixed, or conditional ("pass or retry") |
| **State** | What flows between nodes; reducers merge parallel updates |

- LangGraph is one framework that makes these explicit
- A **gate** is a conditional edge on a node's output

<!-- LangGraph's examples: three writers fan out from the start and feed one aggregator; a gate sends a draft to the end or back for improvement. -->

---

## Two shapes worth knowing

```
   Diamond                  Fan-in at a barrier
    task                      same problem
  /  |  \                   /    |    \
 A   B   C                lens1 lens2 lens3
  \  |  /                   \    |    /
 synthesiser              [ BARRIER: wait for all ]
                                 |
                            merge, then fix
```

- **Diamond** = orchestrator-workers
- **Barrier**: nothing moves until **every** branch has reported

<!-- In Claude Code's dynamic workflows, parallel() runs a set of agent tasks and waits for all of them: that is the barrier. It stops one reviewer's fix landing before the other angles are heard. -->

---

## The costs of going wide

- Small models per node cut cost **per agent**, not in total
- Claude Code docs: a workflow can use **meaningfully more tokens** than a conversation
- Large-run warning above 25 agents or 1.5M projected tokens (as of 2026-10)
- Try a **small slice** first
- A wrong final answer gives **no trail** back to the node that caused it

> Verify **inside** the graph, at each node, not once at the end.

<!-- Anthropic's agents paper names compounding errors as the main risk of autonomy. An early error is built on by every node downstream. -->

---

## Built-in checks and their limits

- **Toolchain:** linters, type checkers and tests. Put the commands in `CLAUDE.md`
- **`/verify`:** build, run and observe the changed app
- **Code Review** (multi-agent PR review), GitHub Actions, spec validation, rubric graders
- Tests catch **breakage**, not intent
- Turn the **manual corrections you keep repeating** into skills

<!-- From the Claude Code team's verification-loops post (July 2026). Their example project rule: reject any migration that drops a column without a backfill step. Feature status as of 2026-10. -->

---

## The check that matters most in a graph

- Each node sees **only its slice**
- It can't notice it has drifted from the whole
- So review each output **against the original requirements**
- Template: "review the diff against PLAN.md… report **gaps, not style preferences**"

<!-- The template is from Claude Code's best-practices guide (adversarial review step). -->

---

## Where checks run

| Mode | How | Good for |
|---|---|---|
| **Standalone** | You invoke it | Security scan, accessibility audit |
| **Embedded** | Inside the producing skill | Checks tied to one kind of output (editable skills only) |
| **Chained** | Runs after another skill; wrappers for skills you can't edit | Several angles in sequence |
| **On every PR** | In CI | Team-wide gates, once stable |

<!-- If you run a standalone check after every change, embed or chain it. Hold off on PR-wide gates while the chain is still changing. -->

---

## One check per angle, merged once

- Claude Code team's chain: **`/code-review` → `/simplify` → `/verify` → `/design`** (against `DESIGN.md`)
- One concern per check, measured its own way
- In parallel: one agent per angle → **barrier** → one merge step that ranks and de-duplicates
- Chains **raise token spend**: test before rolling out

<!-- Dynamic-workflow docs include this pattern: review every changed file, then merge per-file findings into one ranked summary. -->

---

## Cheaper visual checks

- UI checks usually drive a browser and take screenshots
- A full browser is slow when repeated inside a graph
- **chrome-headless-shell**: lightweight binary for screenshots
- Trade-off: less faithful than full Chrome in headless mode
- Light tool for frequent checks, **faithful one for the final pass**

<!-- Chrome's documentation positions the shell for lightweight automation and new headless mode for high-accuracy end-to-end testing. -->

---

## Who judges

- The builder is biased: agents "reliably skew positive when grading their own work"
- Use a **fresh context**: a reviewer session or subagent that sees only the diff and criteria
- `git diff main | claude -p "<review prompt>"`
- `--bare` skips local hooks, skills and `CLAUDE.md` for reproducible runs

<!-- The quote is from Anthropic's harness-design post. Writer/Reviewer sessions are in Claude Code's best practices. The -p behaviour is from the headless docs (as of 2026-10). -->

---

## Reviewers fail in both directions

- **Over-reporting:** "A reviewer prompted to find gaps will usually report some, even when the work is sound"
- **Under-testing:** out of the box, a QA agent tested superficially and approved flawed work
- In a graph, both errors **multiply**
- Judge a reviewer by **correct** findings, not by how many
- Scope it, calibrate it, and don't economise on the judging model

<!-- First quote: Claude Code best practices. Second point: Anthropic's harness-design post, where tuning the QA agent took several rounds of reading its logs. See topic 18 for calibrating judges. -->

---

## What lasts vs. what will date

**Lasts**
- Parallelism trades total cost and traceability for speed
- Gate each step that others build on. Check against the requirements
- Independent, focused reviewers, joined at a barrier
- Start simple; add structure only when it helps

**Will date**
- Skill names, Code Review / rubric features, workflow limits, `-p` / `--bare`, LangGraph APIs

<!-- All product details as of 2026-10. -->

---

## Try it yourself

1. Draw one workflow as nodes and edges: chain, diamond or barrier?
2. Add a programmatic **gate** after the riskiest node
3. Run a requirements review in a fresh `claude -p` session on a diff
4. Grade the grader: small vs. large model, count **correct** findings
5. Split one review into three angles plus a merge step

<!-- Exercise 4 shows directly why the judging step deserves a strong model. -->

---
layout: end
---

# Key takeaways

- Graphs give you **speed and per-node model choice**, at a higher total cost
- **Gate every node** and check against the original requirements
- The judge should be **independent, scoped and strong**

claude.com/blog/building-verification-loops-in-claude-code-with-skills · anthropic.com/engineering/building-effective-agents · code.claude.com/docs/en/workflows
