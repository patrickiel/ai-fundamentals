# The Harness Matters as Much as the Model: Setting Up Coding Agents for Large Codebases

> **Core idea in one sentence:** On large codebases, coding agents succeed or fail mainly on whether they find the right context, so results depend as much on the harness around the model (how it searches, which instructions load and when, which rules are enforced, which tools and helpers it has) as on the model itself.

## TL;DR

- Agents find code either by **retrieval** from a pre-built embedding index (RAG) or by **agentic search**: exploring the live repository with tools like `glob` and `grep`. Claude Code uses agentic search because indexes go stale [1][2][3].
- Agentic search "works best when Claude has enough starting context to know where to look" [1]. Much of the harness exists to provide that starting context.
- Anthropic's guidance for large codebases: the harness "determines how Claude Code performs more than the model alone" [1].
- The main pieces are **CLAUDE.md files, hooks, skills, plugins, LSP code intelligence, MCP servers and subagents**. Each controls what enters context, and when [1][4].
- Keep always-loaded instructions short (Claude Code's docs target **under 200 lines** per CLAUDE.md), scope the rest by directory or task, and **enforce** critical rules with hooks, since instructions are context, not enforcement [4][5][6].
- Review the whole setup **every three to six months**, because instructions written for one model can hold back the next [1].

## How agents find code: retrieval vs. agentic search

### Retrieval (RAG)

A retrieval system embeds the codebase into a vector index ahead of time. At query time, it finds the chunks most similar to the request and adds them to the model's context (topic 16 covers the mechanics). Anthropic's context-engineering guide calls this "embedding-based pre-inference time retrieval" and notes that many AI applications use it [2].

For code that changes daily, the index is a copy that drifts. Anthropic's large-codebase guide puts it concretely: a stale index can return "a function the team renamed two weeks ago" [1]. And in a very large codebase, the most *similar* chunk is often not the *right* one.

### Agentic search

Claude Code instead "navigates a codebase the way a software engineer would": it lists and reads files, runs `grep`, and follows references. It runs locally and needs no index [1]. The context-engineering guide describes this as a **hybrid**: "CLAUDE.md files are naively dropped into context up front, while primitives like glob and grep allow it to navigate its environment and retrieve files just-in-time", which bypasses "the issues of stale indexing and complex syntax trees" [2].

Boris Cherny, who created Claude Code, has said the team tried local vector databases and recursive model-based indexing before settling on this approach. Both alternatives brought stale indexes and permission complexity. His summary: "Plain glob and grep, driven by the model, beat everything" [3].

| | Retrieval (RAG) | Agentic search |
|---|---|---|
| Source of truth | A pre-built index (can go stale) | The live file system |
| How it selects | Semantic similarity | Iterative listing, searching, reading |
| Context quality | May include near-miss chunks | Narrowed to what the agent has confirmed |
| Weak spot | Fast-changing code, permissions | Needs signposts to know where to look |

Agentic search has a cost too. A vague search across a huge codebase can exhaust the context window [1]. So the rest of this topic is about **signposts**: giving the search a good starting point and keeping what it loads relevant. Retrieval isn't obsolete. Some advanced teams expose structured code search to Claude as an MCP tool [1].

## Why the harness matters as much as the model

The **harness** is everything around the model that shapes its behaviour: instruction files, tools, hooks, skills, subagents and configuration. Anthropic's guide states that this ecosystem "determines how Claude Code performs more than the model alone" [1]. The context-engineering guide gives the underlying reason: context is "a finite resource with diminishing marginal returns", and good context engineering means "finding the smallest possible set of high-signal tokens" for the task [2] (topics 03 and 04).

Built-in defaults suit codebases in general. Large codebases need a harness tuned to *their* structure, conventions and tools. Order matters as well: "each layer builds on what came before" [1]. Claude Code's documentation recommends growing the setup in response to specific triggers rather than configuring everything up front. For example, if Claude gets a convention wrong twice, that convention belongs in CLAUDE.md [4].

## The pieces of a harness

Anthropic's guide summarises each component and the mistake teams most often make with it [1]:

| Component | When it loads | Best for | Common confusion |
|---|---|---|---|
| CLAUDE.md | Every session | Conventions and codebase knowledge | Putting reusable expertise here instead of in a skill |
| Hooks | On lifecycle events | Consistent automated behaviour, capturing learnings | Using prompts for steps that should run automatically |
| Skills | On demand | Reusable expertise | Loading everything into CLAUDE.md |
| Plugins | Once configured | Distributing a working setup | Letting good setups stay tribal |
| LSP | Once configured | Symbol navigation, errors in typed languages | Assuming it works automatically |
| MCP servers | Once configured | Internal tools Claude can't otherwise reach | Building MCP before the basics work |
| Subagents | When invoked | Separating exploration from editing; parallel work | Exploring and editing in one session |

### 1. CLAUDE.md: short, layered, maintained

CLAUDE.md files load at the start of every session [5]. Anthropic's advice for large repositories is that "the root file should be pointers and critical gotchas only", with local conventions in subdirectory files [1].

- **Size:** as of 2026-10, Claude Code's docs say to "target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence." Splitting a file with `@imports` helps organisation but saves nothing, because imports also load at launch [5].
- **Layering:** files above the working directory load at launch. Files in subdirectories load on demand, when Claude reads or edits files there [5]. Anthropic recommends starting sessions in the relevant subdirectory, since Claude walks up the tree and loads every CLAUDE.md on the way [1].
- **Path-scoped rules:** `.claude/rules/*.md` files with `paths:` frontmatter load only when Claude works on matching files [5].
- **Limits:** CLAUDE.md is "context, not enforced configuration". To block an action regardless of what the model decides, use a hook [5].

### 2. Hooks: enforce, don't ask

Hooks are commands that Claude Code runs at lifecycle events, giving "deterministic control: certain actions always happen rather than relying on the LLM to choose to run them" [6]. Claude Code's docs put it bluntly: "An instruction like 'never edit `.env`' in CLAUDE.md or a skill is a request, not a guarantee" [4].

- **`PreToolUse`** runs before a tool call. A script that exits with **code 2** blocks the action, and its stderr goes back to Claude as feedback. The docs' example blocks edits to `.env`, lock files and `.git/` [6].
- **`SessionStart`** can add context, for example re-injecting critical notes after compaction [6].
- **`PostToolUse`** can run formatters or linters after each edit [6].
- **`Stop`** runs when Claude finishes responding. Anthropic suggests a stop hook that will "reflect on what happened during a session and propose CLAUDE.md updates while the context is fresh" [1]. Stop hooks fire after every response, not only at task completion, and need a guard (`stop_hook_active`) to avoid loops [6]. Review the proposed updates before accepting them.

### 3. Skills: expertise on demand

Skills load their full content only when relevant (topic 11). Anthropic recommends moving specialised workflows out of CLAUDE.md into skills and **scoping skills to paths**: a payments team's deployment skill can be bound to its directory so it "never auto-loads" elsewhere in the monorepo [1]. Claude Code supports this with the `paths` frontmatter field [4].

### 4. Plugins: share the harness

A plugin bundles skills, hooks, subagents and MCP servers into one installable unit [4]. For teams, this stops good setups from staying "tribal". Updates can be distributed through managed marketplaces [1]. Plugins run code with your privileges, so vet third-party ones (topic 12).

### 5. LSP: navigate symbols, not strings

A language server gives Claude IDE-style navigation: go to definition, find references, type errors after edits. "Without it, Claude pattern-matches on text and can land on the wrong symbol". A `grep` for a common function name can return thousands of matches, while "LSP returns only the references that point to the same symbol" [1]. In Claude Code, the LSP tool stays inactive until you install a code intelligence plugin for your language (as of 2026-10) [4]. One company in Anthropic's guide deployed LSP organisation-wide *before* rollout so C and C++ navigation would be reliable [1].

### 6. MCP: reach internal systems

MCP servers give Claude access to internal tools, data and APIs it can't otherwise reach (topics 09 and 10). Anthropic lists "building MCP connections before the basics work" as a common mistake [1]. Get instructions, hooks and navigation right first.

### 7. Subagents: isolate exploration

A subagent "runs in its own context window" and "returns only the summary" [7]. The context-engineering guide notes that such summaries are often 1,000–2,000 tokens, however much the subagent read [2] (topic 05).

- Claude Code's built-in **Explore** agent is read-only and built for code search. As of 2026-10, a project or user subagent named `Explore` overrides it, so you can write one that already knows your repository's layout [7].
- A documented pattern is a read-only subagent that maps a subsystem and writes its findings to a file, which the main agent then uses for editing [1].
- Independent research can run in parallel. Many detailed results still consume the main context, so ask for concise returns [7].

## Large-codebase practices

From Anthropic's guide [1]:

- **Scope test and lint commands per subdirectory.** Running the full suite after changing one service "causes timeouts and wastes context". This is harder in compiled monorepos with deep cross-directory dependencies.
- **Exclude what the agent shouldn't touch.** Commit `permissions.deny` rules in `.claude/settings.json` for generated files, build artifacts and third-party code, so the whole team gets them.
- **Write a codebase map** when the directory structure doesn't explain the code: "a lightweight markdown file at the repo root listing each top-level folder with a one-line description". For hundreds of top-level folders, layer the maps.
- **Use LSP for symbol search** in typed languages.
- **Review the setup** "every three to six months", and after major model releases. Instructions written for one model's weaknesses can constrain a later model. One example is a rule forcing single-file refactors that would block coordinated cross-file edits. Hooks can become redundant too: one team's hook enforcing `p4 edit` for Perforce became unnecessary once Claude Code added native Perforce support.
- **Assign an owner.** The fastest rollouts had someone responsible for the configuration, at minimum one directly responsible individual. Regulated organisations started with approved skills, required code review and limited access.

The guide also names its limits. Hierarchical CLAUDE.md can break down at hundreds of thousands of folders, or on version control other than git [1].

## What lasts vs. what will date

- **Durable principles:**
  - Agent performance on large codebases is mostly a **context-finding** problem.
  - Search the **live source of truth**, and give the search signposts (maps, scoped instructions, symbol navigation).
  - **Layer context:** a short always-on core, scoped files per area, on-demand skills, isolated subagents.
  - **Enforce** critical rules mechanically. Instructions are requests.
  - **Package and share** the harness, and give it an owner.
  - **Re-evaluate** the harness as models improve, and remove what no longer helps.
- **Time-sensitive details (as of 2026-10):**
  - Claude Code file names, hook event names, exit-code conventions, the `paths` field and plugin mechanics.
  - The "under 200 lines" guidance and subagent override rules.
  - Whether a given tool uses an index, agentic search or both. This varies by product and changes.

## Try it yourself

1. **Audit your instruction file.** Count its lines. Move directory-specific content into subdirectory files or path-scoped rules, and procedures into skills. Compare behaviour on the same task before and after.
2. **Turn one instruction into a hook.** Pick a rule the agent sometimes breaks (for example, "never edit generated files") and enforce it with a `PreToolUse` hook that exits with code 2 and a clear message.
3. **Add a learning loop.** Configure a stop hook that proposes CLAUDE.md additions based on the session. Review each proposal before accepting it.
4. **Compare text search with LSP.** In a project where a common function name appears in several modules, ask the agent to find one specific definition with and without a code intelligence plugin. Count tool calls and wrong hits.
5. **Write a codebase map** for a repository you don't know well, and count the search commands the agent needs to locate three features with and without it.

## Key terms

- **Harness**: everything around the model that shapes its behaviour: instructions, tools, hooks, skills, subagents and configuration.
- **RAG (retrieval-augmented generation)**: retrieving chunks from a pre-built embedding index and adding them to the model's context.
- **Agentic search**: the agent exploring the live file system (`glob`, `grep`, reading files) to find relevant code itself.
- **Just-in-time context**: keeping lightweight references (paths, queries) and loading content only when needed.
- **CLAUDE.md**: a Markdown file of project conventions loaded into every session. Subdirectory files load on demand.
- **Hook**: a command that runs automatically at a lifecycle event (before a tool call, at session start, at stop) to enforce behaviour or add context.
- **LSP (Language Server Protocol)**: a standard that provides symbol-level code intelligence such as go-to-definition and find-references.
- **Subagent**: a delegated agent with its own context window that returns only a summary.
- **Codebase map**: a short file that describes each top-level folder so the agent knows where to look.

## Sources

1. [How Claude Code works in large codebases: Best practices and where to start](https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start) (Anthropic, Claude blog, 2026-05-14). Agentic search vs. RAG, "the harness matters as much as the model", the component table, layered CLAUDE.md, stop-hook learning, path-scoped skills, LSP, subagent mapping, codebase maps, exclusions, review cadence and ownership.
2. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Prithvi Rajasekaran et al., Anthropic Engineering, 2025-09-29). Context as a finite resource, just-in-time vs. pre-inference retrieval, Claude Code's hybrid strategy, and condensed subagent summaries.
3. [Building Claude Code with Boris Cherny](https://newsletter.pragmaticengineer.com/p/building-claude-code-with-boris-cherny) (Gergely Orosz, The Pragmatic Engineer, 2026-03-04). The Claude Code creator's account of trying vector databases and indexing before choosing model-driven glob and grep.
4. Claude Code documentation: [Extend Claude Code](https://code.claude.com/docs/en/features-overview) and [Skills](https://code.claude.com/docs/en/skills) (Anthropic, accessed 2026-10-14). How CLAUDE.md, skills, hooks, MCP, subagents, code intelligence and plugins load and cost context, "put guardrails in hooks", growing a setup by trigger, and the `paths` field.
5. [How Claude remembers your project](https://code.claude.com/docs/en/memory) (Claude Code documentation, Anthropic, accessed 2026-10-14). CLAUDE.md size guidance, launch vs. on-demand loading, path-scoped rules, imports, and "context, not enforced configuration".
6. [Automate actions with hooks](https://code.claude.com/docs/en/hooks-guide) (Claude Code documentation, Anthropic, accessed 2026-10-14). Hook events, exit code 2 blocking with stderr feedback, protected-file and SessionStart examples, and Stop-hook behaviour.
7. [Subagents](https://code.claude.com/docs/en/sub-agents) (Claude Code documentation, Anthropic, accessed 2026-10-14). Isolated context, built-in Explore, custom definitions and overrides, and parallel use.
