# Agent Skills: Packaging Know-How with Progressive Disclosure

> **Core idea in one sentence:** A skill is a folder of instructions, reference files and scripts that an agent loads in stages, so only a short name and description sit in context until a task needs the rest, which lets a general agent take on specialist procedures without filling its context window.

## TL;DR

- A **skill** is a directory containing a `SKILL.md` file: YAML frontmatter with a required `name` and `description`, followed by Markdown instructions. It can also bundle scripts, reference documents and assets [1][6].
- Skills use **progressive disclosure**. About 100 tokens of metadata per skill load at startup. The `SKILL.md` body loads only when the skill is triggered. Bundled files load only when read, and scripts run without their code entering context [3][6].
- The **description is the trigger**. The agent decides whether to use a skill by matching the request against it, so it must say what the skill does *and* when to use it [3][4].
- **Skills vs. MCP:** MCP gives the agent tools and data access for external systems. Skills give it knowledge and workflows, including how to use those tools well. The two combine [5].
- Network access **depends on where the skill runs**. As of 2026-10: none on the Claude API, configurable on claude.ai, and the same as any local program in Claude Code [3].
- Skills contain instructions and code, so installing one is like installing software. Topic 12 covers how to vet them [1][3].

## The problem skills solve

Much agent work is repetitive: the same report format, the same review checklist, the same extraction steps. Re-explaining the procedure in every conversation is slow and gives inconsistent results. Anthropic's documentation describes skills as a way to package "workflows, context, and best practices that turn a general-purpose agent into a specialist", organized "like an onboarding guide you'd create for a new team member" [3].

The obvious alternative, putting every procedure into the system prompt or a large instruction file, runs into the context limit. Topic 03 explained why context is a finite budget. Tool integrations show the same problem: Anthropic's engineers noted in late 2025 that "most MCP clients load all tool definitions upfront directly into context", which at scale can mean hundreds of thousands of tokens before the agent reads the request [8]. (As of 2026-10, Claude Code defers full MCP tool schemas until they are needed [5].) Skills were designed from the start to load in stages.

## Anatomy of a skill

The open Agent Skills specification defines the layout [6]:

```
pdf-processing/
├── SKILL.md       # required: frontmatter + instructions
├── scripts/       # optional: executable code
├── references/    # optional: docs read on demand
└── assets/        # optional: templates, schemas, data
```

