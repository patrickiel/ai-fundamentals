# Agent-to-Agent (A2A): A Shared Protocol for Agents to Delegate Work to Each Other

> **Core idea in one sentence:** MCP connects an agent to its own tools and data, while A2A connects one agent to *other agents*: each agent publishes an Agent Card describing what it can do, so other agents can find it and hand it tasks over standard web protocols, without seeing how it works inside.

## TL;DR

- **A2A (Agent2Agent)** is an open protocol that Google announced in April 2025 with 50+ partners [1]. It moved to the Linux Foundation in June 2025 [6], reached a stable **v1.0** in March 2026 [5], and joined the **Agentic AI Foundation** in August 2026 [7].
- Each agent publishes an **Agent Card**, a JSON document describing its identity, skills, endpoints, supported content types and authentication. Public cards conventionally live at `/.well-known/agent-card.json` [2][4].
- Work is organised around **tasks**. A client sends a **message** made of **parts**. The remote agent returns a message or a task with a lifecycle state and output **artifacts** [2].
- **Opaque execution:** agents collaborate "without needing to share their internal thoughts, plans, or tool implementations" [2].
- **A2A doesn't replace MCP.** "A2A connects the agents to each other; MCP connects each agent to its own tools" [3].
- Self-description drives delegation. A vague Agent Card leads to wrong hand-offs, just as a vague tool description does in MCP (topic 10).

## The problem A2A addresses

Agents tend to specialise: one sources job candidates, another runs background checks, another plans routes. Useful work often spans several of them, built by different teams on different frameworks and run by different organisations. Without a shared protocol, each pair of agents needs custom integration code.

Google's announcement framed A2A as a way for agents "to communicate with each other, securely exchange information, and coordinate actions" across enterprise platforms [1]. The A2A docs make a distinction that explains why tool protocols aren't enough. Tools are "primitives with well-defined, structured inputs and outputs". Agents are "more autonomous systems. They reason, plan, and use multiple tools", and they keep state across multi-turn exchanges [3]. Asking an agent to do something works differently from calling a function. It may take hours, ask follow-up questions, or need you to log in.

## Design principles

The v1.0 specification lists five guiding principles [2]:

| Principle | What it means |
|---|---|
| **Simple** | Reuse HTTP, JSON-RPC 2.0 and Server-Sent Events rather than inventing transports |
| **Enterprise ready** | Authentication, authorisation, security, privacy, tracing and monitoring that match existing practice |
| **Async first** | Built for long-running tasks and human-in-the-loop interactions |
| **Modality agnostic** | Text, audio/video via file references, structured data and forms, even embedded UI |
| **Opaque execution** | Agents share capabilities and results, not their internal state, memory or tools |

The last principle matters most. A caller never needs to know whether the remote agent uses LangGraph, CrewAI or hand-written code, or which model it runs. It sees only the card and the protocol.

## How A2A works

### 1. The Agent Card: how an agent describes itself

The specification defines the Agent Card as "a JSON metadata document published by an A2A Server" that describes identity, capabilities, skills, endpoint and authentication requirements [2]. A shortened example in v1.0 form, based on the spec's own sample:

```json
{
  "name": "GeoSpatial Route Planner Agent",
  "description": "Provides route planning, traffic analysis, and custom map generation.",
  "version": "1.2.0",
  "supportedInterfaces": [
    { "url": "https://georoute-agent.example.com/a2a/v1",
      "protocolBinding": "JSONRPC", "protocolVersion": "1.0" }
  ],
  "capabilities": { "streaming": true, "pushNotifications": true },
  "defaultInputModes": ["application/json", "text/plain"],
  "defaultOutputModes": ["application/json", "image/png"],
  "skills": [
    { "id": "route-optimizer-traffic", "name": "Traffic-Aware Route Optimizer",
      "description": "Optimal routes using real-time traffic.", "tags": ["routing"] }
  ]
}
```

Key fields include the endpoints and protocol versions the agent supports, optional capabilities (streaming, push notifications, an extended card), accepted and produced media types, security schemes, and a list of **skills**, each with an ID, name, description, tags and examples [2].

**Why the card matters:** a delegating agent decides whether and how to use a remote agent from its card. If the skills are vaguely described, the agent delegates to the wrong place or doesn't delegate when it should. Write a card the way you would write a tool description: precise, specific and honest about limits.

In v1.0, cards **MAY** be signed with JSON Web Signatures, and clients **SHOULD** verify at least one signature before trusting a card [2]. Sensitive details belong in an **authenticated extended card**, not the public one [4].

### 2. Discovery: finding the right agent

The discovery guide lists three strategies [4]:

- **Well-known URI:** `https://{agent-server-domain}/.well-known/agent-card.json`. Good for public or domain-controlled agents.
- **Curated registries:** enterprise or marketplace catalogues searchable by skill or tag. Note that "the current A2A specification does not prescribe a standard API for curated registries."
- **Direct configuration:** card URLs or contents preconfigured in config files or environment variables. Simple, but every change needs reconfiguration.

