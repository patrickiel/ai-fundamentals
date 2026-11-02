---
theme: ../theme
chapter: '05'
header: 'AI Fundamentals · 05 · Compaction, notes, sub-agents'
layout: cover
---

# Long-Running Agents

**Compress the history, keep state in files, and split big jobs into clean windows**

---

## The problem

- Each turn the model sees one finite **context window**
  - system prompt · tools · memory files · file reads · tool results
- Some tasks need far more tokens than any window holds
- Bigger windows don't solve it:

> "context windows of all sizes will be subject to context pollution and information relevance concerns"

- Anthropic's three techniques: **compaction · structured notes · sub-agents**

<!-- Quote from Anthropic's "Effective context engineering for AI agents" (Sept 2025). Topic 04 covers why quality drops as context grows. -->

---

## Compaction

- Summarise a nearly full context, continue in a **fresh window**
- Claude Code keeps: architectural decisions, unresolved bugs, implementation details
- Drops: redundant tool output
- Writing your own? Tune for **recall first**, then **precision**

```
[long history ...................] → [summary] + room to work
```

<!-- Recall first so nothing important is lost; then trim. That advice is for anyone writing a compaction prompt. -->

---

## Automatic vs. manual

| | Automatic | Manual |
|---|---|---|
| When | Near the limit (`/autocompact` sets it) | When you run `/compact` |
| What's kept | Model's guess | What you say: `/compact focus on the API changes` |
| Risk | Fires mid-task, when rot is worst | You must remember |

- Standing rules in `CLAUDE.md`: "when compacting, preserve modified files and test commands"
- Conversation-only instructions get summarised away

<!-- Claude Code commands as of 2026-10. After compaction, root CLAUDE.md, auto memory and the plan reload from disk. -->

---

## Lightest touch: clear old tool results

- Old raw tool output is rarely needed again
- API **context editing** (beta, as of 2026-10): `clear_tool_uses_20250919`
  - default trigger 100k input tokens · keeps last 3 tool uses · exempt tools you need
- Anthropic's 100-turn web-search eval: workflows finished, **84%** fewer tokens
- API also offers server-side compaction (beta)

<!-- Context editing and the memory tool launched with Claude Sonnet 4.5 in September 2025. Numbers are Anthropic's own evaluation. -->

---

## Persistent memory files

| | `CLAUDE.md` | Auto memory |
|---|---|---|
| Written by | You | Claude |
| Holds | Rules, commands, conventions | Learnings, corrections |
| Loaded | Every session, in full | First 200 lines / 25KB of `MEMORY.md` |

- Keep `CLAUDE.md` under ~200 lines; move occasional workflows into **skills**
- "Remember X" → auto memory; ask explicitly for `CLAUDE.md`

<!-- Each Claude Code session starts with a fresh window. These two files are how knowledge carries over. Topic 11 covers skills. -->

---

## Structured note-taking

- Agent **writes notes to files** outside the window, reads them back when needed
- Examples: to-do lists, `NOTES.md`
- Claude playing Pokémon kept notes across thousands of steps:

> "for the last 1,234 steps I've been training my Pokémon in Route 1"

- State survives compaction, clearing, restarts

<!-- From Anthropic's context-engineering guide. The notes act as the agent's memory between context resets. -->

---

## A worked pattern for coding

- **Initializer session:** feature list (all failing) · `claude-progress.txt` · `init.sh` · first commit
- **Each coding session:**
  - read progress file + git log → pick next feature
  - run a basic end-to-end test first
  - **one feature at a time**, commit, update notes
- Feature list in **JSON**: the model was less likely to wrongly edit it than Markdown

<!-- Anthropic, "Effective harnesses for long-running agents" (Nov 2025). It fixes two failures: doing too much at once, and declaring the job done too early. -->

---

## The memory tool (API)

- Claude manages files in `/memories`: view · create · str_replace · insert · delete · rename
- **Client-side**: your app runs each operation on your storage
- Injected protocol: check memory first, **"ASSUME INTERRUPTION"**
- Pairs with context editing: Claude is warned before old results are cleared
- Memory + context editing: **+39%** on an internal agentic-search eval
- Validate paths: block `/memories/../../secrets.env`

<!-- Path traversal is the main security note in the docs. Your handler must reject anything resolving outside /memories. -->

---

## Sub-agent architecture

```
User ─► Lead agent (plans, delegates)
          ├─► Sub-agent A (clean context) ─► summary
          ├─► Sub-agent B (clean context) ─► summary
          └─► Sub-agent C (clean context) ─► summary
User ◄─ Lead combines summaries
```

- Sub-agent may use tens of thousands of tokens, returns **~1,000–2,000**
- Can write output to storage and return **references**
- Claude Code: built-in Explore, Plan, general-purpose; custom in `.claude/agents/`

<!-- Anthropic's research system: Opus 4 lead with Sonnet 4 sub-agents beat a single Opus 4 agent by 90.2% on its internal eval. -->

---

## Sub-agents have costs

- **Tokens:** agents ≈ 4× chat; multi-agent ≈ **15×** chat
- **Coordination failures:** 50 sub-agents for a simple query; duplicated work
  - Fix: explicit objective, output format, tools, boundaries
- **Poor fits:** shared context, many dependencies, most coding tasks
- Sub-agent requests count toward the same usage limits

<!-- "Most coding tasks involve fewer truly parallelizable tasks than research." Topic 23 compares subagents and agent teams. -->

---

## Which technique when?

| Situation | Reach for |
|---|---|
| Long back-and-forth that must keep flowing | **Compaction** |
| Iterative development with clear milestones | **Structured notes** + memory files |
| Complex research that can run in parallel | **Sub-agents** |

Combine them: `CLAUDE.md` for rules · progress file for state · compaction at milestones · sub-agents for big investigations

<!-- This mapping is from Anthropic's guide. Choose by task shape, not by which feature is newest. -->

---

## What lasts vs. what will date

**Lasts**
- Context is a budget, not storage
- Summarise for recall, then precision
- Durable state in structured files, loaded just in time
- Clean contexts for sub-tasks; return summaries or references

**Will date**
- API feature names, versions, beta status
- Claude Code commands, auto memory limits, subagent names
- 39% / 84% / 90.2% / 15× (2025 evaluations)

<!-- The three techniques predate these products and will outlast them. -->

---

## Try it yourself

1. `/context` → `/compact` with a focus → ask about what you kept
2. Prune `CLAUDE.md`; move a workflow into a skill; compare `/context`
3. Keep `progress.md` + a JSON feature list; resume a fresh session from them
4. Build a small API agent with the memory tool and a path-validating handler
5. Run one research task with and without sub-agents; compare quality and tokens

<!-- Each exercise isolates one technique so you can see its effect. -->

---
layout: end
---

# Key takeaways

- **Compact deliberately**, with instructions, and clear old tool output first
- **Keep state in files**: memory files, progress logs, feature lists
- **Split work into clean sub-agent contexts** only where it truly separates

[Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)\
[Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)\
[Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
