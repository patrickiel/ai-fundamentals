# Standards for Agents: MCP for Tools, AGENTS.md for Project Context

> **Core idea in one sentence:** MCP gave AI applications one shared way to connect to tools and data, and AGENTS.md applies the same move to project context: one plain Markdown file that any participating coding agent reads, instead of a separate rules file for every tool.

## TL;DR

- **MCP is a convention, not a new model ability.** Models could already call functions. MCP standardises how an AI application discovers and calls tools and data sources, so each side implements the protocol once instead of writing glue code for every pair [1][2][3].
- MCP is built on **JSON-RPC 2.0** and borrows its shape from the Language Server Protocol: hosts, clients and servers, with servers offering tools, resources and prompts [3].
- Coding agents need **project context** at the start of each session. Each tool invented its own file, and copies kept by different writers **drift apart**.
- **AGENTS.md** is a plain-Markdown "README for agents" that a long list of coding agents read. No required fields, nearest file wins, chat prompts override [4].
- Both now sit in the **Agentic AI Foundation** under the Linux Foundation, founded in December 2025 with MCP, AGENTS.md and goose as its first projects [5][6].
- As of 2026-10, **Claude Code reads AGENTS.md** when a repo has no CLAUDE.md [7]. **MCP server configuration** is still per client, though a portable `.mcp.json` format is spreading [7][8].

## What MCP actually solved

Anthropic released the **Model Context Protocol** on 2024-11-25 as "a new standard for connecting AI assistants to the systems where data lives." The announcement names the problem directly: "Every new data source requires its own custom implementation, making truly connected systems difficult to scale" [1]. The launch included the specification and SDKs, local MCP server support in the Claude Desktop apps, and an open-source repository of ready-made servers for systems such as Google Drive, Slack, GitHub, Git and Postgres [1].

None of this gave models a new skill. Function calling already existed. What MCP added was an **agreement** on how the two sides talk:

```
Without a shared protocol:          With MCP:
  App A ──custom code── Tool 1        App A ─┐           ┌─ Server 1
  App A ──custom code── Tool 2        App B ─┼── MCP ────┼─ Server 2
  App B ──custom code── Tool 1        App C ─┘           └─ Server 3
  App B ──custom code── Tool 2
  (one integration per pair)          (each side implements MCP once)
```

The MCP docs put it as an analogy: "Think of MCP like a USB-C port for AI applications" [2]. The specification makes a more technical comparison. MCP "takes some inspiration from the Language Server Protocol", which standardised how editors support programming languages, and does the same for "context and tools" in AI applications [3].

### The moving parts

The specification (current revision **2026-07-28**, as of 2026-10) defines three roles that exchange JSON-RPC 2.0 messages [3]:

| Role | What it is | Example |
|---|---|---|
| **Host** | The LLM application that starts connections | A chat app, an IDE, a coding agent |
| **Client** | A connector inside the host, one per server | The host's link to your GitHub server |
| **Server** | A service that offers context and capabilities | A server exposing your issue tracker |

Servers can offer **tools** (functions the model can call), **resources** (data for the user or model) and **prompts** (templated messages and workflows) [3]. Topic 10 builds a server and looks at tools in detail.

The protocol keeps changing. The 2026-07-28 revision made MCP stateless: it removed the `initialize` handshake and protocol-level sessions, moved long-running tasks into an optional extension, and deprecated the roots, sampling and logging features [3]. If you read an older tutorial, expect differences.

### Security is part of the standard

The specification says MCP "enables powerful capabilities through arbitrary data access and code execution paths". It asks hosts to get explicit user consent before exposing data or invoking tools, and to treat tool descriptions and annotations as untrusted unless they come from a trusted server [3]. A standard plug makes it easy to connect many servers, and that includes servers you haven't reviewed. Topic 13 covers what can go wrong when untrusted content and tool access meet.

### From one company's protocol to shared governance

On 2025-12-09 the Linux Foundation announced the **Agentic AI Foundation (AAIF)**. Its founding projects were Anthropic's MCP, Block's goose and OpenAI's AGENTS.md. Platinum members include AWS, Anthropic, Block, Bloomberg, Cloudflare, Google, Microsoft and OpenAI [5]. Anthropic's post on the donation reported more than 10,000 active public MCP servers and 97M+ monthly SDK downloads across Python and TypeScript, and adoption by ChatGPT, Cursor, Gemini, Microsoft Copilot and VS Code. It also said MCP's governance model "will remain unchanged" [6]. The figures are from December 2025.

## The next fragmentation problem: context files

### Why agents need a context file

