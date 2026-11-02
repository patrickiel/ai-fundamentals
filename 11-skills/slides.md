---
theme: ../theme
chapter: '11'
header: 'AI Fundamentals · 11 · Agent skills'
layout: cover
---

# Agent Skills

**A folder of know-how that loads in stages, so only a name and description sit in context until the task needs the rest**

---

## The problem

- Many agent tasks are **repetitive**: same format, checklist, steps
- Re-explaining each time is slow and **inconsistent**
- Putting every procedure in the prompt hits the **context budget**
- Many MCP clients load **every tool definition up front**
- Need: specialist behaviour **without** filling context

<!-- Anthropic describes skills as an onboarding guide for a new team member. The MCP point comes from Anthropic's "Code execution with MCP" post (Nov 2025); Claude Code now defers full tool schemas. -->

---

## Anatomy of a skill

```
pdf-processing/
├── SKILL.md       ← frontmatter + instructions (required)
├── scripts/       ← executable code
├── references/    ← docs read on demand
└── assets/        ← templates, schemas, data
```

- `name`: ≤ 64 chars, lowercase + hyphens, matches folder
- `description`: ≤ 1,024 chars, **what it does + when to use it**

<!-- Layout and limits from the open Agent Skills spec at agentskills.io. Anthropic's surfaces also reserve the words "anthropic" and "claude". -->

---

## Three kinds of content

| Part | Strength |
|---|---|
| Instructions (`SKILL.md`, references) | **Flexible guidance** |
| Scripts | **Reliability**: deterministic, only output enters context |
| Resources (schemas, templates) | **Factual lookup** |

> Example: the PDF skill ships a form-field extraction script; `forms.md` loads only when a form must be filled

<!-- Anthropic's engineering post: some operations are better done by code than by token generation. -->

---

## Progressive disclosure

| Level | When | Cost |
|---|---|---|
| 1. Metadata | Always, at startup | **~100 tokens** per skill |
| 2. `SKILL.md` body | When triggered | **< 5k tokens** recommended |
| 3+. Files and scripts | Only when needed | **0** until read; scripts: output only |

- Keep `SKILL.md` **< 500 lines**; link references **one level deep**

<!-- Figures from the Claude Platform docs and the spec. Nested reference chains risk partial reads. -->

---

## The description is the router

- Agent chooses among possibly **100+ skills** by description
- Say **what** and **when**, with the words users actually use
- Write in **third person**: "Processes Excel files…"
- Claude Code truncates the listing text at **1,536 chars**: lead with the use case

<!-- Vague descriptions like "Helps with documents" are the most common reason a skill never triggers. -->

---

## Writing skills that work

- **Concise**: "the context window is a public good"
- **Degrees of freedom**: exact script for fragile steps, heuristics for judgement
- **Evaluate first**: baseline without the skill, 3 scenarios, then minimal instructions
- **Two sessions**: one refines the skill, a fresh one uses it
- Test on **every model** you'll use

<!-- All from Anthropic's skill authoring best practices. The narrow-bridge vs open-field analogy explains degrees of freedom. -->

---

## Skills vs. MCP

| | MCP | Skills |
|---|---|---|
| Provides | **Tools + data access** | **Knowledge + workflows** |
| Example | Query DB, post to Slack | Schema notes, deploy checklist |
| Context | Depends on client | Metadata until triggered |

**Combine them**: MCP for the connection, a skill for how to use it well

<!-- Claude Code docs: "MCP connects Claude to external services. Skills extend what Claude knows, including how to use those services effectively." Use ServerName:tool_name inside skills. -->

---

## Where skills run (as of 2026-10)

- **Claude Code**: `~/.claude/skills/`, `.claude/skills/`, plugins; `/skill-name` or automatic
- **claude.ai**: upload zip on paid plans; private per user
- **Claude API**: code execution container; `/v1/skills`
- Custom skills **don't sync** between surfaces

<!-- Custom slash commands have been merged into skills in Claude Code. -->

---

## Network access depends on the surface

| Surface | Network |
|---|---|
| Claude API | **None**, no package installs |
| claude.ai | Full, partial or none (settings) |
| Claude Code | **Full**, like any local program |

> Check the runtime before designing a skill around live data. Use MCP for authenticated external access.

<!-- "Skills can't use the network" is only true on the API. -->

---

## An open format and examples

- Published as an **open standard** (Dec 2025): agentskills.io
- `anthropics/skills`: skill-creator, mcp-builder, algorithmic-art, canvas-design, docx/pdf/pptx/xlsx
- Document skills: **source-available**, not open source
- Examples are for **demonstration**: test before relying on them

<!-- Install in Claude Code with /plugin marketplace add anthropics/skills. -->

---

## Creating skills with Claude

- **skill-creator** interviews you, builds the folder and `SKILL.md`
- Claude Code plugin (as of 2026-10): adds **evals**, with/without comparisons, trigger tuning
- Or just ask: "turn what we just did into a skill"
- Then **review every file** and test the trigger

<!-- Security: a skill can direct tools or code in ways that don't match its stated purpose. Topic 12 covers vetting. -->

---

## What lasts vs. what will date

**Lasts**
- Progressive disclosure: short index, detail on demand
- Access (MCP) ≠ know-how (skills)
- Scripts for exact steps; description as router
- Skills are files: review, test, version

**Will date**
- Paths, frontmatter extensions, token/line figures
- Per-surface network rules, plans, skill-creator packaging

<!-- The layering principle applies to any agent, whatever happens to the format. -->

---

## Try it yourself

1. List 5 repeated tasks; write a **what + when** description for each
2. Build `SKILL.md` + one script by hand; check the spec's naming rules
3. 10 prompts (5 trigger, 5 don't); tune the description
4. Add a reference file; watch **which files** the agent opens
5. Generate a skill from a finished task; trim what the model doesn't need

<!-- Exercise 3 teaches the most: routing depends almost entirely on the description. -->

---
layout: end
---

# Key takeaways

- A skill = **instructions + scripts + resources** in a folder
- **Progressive disclosure** keeps many skills cheap until used
- Skills package **know-how**; MCP provides **access**

anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills\
platform.claude.com/docs/en/agents-and-tools/agent-skills/overview\
agentskills.io/specification
