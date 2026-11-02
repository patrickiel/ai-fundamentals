# How MCP Servers Work: Tools as Described Functions

> **Core idea in one sentence:** An MCP server is a small program that exposes ordinary functions as tools, each with a name, a description and a typed input schema, so a model can decide when to call it, supply the right arguments and use what comes back.

## TL;DR

- An MCP **tool** is a function plus metadata the model reads: a `name`, a `description` and an `inputSchema` (JSON Schema), optionally an `outputSchema` [3].
- The official SDKs generate that metadata for you. In Python, a decorated function's **type hints and docstring become the tool definition** [1][2].
- A local server is **a command the client launches**. It talks JSON-RPC over stdin/stdout, so it must never print to stdout [1].
- The **description is the model's main guide** for choosing a tool. Anthropic's guidance: write it as you would for a new colleague, and expect small wording changes to matter [4].
- Return **actionable errors** as tool results (`isError: true`) so the model can correct itself. Reserve protocol errors for broken requests [3].
- Test with the **MCP Inspector** before you connect a real client [5]. To have an agent write the server, give it the current docs, because the SDKs change (Python SDK 2.x renamed `FastMCP` to `MCPServer`, as of 2026-10) [2].

## The mental model: input → output, plus a description

A tool is a function. What MCP adds is a standard way for a client to find out what functions exist and how to call them. The quickstart describes what happens when you ask a question [1]:

1. The client sends your question to the model.
2. The model looks at the available tools and decides which to use.
3. The client runs the chosen tool through the MCP server.
4. The result goes back to the model.
5. The model writes a natural-language answer.

On the wire, step 2 depends on a `tools/list` request, which returns each tool's definition, and step 3 is a `tools/call` request with a tool name and arguments [3]. The server's job is to wrap your function so it answers those two requests. The specification calls tools **model-controlled**: the model picks them. It also says there **SHOULD** always be a human able to deny a tool call [3].

## Step 1: Pick an SDK

The MCP docs list official SDKs for TypeScript, Python, C#, Go, Rust and Ruby (Tier 1), Java (Tier 2), and Swift, PHP and Kotlin (Tier 3), as of 2026-10 [2]. This topic uses Python. The quickstart sets up a project with `uv` [1]:

```bash
uv init currency
cd currency
uv venv
uv add "mcp[cli]"     # the [cli] extra adds the `mcp` command
```

`pip install "mcp[cli]"` works too [2].

> **Version note (as of 2026-10):** Python SDK 2.x renamed the high-level `FastMCP` class to `MCPServer`. The import is now `from mcp.server import MCPServer`, and the old `mcp.server.fastmcp` path "is gone rather than deprecated" [2]. Most tutorials written before mid-2026 use the old name.

## Step 2: Write a minimal server

A dollars-to-pounds converter with a fixed rate is enough to show every part:

