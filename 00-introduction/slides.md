---
theme: ../theme
chapter: '00'
header: 'AI Fundamentals · 00 · Introduction'
layout: cover
---

# AI Fundamentals

**Working with and building on large language models**

Track 0 primer + 30 chapters

---

## Why this course

- AI tools change **every month**: new models, new features, new names
- The principles underneath change **much more slowly**
  - Models are stateless and read one stream of tokens
  - Context is a limited budget
  - Agents are only as good as the checks they can run
- Learn the **principles** and apply them across tools

<!-- The aim is that most of the course is still true in two years, even when every product name in it has changed. -->

---

## What you'll be able to do

- Get **consistently better answers** from chat assistants and reasoning models
- **Engineer the context** an agent works from, and keep it healthy over long tasks
- Work with **coding agents** that check their own work against a spec
- Understand **MCP, Skills and A2A**, and the security risks they bring
- Build and **measure** AI features: RAG, evals, cost and privacy
- **Design agent systems**: loops, subagents, graphs, harnesses, parallel sessions

---

## How each chapter works

**Key ideas** → explanation → **examples** → exercises → key takeaways

<!-- Read the article; use the deck for review or to teach the chapter to others. -->

---

## The map

<style>
table th, table td { padding: 0.45em 1em; }
</style>

| Track | Theme | Chapters |
|---|---|---|
| 0 | How LLMs work | Introduction |
| 1 | Using models | 01–02 |
| 2 | Context engineering | 03–05 |
| 3 | Working with coding agents | 06–08 |
| 4 | Tools, protocols and capabilities | 09–15 |
| 5 | Building AI features | 16–20 |
| 6 | Agent architecture | 21–27 |
| 7 | The bigger picture | 28–30 |

<!-- Each track builds on the ones before it, except Track 7, which can be read at any time. -->

---

## Track 0 · How LLMs work

- **Tokens** are pieces of text, including parts of words and punctuation
- The model predicts the **next token**, adds it to the sequence and repeats
- **Embeddings** represent tokens as numbers
- **Attention** combines information from tokens in the context
- Fluent output can still contain invented facts: **hallucinations**

<!-- Example: after "The capital of France is", the model may assign a high probability to "Paris". The prediction process can also produce a convincing error. Sources: https://www.youtube.com/watch?v=zjkBMFhNj_g and https://www.3blue1brown.com/lessons/gpt and https://www.youtube.com/watch?v=7xTGNNLPyMI -->

---

## Training, inference and context

- **Pre-training** learns patterns by predicting tokens in large datasets
- **Post-training** shapes assistant behaviour through examples and rewards
- **Inference** uses the trained model to generate an answer
- Your prompt changes the **context**, without updating the model's weights
- The **context window** limits what fits in each call, including history and documents

<!-- A chat application's saved history or memory must still be selected and supplied to the model. Training changes weights. Ordinary chat provides working material. Sources: https://www.youtube.com/watch?v=zjkBMFhNj_g and https://www.youtube.com/watch?v=7xTGNNLPyMI and https://www.3blue1brown.com/lessons/gpt -->

---

## Tools, workflows and agents

- A **tool** lets the application search, calculate or run a test for the model
- Tool results become input for the next model call
- A **workflow** follows steps defined in code
- An **agent** lets the model choose its next steps and tool calls

**Example:** a coding assistant proposes a fix, runs a test and revises using the result

<!-- The test gives evidence about whether the fix works. Agent systems need useful feedback and stopping conditions. Prefer simple designs until the task needs more autonomy. Source: https://www.anthropic.com/engineering/building-effective-agents -->

---

## Track 0 · Sources

