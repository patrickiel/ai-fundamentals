# Dynamic Context Discovery: Let the Agent Pull Context Instead of Pushing It All In

> **Core idea in one sentence:** Keep the always-loaded context small by putting bulky material (tool output, chat history, skill instructions, tool definitions, terminal logs) where the agent can find it, give the agent short names and pointers, and let it load the specific part it needs when it needs it.

## TL;DR

- **Static context** is loaded every time. **Dynamic context discovery** gives the agent less up front and lets it pull in what it needs. Cursor reports this saves tokens and can improve answers by removing confusing or contradictory material [1]. Anthropic calls the same idea **just-in-time** retrieval and **progressive disclosure** [2].
- Cursor applies it in five places: long tool output, chat history, Agent Skills, MCP tools and terminal sessions, mostly by writing things to **files** the agent can search [1].
- For MCP, Cursor's per-server tool folders cut total agent tokens by **46.9%** in runs that called an MCP tool [1]. Anthropic's **Tool Search Tool** cut tool-definition tokens by about 85% and improved tool-selection accuracy [4]. Presenting MCP servers as **code files** cut one example from 150,000 to 2,000 tokens [3].
- **Agent Skills** load in three stages: name and description at startup, full instructions when relevant, extra files only as needed [5][6].
- Claude Code does much of this natively as of 2026-10: MCP tool search is on by default, and oversized Bash and MCP output is saved to a file with a preview [7].
- The cost is extra steps. Discovery is slower than having everything loaded and depends on good names and descriptions [2].

## Static vs. dynamic context

Every token in the window competes for the model's attention (topic 04). The usual instinct is to give the agent everything it might need. Cursor's January 2026 post argues the opposite: as models got better at acting as agents, giving them **fewer details up front** worked better, because they can fetch what they need themselves [1].

Cursor gives two benefits. Only the data the agent actually needs enters the window, which saves tokens. And with less irrelevant material, there is less to confuse the model or contradict the relevant parts [1].

Anthropic's context-engineering guide describes the same shift. Instead of retrieving everything before the model runs, agents keep "lightweight identifiers (file paths, stored queries, web links, etc.)" and load data with tools at runtime. Folder structure, file names and timestamps tell the agent what something is before it opens it, and each step informs the next. Anthropic calls this **progressive disclosure** [2]. Its guide also notes the trade-off: runtime exploration is slower than reading pre-loaded data, and it needs well-designed tools to avoid wasted steps. A **hybrid** often works best. Claude Code loads `CLAUDE.md` up front and uses `glob` and `grep` to find everything else [2].

Cursor chose **files** as the common interface, describing them as a simple primitive that is safer than a new abstraction that might not hold up as agents improve. It also says it's "not clear if files will be the final interface for LLM-based tools" [1].

## 1. Long tool output goes to files

### The problem

Shell commands and MCP tools can return huge outputs. Keeping them inline fills the window. The common alternative, **truncation**, can cut exactly the part that mattered [1].

### The approach

Cursor writes long shell and MCP output to a file. The agent checks the end with `tail` and reads more if it needs to. Cursor reports this led to fewer unnecessary summarisations when the context filled up [1].

Claude Code does the same as of 2026-10 [7]:

- **Bash:** output up to roughly 30,000 characters arrives inline. Beyond that, Claude gets the path of a saved file plus a preview of up to 2,000 characters, and reads or searches the file for the rest.
- **MCP:** Claude Code warns when a tool's output exceeds 10,000 tokens. Successful text results longer than 50,000 characters are saved to a file and replaced in the conversation with the file path.

A related idea is to **filter before the model sees anything**. Anthropic's code-execution approach lets the agent process a large dataset in code and see, for example, five rows instead of 10,000 [3]. Claude Code's docs suggest a hook that greps a log for `ERROR` and returns only matching lines, "reducing context from tens of thousands of tokens to hundreds" [7].

## 2. Chat history stays searchable after summarisation

When the window fills, agents summarise and continue, and summaries lose detail. Cursor gives the agent a reference to the full chat history file after summarisation, so it can search for a detail the summary dropped [1].

Claude Code takes a similar approach when you summarise part of a session from the `/rewind` menu. Summarising "doesn't change files on disk, and the original messages stay in the session transcript, so Claude can still reference the details" [7]. Whatever the tool, the principle is the same: treat the summary as an index and keep the raw record as a backstop.

## 3. Skills: names up front, instructions on demand