```python
# currency.py
from mcp.server import MCPServer

mcp = MCPServer("currency")

USD_TO_GBP = 0.79  # fixed rate, for illustration only

@mcp.tool()
def convert_usd_to_gbp(amount: float) -> float:
    """Convert an amount in US dollars to British pounds.

    Uses a fixed illustrative rate, not live market data.

    Args:
        amount: Amount in US dollars
    """
    return round(amount * USD_TO_GBP, 2)

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

The quickstart explains that the class "uses Python type hints and docstrings to automatically generate tool definitions" [1]. The definition a client receives from `tools/list` looks roughly like this (exact output varies by SDK version):

```json
{
  "name": "convert_usd_to_gbp",
  "description": "Convert an amount in US dollars to British pounds. ...",
  "inputSchema": {
    "type": "object",
    "properties": { "amount": { "type": "number" } },
    "required": ["amount"]
  }
}
```

| Part | In the code | What the model gets from it |
|---|---|---|
| **Name** | function name | A handle that should say what the tool does |
| **Description** | docstring | What the tool does and when to use it |
| **Input schema** | type hints | Which arguments to supply, and their types |
| **Output schema** (optional) | return type | The shape of structured results [3] |

The specification adds rules for names: 1–128 characters, case-sensitive, using only letters, digits, `_`, `-` and `.`, unique within a server [3]. It also warns that two servers can both expose a tool called `search`. Clients that combine servers SHOULD disambiguate, for example by prefixing the server name [3].

### The stdout trap

A stdio server exchanges JSON-RPC messages over standard input and output. The quickstart is blunt: "Never write to stdout. Writing to stdout will corrupt the JSON-RPC messages and break your server." Use the `logging` module, which writes to stderr [1]. A stray `print()` is the most common reason a first server "connects but does nothing".

## Step 3: Test it before connecting a client

The **MCP Inspector** is the reference tool for testing servers. It runs through `npx` (Node 22.19.0 or newer as of 2026-10) and takes the command that starts your server [5]:

```bash
npx @modelcontextprotocol/inspector uv --directory /ABSOLUTE/PATH/currency run currency.py
```

The web UI lets you list the tools, call one with arguments and see the raw JSON-RPC exchange. A `--cli` mode makes it scriptable, for example `--method tools/list` [5]. The Python SDK also has a shortcut, `uv run mcp dev currency.py`, which opens the Inspector [2].

## Step 4: Register the server with a client

For a local server you tell the client **which command starts it**. In Claude Desktop that goes in `claude_desktop_config.json`, under `mcpServers` [1]:

```json
{
  "mcpServers": {
    "currency": {
      "command": "uv",
      "args": ["--directory", "/ABSOLUTE/PATH/currency", "run", "currency.py"]
    }
  }
}
```

The quickstart stresses **absolute paths**, notes that `command` may need the full path to `uv`, and says to restart the app after editing [1]. Other clients store the same information elsewhere (topic 09 compares config files). Once connected, ask "convert $20 into pounds" and watch the model choose the tool, pass `amount = 20`, and use the result.

## Designing tools the model can use well

Getting a server running is the easy part. Anthropic's engineering post "Writing effective tools for agents" treats tools as "a new kind of software which reflects a contract between deterministic systems and non-deterministic agents" [4]. Its main recommendations:

- **Describe tools for a newcomer.** "Think of how you would describe your tool to a new hire on your team." Spell out formats, terms and relationships the model can't infer. "Even small refinements to tool descriptions can yield dramatic improvements" [4].
- **Name parameters unambiguously.** "Instead of a parameter named `user`, try a parameter named `user_id`" [4].
- **Build fewer, more capable tools.** Don't wrap every API endpoint. A `schedule_event` tool that finds availability and books a meeting beats separate `list_users`, `list_events` and `create_event` tools, and `search_contacts` beats `list_contacts`. "More tools don't always lead to better outcomes" [4].
- **Namespace related tools**, for example `asana_search` and `jira_search`, so the boundaries are clear when many servers are connected [4].
- **Return meaningful context.** Prefer readable names over opaque IDs, and consider a `response_format` parameter that lets the model ask for concise or detailed output [4].
- **Be token-efficient.** Paginate, filter and truncate with sensible defaults. Claude Code limited tool responses to 25,000 tokens by default at the time of the post (Sept 2025) [4].
- **Evaluate, don't guess.** Build realistic multi-step tasks, run them through an agent, read the transcripts, and refine the tools. Keep held-out tasks to avoid overfitting [4] (see topic 18).

### Errors the model can act on

The specification separates two kinds of error [3]:

| | Protocol error | Tool execution error |
|---|---|---|
| Cause | Unknown tool, malformed request, server fault | API failure, bad input, business-rule violation |
| Returned as | JSON-RPC `error` | A normal result with `isError: true` |
| Shown to the model? | MAY be | SHOULD be, "to enable self-correction" |

An error such as "Invalid departure date: must be in the future. Current date is 08/08/2025" lets the model retry with a fixed argument [3]. In Python SDK 2.x, only the message of a raised `ToolError` reaches the model. Other exceptions become a generic "Error executing tool" message, with details in the server log [2]. Write error messages for the model.

### State and security

The 2026-07-28 spec has no protocol-level session. A server that needs state across calls, such as a shopping basket or a browser session, should return an explicit **handle** from one tool and accept it as an argument on the next. It should also state in the description how long the handle lives [3].

Servers **MUST** validate all inputs, implement access controls, rate-limit calls and sanitise outputs. Clients SHOULD confirm sensitive operations and show tool inputs to the user before calling, to avoid data exfiltration [3]. A tool runs with whatever access you give it (see topic 13).

## Let an agent write the server

You don't need to memorise SDK syntax, but the agent needs **current, focused documentation**. SDK APIs move: an agent trained on 2025 code will happily write `from mcp.server.fastmcp import FastMCP`, which fails on Python SDK 2.x [2]. Practical options:

- Point the agent at the specific pages it needs: the build-server quickstart, the SDK docs and the tools section of the spec. Each MCP docs page points to an `llms.txt` index "to discover all available pages", which an agent can use to find the right one [1].
- The MCP docs describe a set of **agent skills** (the `mcp-server-dev` plugin) that guide a coding assistant through server design. They ask what the server connects to, who will use it, how many operations it needs, and what auth applies, then recommend a deployment path: remote Streamable HTTP, a packaged local bundle (MCPB), or local stdio for prototyping [6].
- A common pattern is to give the agent the specific doc page or README rather than a whole repository. The relevant material stays in focus and the context stays small.

The same pattern covers most real tools. Pure logic (the converter), a **local library** and a **remote HTTP API** (the quickstart's weather server calls the US National Weather Service [1]) all become "inputs in, result out, with a description". A sensible habit when wrapping a library or API: run its own example on its own first, then wrap one function as a tool.

## What lasts vs. what will date

- **Durable principles:**
  - A tool is a function plus a name, a description and an input schema. The model chooses by reading them.
  - Descriptions and parameter names are prompts. Write them for a reader who knows nothing else, and test changes.
  - Fewer, workflow-shaped tools beat thin wrappers around every endpoint.
  - Return errors the model can act on. Keep stdout clean on stdio servers.
  - Test with an inspector before a real client. Treat every tool as code with real permissions.
- **Time-sensitive details:**
  - Python SDK 2.x names (`MCPServer`, `mcp.server`), the `mcp dev` CLI, and SDK tiers (as of 2026-10).
  - Spec revision 2026-07-28 details: statelessness, handles, `resultType`, tool-name rules.
  - Inspector requirements and flags; Claude Desktop config paths.
  - The 25,000-token default in Claude Code (Sept 2025).

## Try it yourself

1. **Build the one-tool server** above, open it in the Inspector, and call it with valid and invalid input. Read the raw `tools/list` output.
2. **Break the description.** Rename the tool `tool1` with the docstring "does stuff", connect it to a client, and ask the same question. Then restore it.
3. **Add a near-duplicate** (`convert_usd_to_eur`) and check that the model picks correctly from descriptions alone.
4. **Make a good error.** Reject negative amounts with a `ToolError` that says what's wrong and what is allowed. Watch the model recover.
5. **Docs as context.** Ask an agent to add a tool once with no docs and once with the quickstart and SDK docs. Check which version imports correctly on SDK 2.x.

## Key terms

- **MCP server**: a program that exposes tools (and optionally resources and prompts) over the Model Context Protocol.
- **MCP client / host**: the application that connects to servers and lets the model call their tools.
- **Tool**: a callable function identified by a `name`, with a `description`, an `inputSchema` and optionally an `outputSchema`.
- **Input schema**: a JSON Schema describing a tool's parameters. Python SDKs derive it from type hints.
- **stdio transport**: running a server as a local process that speaks JSON-RPC over standard input/output.
- **Tool execution error**: a result marked `isError: true` that tells the model what went wrong so it can retry.
- **MCP Inspector**: the reference tool for listing and calling a server's tools while you develop.
- **Handle**: an explicit ID returned by one tool and passed to later calls, used instead of hidden session state.

## Sources

1. [Build an MCP server](https://modelcontextprotocol.io/docs/develop/build-server) (MCP documentation, accessed 2026-10-14). The Python quickstart: `uv` setup, `MCPServer`, type hints and docstrings as definitions, the stdout warning, Claude Desktop configuration, and what happens on each request.
2. [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) with its [What's new in v2](https://py.sdk.modelcontextprotocol.io/whats-new/) notes, plus the [SDK list](https://modelcontextprotocol.io/docs/sdk) (Model Context Protocol, accessed 2026-10-14). Installation, the `FastMCP`→`MCPServer` rename, `mcp dev`, `ToolError` behaviour, and SDK tiers.
3. [MCP Specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) (Model Context Protocol, 2026-07-28). Tool definition fields, name rules, `tools/list` and `tools/call`, output schemas, the two error types, stateful handles, and security requirements.
4. [Writing effective tools for agents — with agents](https://www.anthropic.com/engineering/writing-tools-for-agents) (Ken Aizawa, Anthropic, 2025-09-11). Tool-design principles: descriptions, parameter names, consolidation, namespacing, meaningful and token-efficient responses, and evaluation-driven iteration.
5. [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) (MCP documentation, accessed 2026-10-14). How to launch the Inspector against a local or remote server, and its web, CLI and TUI modes.
6. [Build with Agent Skills](https://modelcontextprotocol.io/docs/2026-07-28/develop/build-with-agent-skills) (MCP documentation, accessed 2026-10-14). The `mcp-server-dev` skills that guide a coding agent through server design and deployment choices.
