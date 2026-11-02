---
theme: ../theme
chapter: '21'
header: 'AI Fundamentals · 21 · Harness over model'
layout: cover
---

# The Harness Matters as Much as the Model

**On large codebases, agents succeed or fail on finding the right context, and the harness decides that**

---

## How agents find code

| | Retrieval (RAG) | Agentic search |
|---|---|---|
| Source | Pre-built embedding index | **Live** file system |
| Method | Semantic similarity | `glob`, `grep`, read, follow refs |
| Risk | **Stale**: "a function the team renamed two weeks ago" | Needs a starting point |

Claude Code: CLAUDE.md up front + **just-in-time** glob/grep

<!-- Anthropic's context-engineering guide calls this hybrid strategy a way of bypassing stale indexing and complex syntax trees. -->

---

## Why Claude Code dropped the index

- Team tried **local vector databases** and **recursive indexing**
- Problems: **stale indexes**, **permission complexity**
- "Plain glob and grep, driven by the model, beat everything" (Boris Cherny)
- Trade-off: works best with **enough starting context to know where to look**

<!-- Retrieval isn't obsolete: some teams expose structured search to Claude as an MCP tool. -->

---

## Harness over model

- Harness = instructions, tools, hooks, skills, subagents, config
- Anthropic: it "determines how Claude Code performs **more than the model alone**"
- Context is finite: aim for the **smallest set of high-signal tokens**
- Build layers **in order**; add each one when a real trigger appears

<!-- Example trigger from the docs: if Claude gets a convention wrong twice, add it to CLAUDE.md. -->

---

## The pieces and their common mistakes

| Piece | Loads | Common confusion |
|---|---|---|
| CLAUDE.md | Every session | Reusable expertise put here, not in a skill |
| Hooks | On events | Prompts for steps that should be automatic |
| Skills | On demand | Everything loaded into CLAUDE.md |
| Plugins | Once set up | Good setups stay tribal |
| LSP | Once set up | Assuming it works automatically |
| MCP | Once set up | Building it before the basics work |
| Subagents | When invoked | Exploring and editing in one session |

<!-- Table adapted from Anthropic's large-codebase guide. -->

---

## 1. CLAUDE.md: short and layered

- Docs (as of 2026-10): **under 200 lines** per file
- Root = **pointers and critical gotchas**
- Subdirectory files load **on demand**; start sessions in the subdirectory
- `.claude/rules/` with `paths:` for area-specific rules
- `@imports` organise but **don't save context**

> CLAUDE.md is **context, not enforced configuration**

<!-- Longer files consume more context and reduce adherence. -->

---

## 2. Hooks: enforce, don't ask

| Event | Use |
|---|---|
| `PreToolUse` | **Exit 2** blocks; stderr becomes Claude's feedback |
| `PostToolUse` | Format, lint after edits |
| `SessionStart` | Add context (e.g. after compaction) |
| `Stop` | Propose **CLAUDE.md updates** while context is fresh |

> "never edit `.env`" in CLAUDE.md is a **request**; a hook is **enforcement**

<!-- Stop fires after every response; guard with stop_hook_active. Review proposed CLAUDE.md edits before accepting. -->

---

## 3. Skills · 4. Plugins

**Skills**
- Load **on demand**; move procedures out of CLAUDE.md
- **Path-scope** them: a payments deploy skill never loads elsewhere

**Plugins**
- Bundle skills + hooks + subagents + MCP servers
- Share **one harness** across the team; vet third-party ones (topic 12)

<!-- Claude Code's `paths` frontmatter field implements path scoping. -->

---

## 5. LSP: symbols, not strings

- Without it, Claude "pattern-matches on text and can **land on the wrong symbol**"
- `grep` on a common name: thousands of hits; LSP: **same symbol only**
- Claude Code: inactive until you install a **code intelligence plugin**
- One company deployed LSP org-wide **before** rollout (C/C++)

<!-- Code intelligence also reports type errors after edits, which can reduce file reads overall. -->

---

## 6. MCP · 7. Subagents

**MCP**
- Internal docs, data, APIs, structured search
- Only **after** the basics work

**Subagents**
- Own context window; return **only a summary** (often 1–2k tokens)
- Custom `Explore` **overrides** the built-in (as of 2026-10)
- Read-only mapping subagent writes findings to a file

<!-- Parallel subagents speed up independent research, but many detailed returns still fill the main context. -->

---

## Large-codebase practices

- **Tests and lint per subdirectory**: no timeouts, less context
- **`permissions.deny`** in committed settings for generated and third-party code
- **Codebase map**: one line per top-level folder; layer it if huge
- **Review every 3–6 months** and after major model releases
- Give the configuration an **owner**

<!-- Example of an outdated rule: forcing single-file refactors blocks coordinated cross-file edits by newer models. -->

---

## What lasts vs. what will date

**Lasts**
- Large-codebase work is a **context-finding** problem
- Search live sources; give the search **signposts**
- Layer context: core → scoped → on demand → isolated
- Enforce with hooks; share and own the harness; re-review

**Will date (as of 2026-10)**
- File names, hook events, exit codes, `paths`, plugin mechanics
- "Under 200 lines", subagent override rules

<!-- The mechanics are Claude Code's; the layering and enforcement ideas apply to any coding agent. -->

---

## Try it yourself

1. Audit CLAUDE.md: move area detail to rules, procedures to skills
2. Turn one often-broken instruction into a **`PreToolUse` hook**
3. Add a **stop hook** that proposes CLAUDE.md updates; review them
4. Compare symbol lookup **with vs. without LSP**
5. Write a **codebase map** and count search calls saved

<!-- Exercise 2 shows the difference between asking and enforcing most clearly. -->

---
layout: end
---

# Key takeaways

- Agents fail at scale when they **can't find the right context**
- Results depend on **model + harness**: build one for your codebase
- **Layer** context, **enforce** rules with hooks, **re-review** as models change

claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start\
anthropic.com/engineering/effective-context-engineering-for-ai-agents\
code.claude.com/docs/en/features-overview