Cards change rarely, so servers should send `Cache-Control` and `ETag` headers and clients should make conditional requests [4].

### 3. Messages, tasks and artifacts

Once a client has chosen an agent, it sends a **message**: "a communication turn between a client and a remote agent", with a role and one or more **parts** [2]. In v1.0 a part holds exactly one of `text`, `raw` (inline bytes), `url` (a file reference) or `data` (arbitrary JSON) [2]. The agent replies either with a direct message or with a **task**, the unit of work. A task has an ID, an optional `contextId` that groups related work, a status and output **artifacts** [2].

The spec's basic example over the HTTP+JSON binding [2]:

```
POST /message:send
{ "message": { "role": "ROLE_USER",
               "parts": [{ "text": "What is the weather today?" }],
               "messageId": "msg-uuid" } }

→ { "task": { "id": "task-uuid", "contextId": "context-uuid",
              "status": { "state": "TASK_STATE_COMPLETED" },
              "artifacts": [{ "artifactId": "artifact-uuid", "name": "Weather Report",
                              "parts": [{ "text": "Today will be sunny..." }] }] } }
```

Task states include `SUBMITTED` and `WORKING`, two interrupted states (`INPUT_REQUIRED`, `AUTH_REQUIRED`), and the terminal states `COMPLETED`, `FAILED`, `CANCELED` and `REJECTED` [2]. The interrupted states are how A2A supports the "async first" principle: a remote agent can pause and ask for more input or credentials rather than fail.

Core operations include `SendMessage`, `SendStreamingMessage` (updates over Server-Sent Events), `GetTask`, `ListTasks`, `CancelTask` and `SubscribeToTask`, plus push-notification configuration for long jobs. They are available over three standard bindings: JSON-RPC, gRPC and HTTP+JSON/REST [2].

### 4. Chaining: from one agent to a workflow

A remote agent can itself act as a client and delegate part of its task to another agent. That gives you a multi-agent workflow in which no agent needs to know how to do everything, only how to read cards and hand off work. Topic 23 discusses when splitting work across agents pays off.

## Google's example: a hiring pipeline

The launch post illustrated A2A with hiring [1]. A hiring manager asks their agent, inside a unified interface (Google Agentspace in the example), to find candidates matching a job description. That agent works with specialised remote agents to source candidates. Later it helps schedule interviews, and after the interviews another agent runs background checks. The point isn't hiring. A single request was handled by several specialised agents, coordinated through one protocol.

## A2A vs. MCP: different layers, used together

| | MCP | A2A |
|---|---|---|
| Connects | An agent to **tools and data** | An agent to **other agents** |
| The other side is | A capability with structured inputs and outputs | An autonomous agent that reasons, plans and keeps state |
| Self-description | Tool name, description, input schema | Agent Card: skills, interfaces, media types, auth |
| Typical interaction | "Call this function with these arguments" | "Here is a task; work on it and report back" |

The A2A docs use an **auto repair shop** to show the split [3]. A customer, or the customer's assistant agent, talks to a "Shop Manager" agent over A2A. The Mechanic agent uses MCP to drive its diagnostic scanners and lifts. When it needs a part, it asks a supplier's agent over A2A: "Do you have part #12345 in stock for a Toyota Camry 2018?" MCP gives each agent its tools; A2A lets the agents work together.

The two can meet. The docs note that an A2A server "can expose some skills as MCP-compatible resources", which works best for well-defined, stateless, tool-like skills. But the docs say A2A's "main strength is its support for flexible, stateful, collaborative interactions" [3]. The AAIF announcement calls MCP the "vertical integration layer" and A2A the "horizontal protocol enabling peer-to-peer collaboration" [7]. The A2A docs' own advice: "Use both together" [3].

## Governance and status (as of 2026-10)

- **2025-04-09:** Google announces A2A with 50+ technology partners [1].
- **2025-06-23:** the Linux Foundation launches the A2A project, with support from more than 100 companies and the aim of keeping it vendor-neutral [6].
- **2026-03-12:** v1.0, "the first stable, production-ready version", adds multiple protocol bindings, version negotiation, multi-tenancy (many agents behind one endpoint) and signed Agent Cards. A technical steering committee includes AWS, Cisco, Google, IBM Research, Microsoft, Salesforce, SAP and ServiceNow [5].
- **2026-08-27:** A2A joins the Agentic AI Foundation as a Growth Stage project, alongside MCP. The project reports backing from 150+ organisations and native support in Google Cloud, AWS Bedrock AgentCore and Microsoft Azure AI Foundry, plus frameworks such as LangGraph, CrewAI and Pydantic AI [7].

v1.0 changed a lot of names. Slash-style methods such as `message/send` and `tasks/get` became `SendMessage` and `GetTask`. Enum values became prefixed (`TASK_STATE_COMPLETED`, `ROLE_USER`). Separate text, file and data part types merged into one `Part`. The card's single `url` became a `supportedInterfaces` list [5]. A card can advertise v0.3 and v1.0 support at the same time, so clients can migrate gradually [5]. Tutorials from 2025 will show the old names.

