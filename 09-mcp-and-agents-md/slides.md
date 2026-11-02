---
theme: ../theme
chapter: '09'
header: 'AI Fundamentals · 09 · MCP and AGENTS.md'
layout: cover
---

# Standards for Agents

**MCP gave AI apps one way to reach tools. AGENTS.md gives coding agents one file for project context.**

---

## The problem

- Every app–tool pair used to need **custom glue code**
- Every coding agent invented **its own** instructions file
- Copies of the same context, each with its own writer, **drift**
- Both problems have the same fix: a **shared convention**

<!-- Two layers, one lesson. MCP addresses tool access; AGENTS.md addresses project instructions. -->

---

## MCP: a standard, not a new ability

- Announced by Anthropic, **2024-11-25**
- "Every new data source requires its own custom implementation"
- Shipped: spec + SDKs, Claude Desktop support, open-source servers
- Docs: "Think of MCP like a **USB-C port** for AI applications"
- Spec: inspired by the **Language Server Protocol**

<!-- Function calling already existed. MCP standardised how apps discover and call tools, so each side implements it once. Sources: Anthropic announcement, MCP docs and spec. -->

---

## How MCP fits together

```
Host (chat app, IDE, agent)
 ├─ Client ──JSON-RPC 2.0──▶ Server: tools · resources · prompts
 ├─ Client ──JSON-RPC 2.0──▶ Server
 └─ Client ──JSON-RPC 2.0──▶ Server
```

- **Tools**: functions the model calls
- **Resources**: data for user or model
- **Prompts**: templated messages and workflows
- Current spec revision **2026-07-28**: stateless, no handshake

<!-- The 2026-07-28 revision removed sessions and the initialize handshake, moved tasks to an extension, and deprecated roots, sampling and logging. Older tutorials will differ. -->

---

## Security is in the spec

- MCP opens "arbitrary data access and code execution paths"
- Hosts must get **explicit user consent** before tools run
- Tool descriptions and annotations are **untrusted** unless the server is trusted
- An easy plug also makes it easy to connect servers you haven't reviewed

<!-- See topic 13 for what happens when untrusted content meets tool access. -->

---

## Shared governance (Dec 2025)

- Linux Foundation forms the **Agentic AI Foundation (AAIF)**
- Founding projects: **MCP** (Anthropic), **goose** (Block), **AGENTS.md** (OpenAI)
- Anthropic: 10,000+ active public servers, 97M+ monthly SDK downloads
- MCP's governance model "will remain unchanged"

<!-- Figures are from the December 2025 announcements. Platinum members include AWS, Anthropic, Block, Bloomberg, Cloudflare, Google, Microsoft and OpenAI. -->

---

## Why agents need a context file

- Each session starts with a **fresh context window**
- Agents auto-load an instructions file at startup
- It holds what you'd otherwise repeat: commands, conventions, layout, rules
- But each tool chose **its own file**: `CLAUDE.md`, `GEMINI.md`, rules folders...

<!-- Writing project context down is basic context engineering (topic 03). -->

---

## Duplication is small. Drift is big.

- Copy one file into another: tedious but doable
- Then each agent **updates only its own copy**
- Nobody syncs them by hand
- Each agent starts from a **different, partial** view

> Several copies of the same state, each with its own writer, will diverge.

<!-- A general software lesson. The fix is a single source of truth. -->

---

## AGENTS.md: a README for agents

- Plain Markdown, **no required fields**
- **Nearest file wins**; chat prompts override everything
- Nested files for monorepos (OpenAI's main repo has 88)
- Listed test commands get run by agents
- Migrating: rename, keep a symlink under the old name

<!-- Source: agents.md. The value is the agreement that every participating agent looks in the same place. -->

---

## Who reads it (as of 2026-10)

- Format came from Codex, Amp, Jules, Cursor, Factory; now stewarded by the **AAIF**
- 23 listed tools, e.g. Codex, Jules, Gemini CLI, Cursor, Copilot coding agent, VS Code, Zed, Aider, goose, Devin, Windsurf
- 60,000+ open-source projects
- Some need one setting: Aider `read: AGENTS.md`, Gemini CLI `context.fileName`

<!-- Check the agents.md site for the current list before relying on it. -->

---

## Claude Code and AGENTS.md

| Repo has | Claude Code reads |
|---|---|
| `AGENTS.md`, no `CLAUDE.md` | `AGENTS.md` |
| `AGENTS.md` + `CLAUDE.md` | `CLAUDE.md` only |
| `CLAUDE.md` with `@AGENTS.md` | both, via the import |

- Native support from v2.1.277 (as of 2026-10)
- A **Project instructions** setting changes the default
- Trap: a `CLAUDE.local.md` stops AGENTS.md loading by default

<!-- Run /context to see which instruction files actually loaded. -->

---

## What it doesn't fix: MCP config

- AGENTS.md covers **instructions** only
- Server config is still **per client**
  - Claude Code: `.mcp.json` / `~/.claude.json`, key `mcpServers`
  - VS Code: `.vscode/mcp.json`, key `servers`
- VS Code now also reads a portable `.mcp.json` with `mcpServers`

<!-- Convergence by imitation, not a formal standard. Check each client's docs. -->

---

## One lesson, three layers

| Layer | Fragmented | Shared convention |
|---|---|---|
| Tool access | glue per app–tool pair | **MCP** |
| Project instructions | one file per agent | **AGENTS.md** |
| Tool config | one format per client | partly converging (`.mcp.json`) |

**Agreement beats novelty. Keep one source of truth.**

<!-- When many producers and consumers need the same thing, a simple shared format wins. -->

---

## What lasts vs. what will date

**Lasts**
- Shared protocols turn many-to-many into implement-once
- Agents start blank; project context must be written down
- One source of truth prevents drift
- Connecting a tool is a trust decision

**Will date**
- Spec revision details, adoption numbers
- Which agents read AGENTS.md; Claude Code's loading rules
- MCP config file names and keys

<!-- Re-check the spec changelog and the agents.md list before teaching specifics. -->

---

## Try it yourself

1. Inventory and diff every agent rules file in one repo
2. Merge them into one `AGENTS.md`; ask two agents how to run tests
3. In Claude Code, run `/context`; then add `@AGENTS.md` to a `CLAUDE.md` and compare
4. Ask two agents to record a decision: same file or drift?
5. Map where each client stores MCP config; try one portable `.mcp.json`

<!-- Exercise 1 usually shows drift you didn't know about. -->

---
layout: end
---

# Key takeaways

- **MCP is a convention**, and that's why it mattered
- **One AGENTS.md** for all agents avoids duplication and drift
- **Check adoption and trust**: a standard helps only where your tools honour it

modelcontextprotocol.io/specification/latest · agents.md · code.claude.com/docs/en/memory
