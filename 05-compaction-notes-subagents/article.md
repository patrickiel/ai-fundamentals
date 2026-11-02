# Long-Running Agents: Compaction, Structured Notes, and Sub-Agents

> **Core idea in one sentence:** An agent's context window is a finite working space, so work that outlasts one window needs three techniques: compress the history (compaction), keep important state in files outside the window (structured notes and memory), and split large jobs across agents that each start with a clean window (sub-agents).

## TL;DR

- Anthropic's context-engineering guide names three techniques for long-horizon work: **compaction**, **structured note-taking** and **sub-agent architectures**. It also says bigger windows alone won't fix context pollution [1].
- **Compaction** summarises a nearly full context and continues from the summary. Tune it to keep everything relevant first, then trim. Clearing old tool results is the lightest form [1]. In Claude Code, compact yourself with instructions rather than relying on the automatic pass [5][8].
- **Memory files** (`CLAUDE.md`, Claude Code's auto memory) load at the start of each session so the agent doesn't start from zero [5].
- **Structured notes** are files the agent writes and reads back: a progress log, a feature list, a to-do list. They survive compaction and restarts. Anthropic's API offers a **memory tool** for this, and combined with context editing it improved an internal agentic-search eval by 39% [2][3].
- **Sub-agents** do focused work in clean windows and return condensed summaries. Anthropic's multi-agent research system beat a single agent by 90.2% on its internal eval, but used about 15 times the tokens of a chat [1][7].
- Pick by task shape: compaction for one long conversation, notes for iterative work with milestones, sub-agents for complex research that can run in parallel [1].

## Why agents lose track

A model has no memory between turns. Each turn it sees one block of tokens, the context window, and in an agent that block holds the system prompt, tool definitions, memory files, every file read and every tool result so far [5]. As it grows, quality drops (topic 04 covers this "context rot"). Anthropic describes the model's attention as a budget that each token draws on, and argues that context should be curated so the model sees "the smallest possible set of high-signal tokens" [1].

Some tasks run for hours and need far more tokens than any window holds. Anthropic's guide states that waiting for bigger windows won't solve this, because "context windows of all sizes will be subject to context pollution and information relevance concerns" [1]. Its answer is three techniques, which the rest of this topic covers.

## Technique 1: Compaction

### What it is

**Compaction** takes a conversation that is nearing the limit, summarises it, and starts a new context window from that summary. In Claude Code, the model keeps "architectural decisions, unresolved bugs, and implementation details" and drops redundant tool output [1].

The hard part is choosing what to keep. Anthropic's advice for anyone writing a compaction prompt: first maximise **recall**, so nothing important is lost, then improve **precision** by cutting what isn't needed [1].

### Automatic vs. manual

Claude Code compacts automatically as you approach the limit. You can also trigger it yourself [5]:

| | Automatic | Manual |
|---|---|---|
| **When** | Near the limit (threshold configurable with `/autocompact`) | When you run `/compact` |
| **What's kept** | What the model guesses matters | What you tell it, e.g. `/compact focus on the API changes` |
| **Risk** | Fires mid-task, when rot is worst | You have to remember to do it |

Anthropic's Claude Code team notes that automatic compaction happens when the model is least sharp and has to guess where the work is going, so they recommend compacting proactively with a hint about the next step [8]. You can also put standing rules in `CLAUDE.md`, for example "When compacting, always preserve the full list of modified files and any test commands" [5].

After compaction, Claude Code reloads the system prompt, project-root `CLAUDE.md`, auto memory and the current plan from disk, and re-reads up to five recently modified files. Instructions given only in conversation are summarised with everything else [5]. That is one reason to keep durable state in files (Technique 3).

### Lighter forms: clearing old tool results

Anthropic calls **tool result clearing** "one of the safest lightest touch forms of compaction": once a tool was called deep in the history, the agent rarely needs its raw output again [1]. On the Claude API this is available as **context editing** (beta as of 2026-10). The `clear_tool_uses_20250919` strategy replaces the oldest tool results with placeholders once the input passes a trigger. Defaults are 100,000 input tokens, keeping the 3 most recent tool uses, and you can exempt specific tools [4]. The API also offers server-side compaction, which summarises older turns for you (also beta as of 2026-10) [4].

When Anthropic launched context editing with Claude Sonnet 4.5 in September 2025, it reported that in a 100-turn web-search evaluation, context editing let agents finish workflows that would otherwise run out of context, while cutting token use by 84% [2].

## Technique 2: Persistent memory files

Compaction works within a session. Across sessions, the agent starts each one with a fresh window and no memory of the last [5]. Memory files close that gap.

Claude Code has two mechanisms, both loaded at the start of every session [5]:

| | `CLAUDE.md` | Auto memory |
|---|---|---|
| **Who writes it** | You | Claude |
| **Contents** | Instructions and rules: build commands, conventions, architecture notes | Learnings: corrections, preferences, patterns it noticed |
| **Loaded** | In full | First 200 lines or 25KB of `MEMORY.md`. Topic files are read only on demand |

The docs advise keeping each `CLAUDE.md` under about 200 lines, because "longer files consume more context and reduce adherence", and moving workflow-specific instructions into skills, which load only when used (topic 11) [5]. If you ask Claude to "remember" something, it goes into auto memory. Ask explicitly if you want it in `CLAUDE.md` [5].

## Technique 3: Structured note-taking

**Structured note-taking** means the agent regularly writes notes to files outside the context window and reads them back when it needs them [1]. Anthropic's examples range from a simple to-do list to Claude playing Pokémon, where the agent kept notes such as "for the last 1,234 steps I've been training my Pokémon in Route 1" across thousands of steps and context resets [1].

### A worked pattern for coding

Anthropic's article on harnesses for long-running agents gives a concrete setup for projects that span many sessions [6]:

- An **initializer** session creates the scaffolding: a feature list (over 200 features in their example, all marked failing), a progress file (`claude-progress.txt`), an `init.sh` that starts the app, and a first git commit.
- Each later **coding** session gets its bearings by reading the progress file and git log, picks the highest-priority unfinished feature, runs a basic end-to-end test, works on **one feature at a time**, and ends by committing and updating the progress notes.
- The feature list is **JSON, not Markdown**, because the model was less likely to change or overwrite JSON inappropriately.

The design answers two failures the authors saw: agents that tried to do too much at once and ran out of context mid-change, and later agents that saw partial progress and declared the job done [6].

### The memory tool

On the Claude API, the **memory tool** gives the agent a `/memories` directory it can view, create, edit, rename and delete files in. It runs client-side: Claude requests the operation and your application performs it on storage you control [3]. When the tool is enabled, the API adds a protocol to the system prompt that tells Claude to check its memory first and to "ASSUME INTERRUPTION", because its context window "might be reset at any moment" [3]. If the handler doesn't validate paths, a path such as `/memories/../../secrets.env` can reach files outside the directory, so the handler must reject anything that resolves outside `/memories` [3].

Memory pairs with context editing. As the context nears the clearing threshold, Claude gets a warning so it can save important information before old tool results are cleared [4]. In Anthropic's internal agentic-search eval, memory plus context editing improved performance by 39% over baseline, against 29% for context editing alone [2].

### Why notes work

- **State survives** compaction, clearing and restarts, because it lives on disk [1][6].
- **The window stays lean.** Notes are read when needed, not kept in context. This is Anthropic's **just-in-time** approach: keep lightweight identifiers such as file paths and load content with tools at runtime. Folder names, file names and timestamps tell the agent what a file is for before it opens it [1].
- **Structure resists drift.** A fixed format with required fields is harder to quietly corrupt than free prose [6].

## Technique 4: Sub-agent architectures

### The pattern

Instead of one agent holding everything, a **lead agent** plans and coordinates while **sub-agents** do focused work, each in its own clean context window. A sub-agent may use tens of thousands of tokens exploring, but returns "a condensed, distilled summary of its work (often 1,000-2,000 tokens)" [1].

```
User request
   └─► Lead agent (plans, delegates)
         ├─► Sub-agent A (own context + tools) ─► short summary
         ├─► Sub-agent B (own context + tools) ─► short summary
         └─► Sub-agent C (own context + tools) ─► short summary
   ◄── Lead agent combines the summaries ──► answer
```

Anthropic's research system uses this orchestrator-worker design. With Claude Opus 4 as lead and Sonnet 4 sub-agents, it outperformed a single Opus 4 agent by 90.2% on an internal research eval [7]. Sub-agents can also write their output to external storage and pass back only "lightweight references", and the lead saves its plan to memory in case its context gets truncated [7].

In Claude Code, subagents are built in. Explore (read-only search), Plan and general-purpose ship by default, and you can define your own as Markdown files in `.claude/agents/`, each with its own prompt, tool list and model [5].

### The costs

Anthropic is direct about the trade-offs [7]:

- **Tokens:** agents used about 4 times the tokens of chat, and multi-agent systems about 15 times. That only pays off on high-value tasks.
- **Coordination failures:** early versions spawned 50 sub-agents for simple queries, and vague task descriptions led sub-agents to duplicate each other's work. The fix was explicit delegation: objective, output format, tools and task boundaries.
- **Poor fits:** tasks where all agents need the same context or have many dependencies. The authors note that "most coding tasks involve fewer truly parallelizable tasks than research."

Claude Code's docs add that subagent requests count toward the same usage limits, and that many subagents each returning detailed results can still fill the main context [5]. Topic 23 compares subagents with agent teams.

## Choosing a technique

Anthropic's guide matches each technique to a task shape [1]:

| Situation | Reach for |
|---|---|
| Long back-and-forth that needs to keep flowing | **Compaction** |
| Iterative development with clear milestones | **Structured notes** (plus memory files) |
| Complex research or analysis that can run in parallel | **Sub-agents** |

They combine well. A long project typically has a `CLAUDE.md` for standing rules, a progress file and feature list for evolving state, compaction at milestones, and a sub-agent for the occasional large investigation.

## What lasts vs. what will date

- **Durable principles:**
  - Context is a working budget, not storage. Larger windows don't remove the need to curate it.
  - Summarise deliberately, tuning for recall first and precision second.
  - Keep durable state in files outside the window, in a structured format, and load it just in time.
  - Give sub-tasks clean contexts and pass back short summaries or references.
  - Every extra agent costs tokens and coordination. Use them where work really separates.
- **Time-sensitive details:**
  - API feature names, versions and beta status: `clear_tool_uses_20250919`, the memory tool, server-side compaction (as of 2026-10).
  - Claude Code commands and limits: `/compact`, `/autocompact`, auto memory's 200-line/25KB load, built-in subagent names (as of 2026-10).
  - The reported numbers (39%, 84%, 90.2%, 15×), which come from Anthropic's own evaluations of 2025 models.

## Try it yourself

1. **Compact with instructions.** In a long session, run `/context`, then `/compact` with an explicit focus ("keep the data model and open TODOs"). Ask a question that depends on what you told it to keep.
2. **Trim your `CLAUDE.md`.** For each line, ask whether removing it would cause mistakes. Move workflow-specific sections into a skill. Check `/context` before and after.
3. **Set up a progress file.** Ask the agent to keep `progress.md` (done, in progress, next) and a JSON feature list, updating both before each session ends. Start a fresh session and have it get its bearings from those files alone.
4. **Try the memory tool.** On the API, add the memory tool to a small agent with a path-validating handler. Run two sessions and check what it wrote and reread.
5. **Compare one sub-agent split.** Run a research-style task once in a single session and once with "use subagents to investigate X and Y". Compare quality, time and token use.

## Key terms

- **Context window**: everything the model sees on a turn, including system prompt, tools, memory files, files read and conversation.
- **Compaction**: summarising a long context and continuing from the summary in a fresh window.
- **Tool result clearing / context editing**: removing old tool outputs from the context, the lightest form of compaction.
- **Memory file**: a file loaded at session start (`CLAUDE.md`, auto memory's `MEMORY.md`) that carries knowledge between sessions.
- **Structured note-taking**: the agent writing notes to files outside the window and reading them back when needed.
- **Memory tool**: an API tool that lets Claude manage files in a `/memories` directory your application stores.
- **Just-in-time retrieval**: keeping references (paths, queries, links) in context and loading the content only when needed.
- **Sub-agent**: an agent with its own context, prompt and tools that a lead agent delegates to and that returns a condensed result.

## Sources

1. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Anthropic Applied AI team, 2025-09-29). The three long-horizon techniques, compaction tuning, tool result clearing, note-taking examples, sub-agent summaries, just-in-time retrieval, and when to use each.
2. [Managing context on the Claude Developer Platform](https://claude.com/resources/articles/context-management) (Anthropic, 2025-09-29). Launch of context editing and the memory tool with Claude Sonnet 4.5, and the 39% and 84% evaluation results.
3. [Memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) (Claude API docs, accessed 2026-10-14). Commands, client-side design, the injected memory protocol, path-traversal warning, and pairing with compaction.
4. Claude API docs: [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing) and [Compaction](https://platform.claude.com/docs/en/build-with-claude/compaction) (Anthropic, accessed 2026-10-14). Tool result clearing parameters and defaults, the memory warning before clearing, and server-side compaction options.
5. Claude Code documentation: [How Claude remembers your project](https://code.claude.com/docs/en/memory), [Explore the context window](https://code.claude.com/docs/en/context-window), [Best practices](https://code.claude.com/docs/en/best-practices) and [Subagents](https://code.claude.com/docs/en/sub-agents) (Anthropic, accessed 2026-10-14). `CLAUDE.md` and auto memory, what survives compaction, compaction commands, and built-in and custom subagents.
6. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Justin Young, Anthropic, 2025-11-26). The initializer/coding-agent pattern, progress file, JSON feature list, one feature at a time, and the failure modes it addresses.
7. [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (Jeremy Hadfield et al., Anthropic, 2025-06-13). Orchestrator-worker design, the 90.2% result, token costs, coordination failures, and poor-fit task types.
8. [Using Claude Code: session management and 1M context](https://claude.com/resources/articles/using-claude-code-session-management-and-1m-context) (Thariq Shihipar, Anthropic, 2026-04-15). Why automatic compaction can go wrong and why to compact proactively with a hint.