**Agent Skills** package instructions, scripts and reference material for a type of task in a folder with a `SKILL.md` file [5]. Anthropic released the format as an open standard in December 2025 [5]. As of 2026-10, the standard's site lists dozens of adopting tools, including Claude Code, Cursor, OpenAI Codex, Gemini CLI, GitHub Copilot and VS Code [6].

Skills load by progressive disclosure in three stages [5][6]:

1. **Discovery:** at startup, only each skill's `name` and `description` go into the system prompt, "just enough information for Claude to know when each skill should be used".
2. **Activation:** when a task matches, the agent reads the full `SKILL.md`.
3. **Execution:** bundled files are opened, or scripts run, only as needed. A script can run "without loading either the script or the PDF into context."

Because of this, Anthropic says "the amount of context that can be bundled into a skill is effectively unbounded" [5]. Cursor finds the right skill files with `grep` and its own semantic search [1]. In Claude Code, a skill marked `disable-model-invocation: true` isn't even listed at startup and costs no context until you run it with `/name` [7]. Topic 11 covers writing skills, and topic 12 covers their supply-chain risks.

## 4. MCP tools: load definitions only when needed

### The problem

Most MCP clients load every connected tool's name, description and schema into context at the start, whether or not it's used. Anthropic's example: five servers with 58 tools took about 55,000 tokens before any work began, and it has seen tool definitions reach 134,000 tokens internally [4]. Large tool sets also make the model more likely to pick the wrong tool [4] (topic 04 calls this context confusion).

### Three implementations of the same idea

| Approach | How it works | Reported result |
|---|---|---|
| **Cursor: tool folders** [1] | Tool descriptions synced to one folder per server. Only tool names are static context, and the agent looks up details when needed | 46.9% fewer total agent tokens in runs that called an MCP tool (A/B test, statistically significant, high variance by number of servers) |
| **Anthropic API: Tool Search Tool** [4] | Tools marked `defer_loading: true` stay out of context. Claude searches (regex, BM25 or custom) and matching definitions are loaded | ~85% fewer tokens in their example. Accuracy on MCP evals: Opus 4 49% → 74%, Opus 4.5 79.5% → 88.1% |
| **Code execution with MCP** [3] | MCP servers presented as a file tree (`servers/<server>/<tool>.ts`). The agent explores it, or calls `search_tools` with a detail level, and writes code that calls the tools | One workflow went from 150,000 to 2,000 tokens (98.7% less) |

Cursor considered a search-based approach but chose folders so each server's tools stay grouped rather than scattered across a flat index. A side benefit: because the tool list lives on disk, the agent can tell the user when a server needs re-authentication instead of silently losing its tools [1].

Anthropic suggests tool search pays off when definitions exceed about 10,000 tokens, when tool selection is a problem, or with 10 or more tools. It helps less with a few small tools that are used every session [4]. Code execution adds its own costs: running agent-written code needs a secure sandbox with resource limits and monitoring [3].

### In Claude Code

As of 2026-10, Claude Code uses tool search **by default**. Only tool names and server instructions load at session start, so "adding more MCP servers has minimal impact on your context window" [7]. You can change this with `ENABLE_TOOL_SEARCH`: `auto` loads tools up front only while their definitions fit within 10% of the window, and `false` loads everything. A server marked `alwaysLoad` is exempt [7]. Because Claude searches by description, MCP server authors should write clear server instructions and put key terms early in tool descriptions [7]. The docs also note that CLI tools such as `gh` or `aws` are still more context-efficient than MCP servers, since they add no per-tool listing [7].

## 5. Terminal sessions as files

Getting terminal output to an agent used to mean pasting it into the chat, or having the agent's own commands inject full output into context. Cursor now syncs its integrated terminal sessions to the local filesystem, so the agent can `grep` for the relevant lines, for example in the logs of a long-running dev server, to answer "why did my command fail?" [1]. In Claude Code, a backgrounded command's output goes to a file that Claude reads with its file tools [7].

A common pattern in any agent is to run long-lived processes with their output captured to a log file and let the agent search it rather than paste it:

```bash
# capture stdout and stderr (2>&1) to a timestamped log
npm run dev 2>&1 | tee "logs/dev-$(date +%Y%m%d-%H%M%S).log"

# later: pull out only what matters
grep -n -i "error" logs/dev-*.log | tail -20
```

## Putting it together

| Bulky thing | Instead of... | Do this |
|---|---|---|
| Tool output | Keeping it inline, or truncating | Save to a file. Preview, then read or search |
| Chat history | Relying only on lossy summaries | Keep the transcript searchable |
| Skills | Loading every instruction | Name + description up front, body on use |
| MCP tool definitions | Preloading every schema | Names up front. Search or browse, then load |
| Terminal output | Pasting full logs | Log to files, then `grep` and `tail` |

