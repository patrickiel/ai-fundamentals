---
theme: ../theme
chapter: '03'
header: 'AI Fundamentals · 03 · Context engineering'
layout: cover
---

# Context Engineering for AI Coding

**Design the information an agent works from: a small core always loaded, everything else pulled in when the current step needs it**

---

## Where the term came from

> "the art of providing all the context for the task to be plausibly solvable by the LLM"
> Tobi Lütke, 2025-06-19

> "filling the context window with just the right information for the next step"
> Andrej Karpathy, 2025-06-25

- They **named** a practice builders already followed; they didn't invent it

<!-- Lütke posted first; Karpathy's "+1" a week later spread the term widely. -->

---

## Prompt engineering is one part

- **Prompt engineering:** writing and organising the instructions (Anthropic)
- **Context engineering:** curating *all* tokens the model sees at inference
- What goes into context:
  - Instructions · user prompt · examples · history and state
  - Long-term memory · retrieved data (RAG) · tools · output format
- Karpathy: the **app around the model** matters too; it's not a "ChatGPT wrapper"

<!-- Component list draws on Karpathy's post and Philipp Schmid's checklist; the definitions are from Anthropic's engineering post. -->

---

## The central tension

- Tempting: give the model **everything**
- But recall degrades as context grows: **context rot**
- Models have an **attention budget**; every token draws on it
- When the window fills, agents start "forgetting" instructions (Claude Code docs)

Goal: the **smallest possible set of high-signal tokens** for the outcome

<!-- Topic 04 covers the evidence for context rot in depth. -->

---

## Just-in-time context

- Keep **lightweight references**: file paths, links, stored queries
- Load content **with tools when needed**
- Claude Code's hybrid: `CLAUDE.md` loaded up front; glob/grep fetch files on demand
- Link **external docs** (library references) so the agent reads them at the right step

<!-- This is Anthropic's description of just-in-time retrieval in "Effective context engineering for AI agents". -->

---

## The workflow

```
Requirements (PRD / spec)
        │
        ▼
Plan + focused reference files   ← generated once, saved to disk
        │
        ▼
Short always-on rule: "for X, read file Y"
        │
        ▼
Fresh sessions execute stage by stage, verifying each
```

<!-- Cole Medin's public template is one worked example: feature request, generate a plan, execute the plan. -->

---

## Steps 1–2: requirements and focused files

- Write a **spec**: features, out-of-scope, chosen stack, a final verification step
- Tip from Claude Code's docs: have the agent **interview you**, then write the spec to a file

| File | Used for |
|---|---|
| Implementation plan | Staged tasks + checks |
| Project structure | Layout before creating files |
| UI/UX conventions | Interface work |
| Known issues | Check before debugging |

<!-- Generating these files can fill one session's context; that's fine, because the knowledge then lives on disk. -->

---

## Step 3: keep always-on rules short

```text
- Implementing a feature    → read docs/implementation-plan.md
- Working on UI             → read docs/ui-conventions.md
- Creating files or folders → check docs/project-structure.md
- Hitting an error          → check docs/known-issues.md first
```

- **Cursor:** `.cursor/rules`; always / intelligent / by glob / manual; keep rules **under 500 lines**
- **Claude Code:** `CLAUDE.md` **under 200 lines**; path-scoped `.claude/rules/`; `@imports` still load at launch
- Test for each line: "Would removing this cause Claude to make mistakes?"

<!-- Mechanisms and limits as of 2026-10; check current docs. -->

---

## Steps 4–5: survive resets, verify stages

- A written plan = **memory that survives a context reset**
- New session: "continue from the next stage of the plan"
- Models are "extremely effective at discovering state from the local filesystem": progress notes, test files, git
- **Foundations first**, and give each stage a **check the agent can run**

<!-- Topic 05 covers notes and compaction; topic 07 covers verification. -->

---

## Pitfalls

- **Contradictions:** "if two instructions contradict each other, Claude may pick one arbitrarily"
  - e.g. "MVP" in the request, "full product" in the planning rule
- **Agent-chosen stack:** may fit the spec but not your access or skills; decide it yourself
- **Bloated rules get ignored:** start small; add a rule only when a mistake repeats
- **Copied templates:** a starting point, not a substitute for understanding

<!-- Review generated plans before building from them, and prune rule files regularly. -->

---

## Parallel subagents

- Each subagent has **its own context window**, tools and permissions
- Keeps exploration out of the main conversation
- Parallel works best when tasks **don't depend on each other**
  - Good: UI variations, researching separate modules
  - Poor: a staged build where each step needs the last
- Dependent steps: run subagents **in sequence**

<!-- From Claude Code's subagents docs. Topics 23 and 27 go further. -->

---
zoom: 0.9
---

## What lasts vs. what will date

**Lasts**
- Context is something you design
- Small always-on core; load the rest when relevant
- Project knowledge in files, not chat history
- Contradictions → unpredictable behaviour
- Parallelism only for independent work

**Will date**
- Rule mechanisms and line limits (as of 2026-10)
- Subagent features and limits
- The label "context engineering" itself

<!-- The practice is older than the term and will likely outlast it. -->

---

## Try it yourself

1. One-page requirements doc with a chosen stack and out-of-scope list
2. Split project knowledge into four short files
3. Always-on rule under 10 lines mapping situations → files
4. Fresh session: "continue from the plan"; note what's missing
5. Hunt for contradictions in your rules; fix and regenerate

<!-- Exercise 4 shows whether your files really hold the project memory. -->

---
layout: end
---

# Key takeaways

- **Curate, don't dump**: the smallest set of high-signal tokens wins
- **Files are memory**: plan + focused docs + a short map of when to read each
- **Review and prune**: contradictions and bloat quietly break agents

anthropic.com/engineering/effective-context-engineering-for-ai-agents\
code.claude.com/docs/en/memory\
cursor.com/docs/context/rules
