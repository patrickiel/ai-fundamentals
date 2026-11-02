---
theme: ../theme
chapter: '14'
header: 'AI Fundamentals · 14 · Dynamic context discovery'
layout: cover
---

# Dynamic Context Discovery

**Keep bulky material on disk or behind a search, give the agent pointers, and let it pull what it needs**

---

## Static vs. dynamic context

- **Static:** loaded every time, used or not
- **Dynamic:** fewer details up front; the agent fetches what it needs
- Cursor: saves tokens, and less contradictory material to confuse the model
- Anthropic: **just-in-time** retrieval with lightweight identifiers (paths, queries, links)
- Trade-off: slower, and needs good tools and names

<!-- Cursor's "Dynamic context discovery" (Jan 2026) and Anthropic's "Effective context engineering" (Sept 2025) describe the same shift. Claude Code is a hybrid: CLAUDE.md up front, glob and grep for the rest. -->

---

## 1. Long tool output → files

- Inline output fills the window; **truncation** can cut the part that mattered
- Save in full; agent checks the end (`tail`) and reads more as needed
- Claude Code (as of 2026-10):
  - Bash over ~30,000 chars → file path + 2,000-char preview
  - MCP text over 50,000 chars → file path
- Or filter first: code or a hook returns only matching lines

<!-- Cursor reports fewer unnecessary summarisations after this change. Anthropic's code-execution example: the agent sees five rows instead of 10,000. -->

---

## 2. Chat history stays searchable

- Summaries are lossy, and each round loses more
- Cursor: after summarising, the agent gets a reference to the **full history file**
- Claude Code: summarising from `/rewind` keeps the original messages in the transcript
- Treat the summary as an **index**, the raw record as a **backstop**

<!-- The point is not to avoid summaries, but to keep a way back to the details they drop. -->

---

## 3. Skills: progressive disclosure

1. **Discovery:** only `name` + `description` at startup
2. **Activation:** full `SKILL.md` when a task matches
3. **Execution:** extra files and scripts only as needed

- "The amount of context that can be bundled into a skill is effectively unbounded"
- Open standard; adopted by Claude Code, Cursor, Codex, Gemini CLI, Copilot and more (as of 2026-10)

<!-- Anthropic, "Equipping agents for the real world with Agent Skills", and agentskills.io. Topic 11 covers writing skills; topic 12 their security risks. -->

---

## 4. MCP tools: the problem

- Most clients load **every** tool definition at the start
- Anthropic's example: 5 servers, 58 tools ≈ **55,000 tokens** before any work
- Internally seen: **134,000 tokens** of tool definitions
- More tools → more wrong-tool picks (context confusion)

<!-- From Anthropic's "Introducing advanced tool use" (Nov 2025). -->

---

## 4. MCP tools: three fixes

| Approach | Result reported |
|---|---|
| **Cursor:** one folder per server, names only | **46.9%** fewer agent tokens (runs using MCP) |
| **API Tool Search Tool:** `defer_loading`, search, load | ~**85%** fewer tokens; Opus 4.5 accuracy 79.5% → 88.1% |
| **Code execution:** servers as a file tree, call tools from code | 150,000 → **2,000** tokens |

<!-- Cursor's figure is from an A/B test with high variance by number of servers. Code execution needs a secure sandbox. Cursor chose folders over flat search to keep each server's tools grouped. -->

---

## In Claude Code (as of 2026-10)

- Tool search **on by default**: only tool names + server instructions at startup
- `ENABLE_TOOL_SEARCH=auto` → load up front while under 10% of the window
- `ENABLE_TOOL_SEARCH=false` → load everything
- `alwaysLoad` exempts a server
- CLI tools (`gh`, `aws`) are still leaner than MCP servers

<!-- Server authors: tool search matches on descriptions, so write clear server instructions and put key terms early. -->

---

## 5. Terminal sessions → files

- Old way: paste errors into chat, or inject full command output
- Cursor: integrated terminal synced to files; agent `grep`s them
- Common pattern anywhere:

```bash
npm run dev 2>&1 | tee "logs/dev-$(date +%Y%m%d-%H%M%S).log"
grep -n -i "error" logs/dev-*.log | tail -20
```

<!-- 2>&1 merges stderr into stdout so both are captured. In Claude Code, backgrounded commands already write their output to a file. -->

---

## One pattern, five uses

| Bulky thing | Instead of... | Do |
|---|---|---|
| Tool output | inline / truncate | file + preview + search |
| Chat history | lossy summary only | searchable transcript |
| Skills | load all | name + description |
| MCP definitions | preload all | names; search, then load |
| Terminal output | paste full logs | log file + `grep` |

**Pointers and the current slice in the window. Everything else on disk.**

<!-- If you remember one slide, remember this one. -->

---

## Costs and caveats

- Discovery adds **steps and latency**
- Only works with **good names and descriptions**
- Code execution needs **sandboxing**, limits, monitoring
- Load skills and servers only from **trusted sources**
- Cursor: "not clear if files will be the final interface"

<!-- Anthropic recommends tool search when definitions exceed ~10K tokens or 10+ tools; it helps less with a few small tools used every session. -->

---
zoom: 0.9
---

## What lasts vs. what will date

**Lasts**
- Less in the window → better focus
- Save in full and search > truncate
- Progressive disclosure for tools and skills
- Raw records as a backstop for summaries

**Will date**
- Cursor features; the 46.9% figure
- Tool Search Tool numbers and flags
- Claude Code defaults and thresholds
- Which products support skills

<!-- The filesystem plus grep is a simple retrieval layer that will outlast any product's version of it. -->

---

## Try it yourself

1. Run `/context` before your first prompt; defer what you rarely use
2. Compare `ENABLE_TOOL_SEARCH=false` with the default
3. Trigger a long output; ask a question that needs the saved file
4. Log your dev server; diagnose an error from the log only
5. Write a skill with a sharp description; check it loads only when used

<!-- Start with exercise 1. Most people are surprised how much is loaded before the first prompt. -->

---
layout: end
---

# Key takeaways

- **Pointers in the window, bulk on disk**: let the agent pull, not push
- **Never truncate what you can save**: files can be searched again
- **Progressive disclosure** keeps tool and skill context small

[Cursor: Dynamic context discovery](https://cursor.com/blog/dynamic-context-discovery)\
[Anthropic: Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)\
[Anthropic: Introducing advanced tool use](https://www.anthropic.com/engineering/advanced-tool-use)
