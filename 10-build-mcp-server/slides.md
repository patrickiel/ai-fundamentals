---
theme: ../theme
chapter: '10'
header: 'AI Fundamentals · 10 · Building an MCP server'
layout: cover
---

# How MCP Servers Work

**A tool is a function plus a name, a description and a typed input schema that the model reads.**

---

## The problem

- Getting a server to run is easy; getting a model to **use it well** is not
- The model chooses tools only from their **metadata**
- SDK syntax changes between versions
- So: understand the concept, write good descriptions, feed agents **current docs**

<!-- This topic builds a one-tool server, then focuses on tool design, errors and testing. -->

---

## The mental model

```
question → model reads tool definitions (tools/list)
         → picks a tool, fills arguments
         → client runs it via the server (tools/call)
         → result → model writes the answer
```

- Tools are **model-controlled**
- A human SHOULD be able to deny any call

<!-- Steps follow the MCP quickstart's "under the hood" list and the spec's tools section. -->

---

## A minimal server (Python SDK 2.x)

```python
from mcp.server import MCPServer
mcp = MCPServer("currency")

@mcp.tool()
def convert_usd_to_gbp(amount: float) -> float:
    """Convert US dollars to British pounds.
    Uses a fixed illustrative rate, not live data."""
    return round(amount * 0.79, 2)

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

`uv add "mcp[cli]"` · v2 renamed `FastMCP` → `MCPServer` (as of 2026-10)

<!-- The old mcp.server.fastmcp import is gone, not deprecated. Most older tutorials use it. -->

---

## Anatomy of a tool

| Part | In code | What the model gets |
|---|---|---|
| **name** | function name | a handle that says what it does |
| **description** | docstring | what it does, when to use it |
| **inputSchema** | type hints → JSON Schema | arguments and types |
| **outputSchema** | return type (optional) | shape of structured results |

Names: 1–128 chars, `A-Z a-z 0-9 _ - .`, unique per server

<!-- The SDK generates the definition from the signature and docstring. Clients combining servers should prefix names to avoid collisions like two "search" tools. -->

---

## The stdout trap

- stdio servers speak JSON-RPC over **stdin/stdout**
- "**Never write to stdout.** Writing to stdout will corrupt the JSON-RPC messages"
- `print()` breaks the server; use `logging` (stderr)
- Most common cause of "connects but does nothing"

<!-- HTTP servers can log to stdout; stdio servers cannot. -->

---

## Test with the Inspector first

```bash
npx @modelcontextprotocol/inspector \
  uv --directory /ABS/PATH/currency run currency.py
```

- List tools, call them, see the raw JSON-RPC
- `--cli` for scripts and CI · `--tui` for the terminal
- Python shortcut: `uv run mcp dev currency.py`

<!-- Requires Node 22.19.0 or newer as of 2026-10. Test before connecting a real client. -->

---

## Register with a client

```json
{ "mcpServers": {
    "currency": {
      "command": "uv",
      "args": ["--directory", "/ABS/PATH/currency",
               "run", "currency.py"] } } }
```

- Claude Desktop: `claude_desktop_config.json`
- **Absolute paths**; full path to `uv` if needed; restart the app
- A local server = **a command the client launches**

<!-- Other clients keep the same information in other files; topic 09 compares them. -->

---

## Descriptions are prompts

- "Describe your tool to a **new hire** on your team"
- "Even small refinements to tool descriptions can yield **dramatic improvements**"
- `user_id`, not `user`
- Spell out formats and terms the model can't infer

<!-- Source: Anthropic, Writing effective tools for agents (2025-09-11). -->

---

## Fewer, better tools

- Don't wrap every endpoint: `schedule_event` > `list_users` + `list_events` + `create_event`
- `search_contacts` > `list_contacts`
- **Namespace**: `asana_search`, `jira_search`
- Return **readable** fields, offer concise/detailed formats
- Paginate, filter, truncate; every token costs context

<!-- "More tools don't always lead to better outcomes." Evaluate with realistic multi-step tasks and read transcripts (topic 18). -->

---

## Errors the model can act on

| | Protocol error | Tool execution error |
|---|---|---|
| Cause | unknown tool, malformed request | bad input, API failure |
| Returned as | JSON-RPC `error` | result with `isError: true` |
| To the model | MAY | **SHOULD**, for self-correction |

- SDK 2.x: only a `ToolError` message reaches the model

<!-- Example from the spec: "Invalid departure date: must be in the future." The model can retry with a fixed argument. -->

---

## State and security

- Spec 2026-07-28: **no protocol sessions**
  - return a **handle** (e.g. `basket_id`), accept it on later calls
  - state its lifetime in the description
- Servers MUST validate inputs, control access, rate-limit, sanitise outputs
- Clients SHOULD confirm sensitive operations and show inputs first

<!-- A tool runs with whatever access you give it. See topic 13. -->

---

## Let an agent write it

- Give it **current, focused docs**: quickstart, SDK docs, spec tools page
- Old training data writes `mcp.server.fastmcp`, which fails on 2.x
- MCP docs: `mcp-server-dev` **agent skills** ask about the target, users, auth, then pick a deployment path
- Same pattern for logic, a local library, or a remote API

<!-- Deployment paths in the skill: remote Streamable HTTP, MCPB bundle, or local stdio for prototyping. Run a library's own example first, then wrap one function. -->

---

## What lasts vs. what will date

**Lasts**
- Tool = function + name + description + input schema
- Descriptions and parameter names are prompts
- Fewer, workflow-shaped tools; actionable errors
- Test before connecting; tools carry real permissions

**Will date**
- `MCPServer` naming, `mcp dev`, SDK tiers
- Spec 2026-07-28 specifics; Inspector flags; config paths

<!-- Check the SDK changelog before copying code. The anatomy of a tool is the stable part. -->

---

## Try it yourself

1. Build the one-tool server; inspect `tools/list`; call it
2. Rename to `tool1` with "does stuff": does the model still pick it?
3. Add `convert_usd_to_eur`: does it choose correctly?
4. Reject negative amounts with a clear `ToolError`; watch recovery
5. Have an agent add a tool with vs. without current docs

<!-- Exercises 2 and 3 show how much the description matters. -->

---
layout: end
---

# Key takeaways

- An MCP tool is **a function the model reads about**
- **Name, description, schema and errors** decide whether it's used correctly
- **Test with the Inspector** and give agents **current docs**

modelcontextprotocol.io/docs/develop/build-server · anthropic.com/engineering/writing-tools-for-agents