Each agent session starts with an empty context window. Claude Code's documentation states it plainly: "Each Claude Code session begins with a fresh context window", and a project instruction file is how knowledge carries across sessions [7]. Coding agents load such a file automatically at startup. It holds what you would otherwise repeat every time: build and test commands, conventions, project layout, "always do X" rules [7]. Writing that down is a basic piece of context engineering (topic 03).

### Every tool picked a different file

Each agent shipped with its own file name and format: `CLAUDE.md` for Claude Code, `GEMINI.md` by default for Gemini CLI, and rules files in other tools. If you use more than one agent on the same repository, you end up with several files that should say the same thing.

### Duplication is the small problem; drift is the big one

Copying one file into another is tedious but manageable. The harder problem appears once agents start updating their own files:

- Each tool writes only to **its own** file. A decision recorded by one agent never reaches the others.
- Nobody remembers to sync the copies by hand.
- Each agent starts the next session from a different, partial view of the project.

This is a general rule, not something specific to AI: **several copies of the same state, each with its own writer, will diverge.** The fix is one source of truth.

## AGENTS.md: one file for all agents

### What it is

The AGENTS.md site describes the format as a "README for agents: a dedicated, predictable place to provide the context and instructions to help AI coding agents work on your project" [4]. READMEs stay focused on people. AGENTS.md holds the extra detail agents need. The rules are deliberately minimal [4]:

- **Plain Markdown, no required fields.** Use whatever headings help.
- **Nested files for monorepos.** Agents read the nearest AGENTS.md in the directory tree. The site notes that OpenAI's main repository has 88 of them.
- **Precedence:** "The closest AGENTS.md to the edited file wins; explicit user chat prompts override everything."
- **Commands get run.** If you list test commands, agents try to run the relevant checks and fix failures before finishing.
- **Migration:** rename your existing file to AGENTS.md and leave a symbolic link under the old name.

A minimal example in the style of the site's samples:

```markdown
# AGENTS.md

## Setup commands
- Install deps: pnpm install
- Start dev server: pnpm dev
- Run tests: pnpm test

## Code style
- TypeScript strict mode
- Single quotes, no semicolons

## PR instructions
- Run lint and tests before committing
```

What makes it work isn't the content, it's the **agreement**: every participating agent looks in the same place, so you write the file once and every tool reads and updates the same copy.

### Who reads it (as of 2026-10)

The format came out of collaboration between OpenAI Codex, Amp, Google's Jules, Cursor and Factory, and the AGENTS.md project is now stewarded by the AAIF [4][5]. The site lists 23 compatible agents and tools, including Codex, Jules, Gemini CLI, Cursor, GitHub Copilot's coding agent, VS Code, Zed, Warp, Aider, goose, opencode, Devin, Windsurf, Junie, Amp, RooCode and Augment Code. It reports use by more than 60,000 open-source projects [4]. Some tools need one setting: Aider reads it via `read: AGENTS.md` in `.aider.conf.yml`, and Gemini CLI via `context.fileName` in `.gemini/settings.json` [4].

### Claude Code

Claude Code keeps its own `CLAUDE.md`, but as of 2026-10 its documentation says it "can read `AGENTS.md` as your project instructions, so a repository already set up for other coding agents works without adding a `CLAUDE.md`, an import, or a setting" (v2.1.277 or later) [7]. The defaults are worth knowing:

| Your repository has | Claude Code reads |
|---|---|
| `AGENTS.md`, no `CLAUDE.md` / `CLAUDE.local.md` | `AGENTS.md` |
| `AGENTS.md` **and** a `CLAUDE.md` | `CLAUDE.md` only |
| A `CLAUDE.md` that imports `@AGENTS.md` | `CLAUDE.md`, with `AGENTS.md` pulled in |

A **Project instructions** setting changes the default, for example to read both [7]. One trap: adding a personal `CLAUDE.local.md` to a repo that relies on AGENTS.md stops Claude Code from reading AGENTS.md unless you change that setting [7].

### What it does not fix

AGENTS.md covers instructions only. **MCP server configuration is still set per client.** Claude Code keeps project servers in `.mcp.json` under an `mcpServers` key, and personal ones in `~/.claude.json` [7]. VS Code has its own `.vscode/mcp.json` with a `servers` key. As of 2026-10 VS Code also reads a portable `.mcp.json` with `mcpServers` and recommends the portable formats for new servers [8]. That's convergence by imitation, not a formal standard, so check each client's docs.

## The underlying lesson

| Layer | Fragmented state | Shared convention |
|---|---|---|
| Tool access | Custom glue per app–tool pair | MCP |
| Project instructions | One rules file per agent, drifting apart | AGENTS.md |
| Tool configuration | One config format per client | Partly converging on `.mcp.json` (as of 2026-10) |