- [Karpathy: Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g) (2023)
- [3Blue1Brown: But what is a GPT?](https://www.3blue1brown.com/lessons/gpt) (2024)
- [Karpathy: Deep Dive into LLMs like ChatGPT](https://www.youtube.com/watch?v=7xTGNNLPyMI) (2025)
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (2024)

Optional deeper reading. The primer here is enough to begin Track 1.

<!-- The article in this introduction folder includes the primer, a quick exercise and these sources. Revisit Building effective agents before Track 6. -->

---

## Track 1 · Using models

| # | Chapter | Core idea |
|---|---|---|
| 01 | Getting better LLM answers | Be specific, say why, give material, new task = new chat |
| 02 | Reasoning models | Short, goal-focused prompts; zero-shot first; planner vs. workhorse |

<!-- The day-one skills for anyone using a chat assistant. -->

---

## Track 2 · Context engineering

| # | Chapter | Core idea |
|---|---|---|
| 03 | Context engineering | Curate the smallest set of high-signal tokens, not one clever prompt |
| 04 | Context rot | Quality drops as the window fills; compact, clear, delegate, rewind |
| 05 | Compaction, notes, subagents | Three ways to work beyond one context window |

<!-- "Context engineering" spread in June 2025 after Tobi Lütke and Andrej Karpathy argued it described the real skill better than "prompt engineering". -->

---

## Track 3 · Working with coding agents

| # | Chapter | Core idea |
|---|---|---|
| 06 | Agent instruction files | Only what changes default behaviour; every line costs context |
| 07 | Verification first | Give the agent a check it can run |
| 08 | Spec-driven development | Spec → plan → tasks → implement, with checkpoints |

<!-- Chapter 07 is the single biggest improvement in the course: without a check, "looks done" is the only signal the agent has. -->

---

## Track 4 · Tools, protocols and capabilities

| # | Chapter | Core idea |
|---|---|---|
| 09 | MCP and AGENTS.md | Shared standards for tools and project context |
| 10 | How MCP servers work | Tools are described functions; the description guides the model |
| 11 | Agent Skills | Know-how as folders, loaded by progressive disclosure |
| 12 | Skill supply-chain security | Installing a skill is installing software |
| 13 | Prompt injection | Instructions hidden in content; limit the damage by design |
| 14 | Dynamic context discovery | Small static context; pull the rest on demand |
| 15 | A2A protocol | Agents discovering and delegating to other agents |

---

## Track 5 · Building AI features

| # | Chapter | Core idea |
|---|---|---|
| 16 | Embeddings and RAG | Retrieval pipeline, hybrid search, measuring retrieval |
| 17 | Prompting vs. RAG vs. fine-tuning | Instructions, knowledge or weights: pick the right lever |
| 18 | Evals | Error analysis first, small datasets, cheapest grader that works |
| 19 | Open-weight and local models | Open-weight ≠ open source; quantization; test on your task |
| 20 | Data privacy and token cost | Know where data goes; agents multiply cost |

---

## Track 6 · Agent architecture

| # | Chapter | Core idea |
|---|---|---|
| 21 | The harness matters | Results depend on finding the right context |
| 22 | Agent loops | Goal, action, check, stop |
| 23 | Subagents vs. agent teams | Report back vs. coordinate with each other |
| 24 | The advisor strategy | Cheap executor, strong advisor at decisive points |
| 25 | Graphs and verification | Multi-agent workflows; every node needs its own check |
| 26 | Agent harness design | Planner, generator, evaluator: keep doing separate from judging |
| 27 | Running agents in parallel | Isolation, handoffs, one owner per file |

---

## Track 7 · The bigger picture

| # | Chapter | Core idea |
|---|---|---|
| 28 | A lifecycle for agentic systems | Ask if you need an agent; gate on evidence; monitor |
| 29 | Reward hacking and misalignment | Learning to cheat generalized to worse behaviour |
| 30 | The forward deployed engineer | Fitting AI into how work really gets done |

<!-- Read these at any point; they put the technical tracks in context. -->

---

## Threads that run through every track

- **Context is a budget.** More is not better; curate it (03–05, 14, 21)
- **Checks beat hope.** Tests, evaluators, evals: give the system ground truth (07, 18, 22, 25, 26)
- **Agents act with your permissions.** Treat skills, tools and content as untrusted (12, 13, 20)
- **Measure before you optimize.** Evals come before prompting changes, RAG or fine-tuning (17, 18, 28)
- **Simplest thing that works.** A single call or plain code often beats an agent (22, 25, 28)

<!-- When a new tool appears, ask which of these threads it touches; that usually tells you how to use it well. -->

---

## Suggested paths

| If you want to… | Read |
|---|---|
| Use AI assistants well, starting today | Tracks 0–3 |
| Add tools and build AI features | + Tracks 4–5 |
| Design multi-step and multi-agent systems | + Track 6 |
| Understand the wider landscape | Track 7, any time |

Work through the tracks **in order**: later chapters assume earlier ones.

---

## Getting the most out of it

- **Do the exercises.** Each one makes a single principle visible in your own work
- **Try ideas on a real task**, not a toy example
- **Check the dates.** Anything marked "as of 2026-10" may have changed; the principle behind it usually hasn't
- **Follow the sources** when you need depth or the latest details

<!-- The "what lasts vs. what will date" section in each chapter is the quickest review. -->

---
layout: end
---

# Let's begin

- **Track 0 + 30 chapters**: principles first, products second
- **Track 0 is part of this introduction**, then chapter 01: Getting better LLM answers