The window holds **pointers and the current slice**. Everything else stays on disk or behind a search.

## What lasts vs. what will date

- **Durable principles:**
  - Less in the window means better focus. Irrelevant context costs quality as well as tokens.
  - Save in full and search, rather than truncate or keep everything inline.
  - Use progressive disclosure: short names and descriptions first, details on demand.
  - Keep raw records (transcripts, logs) as a backstop for summaries.
  - Discovery trades latency and extra steps for a smaller context, and it only works with good names and descriptions.
- **Time-sensitive details:**
  - Cursor's specific features and the 46.9% figure (January 2026).
  - Tool Search Tool, `defer_loading` and their benchmark numbers (November 2025, beta at launch).
  - Claude Code defaults: tool search on, `ENABLE_TOOL_SEARCH` values, the 30,000-character Bash and 50,000-character MCP thresholds (as of 2026-10).
  - Which products support Agent Skills (as of 2026-10).

## Try it yourself

1. **Audit your static context.** Run `/context` (or your agent's equivalent) before your first prompt. List what loads up front: memory files, tool names, skill descriptions. Defer or remove what you rarely use.
2. **Compare tool loading.** With several MCP servers connected, start Claude Code once with `ENABLE_TOOL_SEARCH=false` and once with the default. Compare starting context and whether the right tools still get used.
3. **Trigger a file offload.** Run a command with very long output (a verbose build or full test log) and watch the agent receive a file path and preview instead of the full text. Ask it a question that requires searching the file.
4. **Log and grep.** Run your dev server with output captured to a log. Cause an error, then ask the agent to diagnose it from the log only, without re-running anything.
5. **Write a skill with a sharp description.** Turn a repeated workflow into a skill. Check that the agent picks it up from the description alone, and that its body only loads when used.

## Key terms

- **Static context**: what loads into every session up front (system prompt, memory files, tool and skill listings).
- **Dynamic context discovery**: giving the agent pointers and letting it fetch details on demand. Cursor's term.
- **Just-in-time retrieval**: keeping lightweight identifiers in context and loading the data with tools at runtime. Anthropic's term.
- **Progressive disclosure**: showing a short name and description first, and revealing full instructions or resources only when needed.
- **Truncation**: cutting a long output to fit the window, at the risk of losing the important part.
- **Tool search**: deferring tool definitions and letting the model search for and load the ones it needs.
- **Agent Skill**: a folder with a `SKILL.md` (name, description, instructions) plus optional scripts and resources, loaded by progressive disclosure.
- **MCP tool definition**: the name, description and input schema of a tool exposed by an MCP server.

## Sources

1. [Dynamic context discovery](https://cursor.com/blog/dynamic-context-discovery) (Jediah Katz, Cursor, 2026-01-06). The static vs. dynamic framing, the five applications, the 46.9% MCP result and its caveats, and why files were chosen.
2. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Anthropic Applied AI team, 2025-09-29). Just-in-time retrieval, lightweight identifiers, progressive disclosure, the hybrid strategy, and the speed trade-off.
3. [Code execution with MCP: Building more efficient agents](https://www.anthropic.com/engineering/code-execution-with-mcp) (Adam Jones and Conor Kelly, Anthropic, 2025-11-04). MCP servers as a file tree, `search_tools` detail levels, the 150,000→2,000-token example, filtering in code, and sandboxing costs.
4. [Introducing advanced tool use on the Claude Developer Platform](https://www.anthropic.com/engineering/advanced-tool-use) (Bin Wu et al., Anthropic, 2025-11-24). The Tool Search Tool, `defer_loading`, token and accuracy results, and when tool search helps.
5. [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) (Barry Zhang, Keith Lazuka, Mahesh Murag, Anthropic, 2025-10-16, updated 2025-12-18). The three levels of progressive disclosure, unbounded skill context, scripts outside the context, and the open-standard release.
6. [Agent Skills overview](https://agentskills.io/) (Agent Skills open standard, accessed 2026-10-14). The discovery/activation/execution stages and the list of adopting products.
7. Claude Code documentation: [MCP](https://code.claude.com/docs/en/mcp), [Tools reference](https://code.claude.com/docs/en/tools-reference), [Checkpointing](https://code.claude.com/docs/en/checkpointing), [Explore the context window](https://code.claude.com/docs/en/context-window) and [Manage costs](https://code.claude.com/docs/en/costs) (Anthropic, accessed 2026-10-14). Default tool search and its settings, output-to-file thresholds, searchable transcripts after summarising, skill loading, and hook-based filtering.