In each case the value comes from **agreement, not new technology**. When many producers and consumers need the same thing, a simple shared format beats a better proprietary one. When state lives in several copies with different writers, keep **one source of truth**.

## What lasts vs. what will date

- **Durable principles:**
  - A shared protocol turns many-to-many integration into "implement once" on each side. That is MCP's real contribution.
  - Agents start each session blank. Project context must be written down and loaded automatically.
  - Keep one source of truth for shared state. Copies with different writers drift.
  - A convention only helps where your tools honour it. Check adoption before relying on it.
  - Connecting a tool is a trust decision. Consent and review still apply.
- **Time-sensitive details:**
  - The spec revision (2026-07-28) and its stateless design, deprecated features and extensions.
  - Adoption figures (10,000+ servers and 97M+ monthly downloads as of Dec 2025, 60,000+ projects for AGENTS.md as of 2026-10).
  - The list of agents that read AGENTS.md, and Claude Code's version requirement and loading rules.
  - MCP config file names, keys and locations per client.

## Try it yourself

1. **Inventory your context files.** In one repo, list every agent-specific rules file (`CLAUDE.md`, `GEMINI.md`, Cursor or Copilot rules). Diff them and note the drift.
2. **Consolidate into AGENTS.md.** Merge the useful content into one `AGENTS.md`. Open a fresh session in two different agents and ask each, "How do I run the tests here?"
3. **Check how Claude Code resolves it.** Run `/context` in Claude Code and look at the memory files listed. Then add a `CLAUDE.md` containing `@AGENTS.md` and check again [7].
4. **Test write-back.** Ask two agents to record a decision "in the project instructions file". Do both write to the same file?
5. **Map your MCP configs.** List where each client stores its server config and which top-level key it uses. Try one portable `.mcp.json` across two clients.

## Key terms

- **Model Context Protocol (MCP)**: an open, JSON-RPC-based standard for connecting AI applications to tools, data and workflows.
- **Host / client / server**: the AI application, its per-server connector, and the service that offers tools, resources and prompts.
- **Integration problem**: needing custom glue code for every app–tool pair. A shared protocol reduces it to one implementation per side.
- **Context file / instructions file**: a Markdown file an agent loads at session start to learn a project's rules and commands.
- **AGENTS.md**: an open, plain-Markdown format for instructions to coding agents, read by many tools and stewarded by the AAIF.
- **Agentic AI Foundation (AAIF)**: a Linux Foundation directed fund formed in December 2025 to host open agent standards, starting with MCP, goose and AGENTS.md.
- **Drift**: copies of the same information diverging because each is updated separately.
- **Single source of truth**: one canonical copy of shared state that every reader and writer uses.

## Sources

1. [Introducing the Model Context Protocol](https://www.anthropic.com/news/model-context-protocol) (Anthropic, 2024-11-25). The launch announcement: the integration problem, what shipped (spec, SDKs, Claude Desktop support, server repository), and early adopters.
2. [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/docs/getting-started/intro) (MCP documentation, accessed 2026-10-14). The USB-C analogy and current client support.
3. [MCP Specification, revision 2026-07-28](https://modelcontextprotocol.io/specification/latest) and its [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) (Model Context Protocol, 2026-07-28). Hosts/clients/servers, JSON-RPC base, the LSP comparison, server features, security principles, and the move to a stateless protocol.
4. [AGENTS.md](https://agents.md/) and its [repository](https://github.com/agentsmd/agents.md) (AGENTS.md project, accessed 2026-10-14). The format, nested-file precedence, migration advice, the list of compatible agents, adoption count, and AAIF stewardship.
5. [Linux Foundation Announces the Formation of the Agentic AI Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation) (Linux Foundation, 2025-12-09). The founding projects (MCP, goose, AGENTS.md), members, and adoption figures.
6. [Donating the Model Context Protocol and establishing the Agentic AI Foundation](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation) (Anthropic, 2025-12-09). Why MCP moved to neutral governance, adoption numbers, and the statement that its governance model stays unchanged.
7. Claude Code documentation: [How Claude remembers your project](https://code.claude.com/docs/en/memory) and [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp) (Anthropic, accessed 2026-10-14). What a project instruction file is for, native AGENTS.md support and its loading rules, and where MCP config lives.
8. [Add and manage MCP servers in VS Code](https://code.visualstudio.com/docs/copilot/chat/mcp-servers) (Microsoft, accessed 2026-10-14). VS Code's own `mcp.json` format and its support for the portable `.mcp.json`.