A minimal `SKILL.md` (adapted from the spec's examples):

```markdown
---
name: pdf-processing
description: Extracts text and tables from PDF files, fills PDF forms, and merges PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
---

# PDF processing
1. For plain text, run `scripts/extract_text.py <file>`.
2. To fill a form, read `references/FORMS.md` first.
```

**Frontmatter rules** [3][6]:

- `name`: up to 64 characters, lowercase letters, digits and hyphens, matching the folder name. Anthropic's surfaces also reject the reserved words "anthropic" and "claude".
- `description`: up to 1,024 characters, saying what the skill does and when to use it.
- Optional in the spec: `license`, `compatibility` (environment needs such as network access), `metadata`, and an experimental `allowed-tools`.
- Individual products add their own fields. Claude Code, for example, supports `disable-model-invocation` (only the user can trigger the skill), `paths` (glob patterns that limit when it activates) and `context: fork` (run it in a subagent) [5].

The three kinds of bundled content have different strengths: "instructions for flexible guidance, code for reliability, resources for factual lookup" [3].

**Why scripts matter.** Anthropic's engineering post argues that some operations are better done by code than by token generation, because code is deterministic [1]. A bundled script also saves context: the agent runs it and "only its output (such as 'Validation passed' or a specific error message) consumes tokens" [3]. The post's example is the PDF skill behind Claude's document features. It ships a pre-written Python script to extract form fields, and keeps form-filling instructions in a separate `forms.md` that loads only when a form actually needs filling [1].

## Progressive disclosure

This is the central mechanism, and the idea most likely to outlast the specific product:

| Level | When loaded | Approximate cost | Content |
|---|---|---|---|
| 1. Metadata | Always, at startup | ~100 tokens per skill | `name` + `description` |
| 2. Instructions | When the skill is triggered | Under 5k tokens recommended | `SKILL.md` body |
| 3+. Resources | Only when needed | Nothing until read | Reference files (read into context), scripts (only output enters) |

*Source: Claude Platform docs and the Agent Skills spec [3][6].*

Because unused files cost nothing, the post's authors note that the amount of context a skill can bundle is practically unlimited [1]. Two layout rules from the spec and authoring guide keep the stages working: keep `SKILL.md` under 500 lines, and link reference files **one level deep** from it, because the agent may only preview files reached through a chain of references [4][6].

The trade-off is that level 1 must be good enough to route on. Anthropic's authoring guide says the description is what the agent uses "to choose the right Skill from potentially 100+ available Skills" [4]. In Claude Code, the description (plus an optional `when_to_use`) is truncated at 1,536 characters in the skill listing, so the key use case should come first (as of 2026-10) [5].

## Writing a skill that works

Anthropic's authoring guide [4] gives practical rules:

- **Be concise.** "The context window is a public good." Assume the model already knows general facts, and only add what it lacks.
- **Match the degree of freedom to the risk.** For fragile, exact sequences (a database migration), give a specific script and say not to change it. For judgement tasks (a code review), give heuristics. The guide's analogy is a narrow bridge with cliffs versus an open field.
- **Write the description in the third person**, with the specific terms a user would say ("Processes Excel files and generates reports", not "I can help you...").
- **Evaluate first.** Run the agent on representative tasks without the skill, record where it fails, write three test scenarios, then write just enough instructions to pass them.
- **Iterate with two instances.** Use one Claude session to draft and refine the skill and a fresh one to use it on real tasks, then feed observed failures back.
- **Test on every model you plan to use**, since a skill that suits a large model may be too terse for a small one.
- **Avoid time-sensitive statements** inside the skill, or isolate them in an "old patterns" section.

## Skills vs. MCP

Both extend an agent, but they answer different needs. Claude Code's documentation puts it this way: "MCP connects Claude to external services. Skills extend what Claude knows, including how to use those services effectively" [5].

| | MCP | Skills |
|---|---|---|
| What it is | A protocol for connecting to external systems | Instructions, knowledge, scripts and resources in a folder |
| Provides | Tools and data access | Procedures, conventions, reference material |
| Typical use | Query a database, post to Slack, drive a browser | A review checklist, a deploy workflow, an API style guide |
| Context cost | Depends on the client (see above) | Metadata only until triggered |

They combine well. One documented pattern is an MCP server for the database plus a skill holding the schema and query patterns [5]. When a skill refers to MCP tools, the authoring guide recommends fully qualified names (`ServerName:tool_name`) so the agent can find the tool when several servers are connected [4]. Topics 09 and 10 cover MCP itself.

## Where skills run (as of 2026-10)

The format is shared, but each product handles skills separately, and custom skills **do not sync between surfaces** [3]:

- **Claude Code:** skills live in `~/.claude/skills/` (personal), `.claude/skills/` (project, committed for the team) or inside a plugin. Claude loads them when relevant, or you invoke one as `/skill-name`. Custom slash commands have been merged into skills. Edits are picked up within a running session [5].
- **claude.ai:** pre-built document skills are on by default. Custom skills are uploaded as zip files in settings on paid plans and are private to each user [3].
- **Claude API:** skills run inside the code execution tool's container. You reference pre-built skills (`pptx`, `xlsx`, `docx`, `pdf`) or upload your own through `/v1/skills` [3].

**Network access differs by surface** [3]:

| Surface | Network access for skills |
|---|---|
| Claude API | None, and no runtime package installation |
| claude.ai | Full, partial or none, depending on user and admin settings |
| Claude Code | Full: "the same network access as any other program on the user's computer" |

A common misconception is that skills cannot call networks at all. That is true only on the API. The durable lesson is narrower: check the runtime before designing a skill that depends on live data, and use MCP when you need authenticated access to an external system.

**An open format.** In December 2025 Anthropic published Agent Skills as an open standard for cross-platform portability [1], with the specification at agentskills.io [6]. Anthropic's public `anthropics/skills` repository holds example skills, including `skill-creator`, `mcp-builder`, `algorithmic-art`, `canvas-design` and the document skills. Many are Apache 2.0, while the document skills are source-available rather than open source. The repository states that the examples are for demonstration and education and should be tested before critical use [7].

## Creating skills with Claude

Anthropic's launch post describes a `skill-creator` skill that asks about your workflow, generates the folder structure, formats `SKILL.md` and bundles resources [2]. As of 2026-10, Claude Code offers it as a plugin (`skill-creator@claude-plugins-official`) that can also generate test cases, compare runs with and without the skill, and tune the description's trigger accuracy [5]. The authoring guide adds that no special tool is required: Claude understands the format, so you can ask it to "create a Skill that captures this pattern we just used" after finishing a task together [4].

Generated skills still need review. Read every file it produced, cut explanations the model doesn't need, and test the trigger.

## Security in brief

A skill can contain instructions and executable code. Anthropic warns that a malicious skill "can direct Claude to invoke tools or execute code in ways that don't match the Skill's stated purpose", and recommends using skills only from trusted sources and auditing everything else, including scripts, dependencies and any instructions to contact external networks [1][3]. Topic 12 treats this in depth.

## What lasts vs. what will date

- **Durable principles:**
  - **Progressive disclosure:** keep a short index always in context and load detail on demand. It applies to any agent design.
  - Separate **access** (connecting to systems) from **know-how** (doing a task well).
  - Put exact, fragile steps in **scripts**, and leave judgement to the model.
  - The **description is the router**. Write it for the requests that should trigger the skill.
  - Skills are plain files: version, review, test and improve them like code.
  - Treat third-party skills as software you are installing.
- **Time-sensitive details (as of 2026-10):**
  - Folder locations, frontmatter fields and product-specific extensions such as `paths` and `context: fork`.
  - Token and length guidance (~100 tokens, under 5k tokens, 500 lines, 1,536-character listing limit).
  - Per-surface network rules and plan availability.
  - How skill-creator is distributed, and which example skills the repository contains.

## Try it yourself

1. **Inventory your repeats.** List five tasks you've explained to an agent more than twice. For each, write a third-person description that says what it does and when to use it.
2. **Build a minimal skill by hand.** Create a folder with `SKILL.md` (frontmatter plus five steps) and one script for the exact part. Check it against the spec's naming rules.
3. **Test the trigger.** Write ten prompts, five that should trigger the skill and five that shouldn't. Adjust the description until all ten route correctly.
4. **Watch the stages.** Add a reference file and a script, then observe which files the agent actually opens on different requests.
5. **Generate and review.** Ask the agent to turn a task you just completed together into a skill. Read every file it produced and remove anything the model didn't need.

## Key terms

- **Skill**: a folder with a `SKILL.md` file plus optional scripts, references and assets that gives an agent a specialised, repeatable capability.
- **`SKILL.md`**: the entry file. YAML frontmatter (`name`, `description`) followed by Markdown instructions.
- **Progressive disclosure**: loading information in stages (metadata, then instructions, then resources) so only what the task needs occupies context.
- **Reference file**: a document inside a skill, read only when the task needs it.
- **Bundled script**: code inside a skill that the agent runs. Only its output enters context.
- **MCP (Model Context Protocol)**: a protocol that connects an agent to external tools and data.
- **Skill creator**: a skill that interviews you and generates new skills.
- **Context window**: the text a model can attend to at once. Everything loaded into it has a cost.

## Sources

1. [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) (Barry Zhang, Keith Lazuka, Mahesh Murag, Anthropic Engineering, 2025-10-16, updated 2025-12-18). The design rationale, the three loading levels, scripts as deterministic tools, the PDF skill example, security advice, and the open-standard update.
2. [Introducing Agent Skills](https://www.claude.com/blog/skills) (Anthropic, 2025-10-16). The launch announcement: composable, portable, efficient and code-capable skills across apps, Claude Code and the API, plus the skill-creator skill.
3. [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) (Claude Platform documentation, accessed 2026-10-14). Loading levels with token costs, per-surface availability, frontmatter limits, runtime and network constraints, and security considerations.
4. [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) (Claude Platform documentation, accessed 2026-10-14). Conciseness, degrees of freedom, description writing, one-level references, evaluation-first development, and MCP tool naming.
5. Claude Code documentation: [Skills](https://code.claude.com/docs/en/skills) and [Extend Claude Code](https://code.claude.com/docs/en/features-overview) (Anthropic, accessed 2026-10-14). Skill locations, invocation control, Claude Code frontmatter fields, the skill-creator plugin, and the MCP-vs-skill comparison.
6. [Agent Skills Specification](https://agentskills.io/specification) (agentskills.io, accessed 2026-10-14). The open format: directory layout, field constraints, optional directories, and progressive-disclosure guidance.
7. [anthropics/skills](https://github.com/anthropics/skills) (Anthropic, GitHub, accessed 2026-10-14). Example skills, licensing of the document skills, the demonstration disclaimer, and Claude Code installation commands.
8. [Code execution with MCP: Building more efficient agents](https://www.anthropic.com/engineering/code-execution-with-mcp) (Adam Jones, Conor Kelly, Anthropic Engineering, 2025-11-04). Why loading every tool definition up front strains context, and how saved code plus a `SKILL.md` becomes a skill.
