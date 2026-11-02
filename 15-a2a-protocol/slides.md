---
theme: ../theme
chapter: '15'
header: 'AI Fundamentals · 15 · A2A protocol'
layout: cover
---

# Agent-to-Agent (A2A)

**MCP connects an agent to its tools. A2A connects agents to each other, through published Agent Cards and tasks.**

---

## The problem

- Agents specialise: sourcing, background checks, routing...
- Real work spans several, built on **different frameworks** by **different organisations**
- Without a shared protocol, every pair needs **custom glue**
- Agents aren't tools: they **reason, plan, keep state**, take hours, ask questions

<!-- The A2A docs: tools are primitives with structured inputs and outputs; agents are autonomous systems. -->

---

## Design principles (spec v1.0)

| Principle | Meaning |
|---|---|
| Simple | HTTP, JSON-RPC 2.0, Server-Sent Events |
| Enterprise ready | auth, security, tracing like existing systems |
| Async first | long-running tasks, humans in the loop |
| Modality agnostic | text, files, structured data, forms |
| **Opaque execution** | share capabilities and results, **not internals** |

<!-- Opaque execution: the caller never needs to know which framework or model the remote agent uses. -->

---

## The Agent Card

```json
{ "name": "GeoSpatial Route Planner Agent",
  "description": "Route planning, traffic analysis, maps.",
  "version": "1.2.0",
  "supportedInterfaces": [{ "url": "https://georoute-agent.example.com/a2a/v1",
      "protocolBinding": "JSONRPC", "protocolVersion": "1.0" }],
  "capabilities": { "streaming": true, "pushNotifications": true },
  "defaultInputModes": ["application/json", "text/plain"],
  "skills": [{ "id": "route-optimizer-traffic",
               "name": "Traffic-Aware Route Optimizer" }] }
```

<!-- Shortened from the v1.0 spec's example. The card is what a delegating agent reads before choosing. -->

---

## Cards drive delegation

- A delegating agent decides **from the card alone**
- Vague skills → wrong hand-offs or none at all
- Same lesson as MCP tool descriptions (topic 10)
- v1.0: cards **MAY be signed** (JWS); clients **SHOULD verify** a signature
- Sensitive details go in an **authenticated extended card**

<!-- Treat a card as a claim to verify, not a fact. -->

---

## Discovery

- **Well-known URI:** `https://{domain}/.well-known/agent-card.json`
- **Curated registries:** search by skill or tag
  - no standard registry API in the spec yet
- **Direct configuration:** URLs in config or env vars
- Cache cards with `Cache-Control` + `ETag`

<!-- Source: A2A agent discovery guide. -->

---

## Messages, tasks, artifacts

```
POST /message:send
{ "message": { "role": "ROLE_USER", "messageId": "msg-uuid",
               "parts": [{ "text": "What is the weather today?" }] } }

→ { "task": { "id": "task-uuid", "contextId": "context-uuid",
     "status": { "state": "TASK_STATE_COMPLETED" },
     "artifacts": [{ "name": "Weather Report",
                     "parts": [{ "text": "Today will be sunny..." }] }] } }
```

Part = exactly one of `text` · `raw` · `url` · `data`

<!-- From the spec's basic task execution example over HTTP+JSON. The agent may reply with a message instead of a task. -->

---

## Task lifecycle and operations

- States: `SUBMITTED` → `WORKING` → `COMPLETED` / `FAILED` / `CANCELED` / `REJECTED`
- Interrupted: **`INPUT_REQUIRED`**, **`AUTH_REQUIRED`**
- Operations: `SendMessage`, `SendStreamingMessage`, `GetTask`, `ListTasks`, `CancelTask`, `SubscribeToTask`, push notifications
- Bindings: **JSON-RPC · gRPC · HTTP+JSON**

<!-- Interrupted states let a remote agent pause and ask for input or credentials instead of failing. -->

---

## Google's example: hiring

1. Manager asks their agent (in Agentspace) for candidates
2. It delegates to a **sourcing** agent
3. Later helps schedule interviews
4. A **background-check** agent vets the chosen candidate

**Point:** one request, several specialised agents, **one protocol**

<!-- From Google's launch post, 2025-04-09. A remote agent can itself delegate further, creating a chain. -->

---

## A2A vs. MCP

| | MCP | A2A |
|---|---|---|
| Connects | agent ↔ **tools / data** | agent ↔ **agents** |
| Other side | structured capability | autonomous, stateful agent |
| Self-description | tool name + schema | Agent Card |
| Interaction | "call this function" | "work on this task, report back" |

"A2A connects the agents to each other; MCP connects each agent to its own tools."

<!-- AAIF framing: MCP is the vertical integration layer, A2A the horizontal one. -->

---

## The repair-shop example

- Customer's agent talks to the **Shop Manager** agent: **A2A**
- **Mechanic** agent drives scanners and lifts: **MCP**
- Mechanic asks a **supplier** agent for part #12345: **A2A**
- Some well-defined skills can also be exposed as MCP resources

"Use both together."

<!-- Source: A2A docs, "A2A and MCP". A2A's main strength is flexible, stateful, collaborative interaction. -->

---

## Status (as of 2026-10)

- **2025-04:** Google announces A2A, 50+ partners
- **2025-06:** Linux Foundation project, 100+ companies
- **2026-03:** **v1.0** stable; multi-tenancy, signed cards, version negotiation
- **2026-08:** joins the **Agentic AI Foundation** with MCP; 150+ organisations
- Native in Google Cloud, AWS Bedrock AgentCore, Azure AI Foundry

<!-- Steering committee: AWS, Cisco, Google, IBM Research, Microsoft, Salesforce, SAP, ServiceNow. -->

---

## v1.0 renamed a lot

- `message/send` → `SendMessage`, `tasks/get` → `GetTask`
- Enums: `TASK_STATE_COMPLETED`, `ROLE_USER`
- Text/File/Data parts → one `Part`
- Card `url` → `supportedInterfaces[]`
- Cards can advertise **v0.3 and v1.0** at once

<!-- Tutorials from 2025 show the old names. Check the spec version before copying code. -->

---

## What lasts vs. what will date

**Lasts**
- Tool integration ≠ agent collaboration
- Self-descriptions drive delegation
- Opaque execution; explicit task states
- Verify cards; don't just trust them

**Will date**
- v1.0 names, bindings, well-known path
- Adoption figures, governance details, SDKs and samples

<!-- Learn the shape of the protocol; re-check the current spec for exact names. -->

---

## Try it yourself

1. Write an Agent Card for a narrow agent; test routing from the card alone
2. Classify 10 integrations as **tool (MCP)** or **agent (A2A)**
3. Clone `a2aproject/a2a-samples`, run the Helloworld agent (port 9999), fetch its `/.well-known/agent-card.json`
4. Compare a 2025 tutorial with the v1.0 spec

<!-- Exercise 1 shows how much delegation depends on description quality. -->

---
layout: end
---

# Key takeaways

- **MCP = agent ↔ tools. A2A = agent ↔ agents.** Use both.
- The **Agent Card** is the contract delegation depends on
- A2A is **stable (v1.0)** and under neutral governance, but names changed

a2a-protocol.org/latest/specification · a2a-protocol.org/latest/topics/a2a-and-mcp