## What lasts vs. what will date

- **Durable principles:**
  - Agent-to-tool integration and agent-to-agent collaboration are different problems with different interfaces and trust needs.
  - **Self-description drives delegation.** Cards and tool descriptions are read by models, so write them precisely.
  - Opaque execution: share capabilities and results, not internals.
  - Long-running work needs explicit task states, including "waiting for input" and "waiting for auth".
  - Treat a remote agent's card as a claim to verify (signatures, TLS, authentication), not as a fact.
- **Time-sensitive details:**
  - v1.0 field and method names, bindings, and the well-known path.
  - Adoption figures and the list of platforms with native support (as of 2026-08).
  - Governance: Linux Foundation project, AAIF Growth Stage, steering-committee members.
  - SDK languages and sample repositories.

## Try it yourself

1. **Write an Agent Card by hand** for a narrow agent, such as an expense-report checker: name, description, two or three skills with examples, media types, auth. Give it with five sample requests to a colleague or an LLM and ask which requests the agent should take. Fix the card wherever it misled them.
2. **Classify integrations.** List ten things an assistant might connect to (calendar, database, translation agent, code-review agent, weather API...). Mark each as a tool (MCP) or an agent (A2A) and write down the test you used.
3. **Run the official sample.** Following the Python tutorial: `git clone https://github.com/a2aproject/a2a-samples.git`, `pip install -r samples/python/requirements.txt`, then `python samples/python/agents/helloworld/__main__.py`. The server listens on port 9999. Fetch `http://127.0.0.1:9999/.well-known/agent-card.json` and read the card [8].
4. **Spot the version.** Compare the field and method names in a 2025 A2A tutorial with the v1.0 spec. List what changed.

## Key terms

- **A2A (Agent2Agent protocol)**: an open protocol for agents built on any framework to discover each other and exchange tasks over HTTP-based bindings (JSON-RPC, gRPC, HTTP+JSON).
- **Agent Card**: a JSON document an agent publishes describing its identity, skills, interfaces, media types and auth requirements.
- **Client agent / remote agent**: the agent that sends a task, and the agent that receives and performs it.
- **Message / Part**: one communication turn, made of parts that each hold text, raw bytes, a file URL or structured data.
- **Task**: the unit of work, with an ID, a context, a lifecycle state and output artifacts.
- **Artifact**: an output an agent produces for a task, made of parts.
- **Opaque execution**: collaborating through declared capabilities without exposing internal reasoning, memory or tools.
- **Well-known URI**: the standard path (`/.well-known/agent-card.json`) where a public Agent Card is served.

## Sources

1. [Announcing the Agent2Agent Protocol (A2A)](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/) (Rao Surapaneni, Miku Jha, Michael Vakoc, Todd Segal; Google, 2025-04-09). The launch: purpose, design principles, client/remote agents, Agent Cards, tasks and artifacts, the MCP relationship, and the hiring example.
2. [A2A Protocol Specification, v1.0](https://a2a-protocol.org/latest/specification/) (A2A Project, accessed 2026-10-14). Guiding principles, the Agent Card, messages, parts, tasks and states, operations, bindings, card signing, and the basic request/response example.
3. [A2A and MCP](https://a2a-protocol.org/latest/topics/a2a-and-mcp/) (A2A Project, accessed 2026-10-14). The tools-versus-agents distinction, the auto repair shop example, and how the two protocols combine.
4. [Agent Discovery in A2A](https://a2a-protocol.org/latest/topics/agent-discovery/) (A2A Project, accessed 2026-10-14). The well-known URI, registries and direct configuration, protecting sensitive cards, and caching.
5. [A2A Protocol Ships v1.0](https://a2a-protocol.org/latest/blog/2026/03/12/a2a-protocol-ships-v10-production-ready-standard-for-agent-to-agent-communication/) (A2A Project, 2026-03-12) and [What's new in v1.0](https://a2a-protocol.org/latest/whats-new-v1/). Stability status, new features, steering committee, renamed methods and fields, and v0.3 compatibility.
6. [Linux Foundation Launches the Agent2Agent Protocol Project](https://linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents) (Linux Foundation, 2025-06-23). The move to vendor-neutral governance and early industry support.
7. [A new chapter for A2A: joining the Agentic AI Foundation](https://a2a-protocol.org/latest/blog/2026/08/27/a-new-chapter-for-a2a-joining-the-agentic-ai-foundation/) (A2A Project, 2026-08-27). AAIF Growth Stage status, adoption figures, platform and framework support, and the vertical/horizontal framing with MCP.
8. [A2A Python tutorial](https://a2a-protocol.org/latest/tutorials/python/1-introduction/) (A2A Project, accessed 2026-10-14). Setting up the samples repository and running the Helloworld agent that serves its card at the well-known path.
