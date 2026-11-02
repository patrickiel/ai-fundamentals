<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 15: The A2A protocol

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter fifteen: the {A2A|A two A} protocol. A shared protocol for agents to delegate work to each other.

MAYA: Last time, in chapter fourteen, we talked about dynamic context discovery. Keep the always-loaded context small, and let the agent pull in what it needs. Today it's not about one agent's context, but about several agents working together?

BEN: Exactly. And here's the one sentence for today. MCP connects an agent to its own tools and data, while {A2A|A two A} connects one agent to other agents. Each agent publishes an Agent Card describing what it can do. Other agents can find it, and hand it tasks over standard web protocols, without seeing how it works inside.

MAYA: So MCP is agent to tool, and {A2A|A two A} is agent to agent.

BEN: That's the short version. {A2A|A two A} stands for {Agent2Agent|Agent to Agent}. It's an open protocol that Google announced in April 2025, with more than fifty partners.

MAYA: [curious] What problem was it trying to solve?

BEN: Agents tend to specialise. One sources job candidates, another runs background checks, another plans routes. Useful work often spans several of them. And they're built by different teams, on different frameworks, and run by different organisations. Without a shared protocol, each pair of agents needs custom integration code.

MAYA: Okay, but here's my question. We already have MCP. Why not just wrap the other agent as an MCP tool, and call it like a function?

BEN: That's the natural first thought. The {A2A|A two A} docs make a distinction that explains why tool protocols aren't enough. Tools are, quote, "primitives with well-defined, structured inputs and outputs". Agents are, quote, "more autonomous systems. They reason, plan, and use multiple tools". And they keep state across multi-turn exchanges.

MAYA: So in practice, what's the difference?

BEN: Asking an agent to do something works differently from calling a function. It may take hours. It may ask you follow-up questions. Or it may need you to log in.

MAYA: [thoughtful] Right. A function just returns. An agent might come back and say, "which city did you mean?"

BEN: Exactly. Google's announcement framed {A2A|A two A} as a way for agents, quote, "to communicate with each other, securely exchange information, and coordinate actions" across enterprise platforms.

MAYA: How is the protocol designed?

BEN: The version one point oh specification lists five guiding principles. One: simple. It reuses HTTP, {JSON-RPC 2.0|JSON R P C two point oh} and Server-Sent Events, rather than inventing new transports. Two: enterprise ready. Authentication, authorisation, security, privacy, tracing and monitoring that match existing practice.

MAYA: Three?

BEN: Async first. It's built for long-running tasks, and for interactions with a human in the loop. Four: modality agnostic. Text, audio and video via file references, structured data and forms, even embedded UI. And five: opaque execution. Agents share capabilities and results, not their internal state, memory or tools.

MAYA: Which one matters most?

BEN: The last one. A caller never needs to know whether the remote agent uses LangGraph, CrewAI or hand-written code, or which model it runs. It sees only the card and the protocol. In the spec's words, agents collaborate "without needing to share their internal thoughts, plans, or tool implementations".

MAYA: Okay, so let's talk about this card. What is an Agent Card?

BEN: The spec defines it as "a JSON metadata document published by an {A2A|A two A} Server". It describes the agent's identity, capabilities, skills, endpoint, and authentication requirements.

MAYA: What does one look like?

BEN: The chapter shows a shortened example, based on the spec's own sample. It's a "GeoSpatial Route Planner Agent", described as providing route planning, traffic analysis and custom map generation. It has a version number. It lists one interface: a web address, using the JSON-RPC binding, protocol version one point oh. Under capabilities, it says it supports streaming and push notifications.

MAYA: And what it accepts and returns?

BEN: It accepts JSON and plain text, and produces JSON and PNG images. And then there's a list of skills. Here just one, the "Traffic-Aware Route Optimizer", which finds optimal routes using real-time traffic, tagged "routing".

MAYA: So the skills list is the part that says what it's actually good at.

BEN: Right. Each skill has an ID, a name, a description, tags and examples. And the card can also list optional capabilities like an extended card, plus security schemes.

MAYA: [curious] Why does the card matter so much?

BEN: Because a delegating agent decides whether and how to use a remote agent from its card. If the skills are vaguely described, the agent delegates to the wrong place. Or it doesn't delegate when it should.

MAYA: That sounds just like tool descriptions in chapter ten.

BEN: It's the same lesson. Write a card the way you'd write a tool description: precise, specific, and honest about limits. Self-description drives delegation.

MAYA: Can I trust a card? What if someone publishes a fake one?

BEN: Good instinct. In version one point oh, cards may be signed with JSON Web Signatures, and clients should verify at least one signature before trusting a card. And sensitive details belong in an authenticated extended card, not in the public one.

MAYA: How does one agent find another in the first place?

BEN: The discovery guide lists three strategies. The first is a well-known address. The card sits at {/.well-known/agent-card.json|slash dot well-known slash agent card dot JSON} on the agent's domain. That's good for public or domain-controlled agents.

MAYA: Second?

BEN: Curated registries. Enterprise or marketplace catalogues that you can search by skill or tag. But note, quote, "the current {A2A|A two A} specification does not prescribe a standard API for curated registries."

MAYA: So every registry might work differently.

BEN: For now, yes. The third is direct configuration. Card addresses or contents are preconfigured, in config files or environment variables. Simple, but every change needs reconfiguration. And because cards change rarely, servers should send {Cache-Control|cache control} and {ETag|E tag} headers, and clients should make conditional requests.

MAYA: Okay. I've found an agent. How do I actually give it work?

BEN: You send a message. The spec calls it "a communication turn between a client and a remote agent". A message has a role, and one or more parts. In version one point oh, each part holds exactly one thing: text, raw inline bytes, a URL pointing to a file, or arbitrary JSON data.

MAYA: And what comes back?

BEN: Either a direct message, or a task. The task is the unit of work. It has an ID, an optional {contextId|context I D} that groups related work, a status, and output artifacts.

MAYA: Can you walk me through an example?

BEN: The spec has a basic one, over the HTTP plus JSON binding. The client posts a message with the user role, and a single text part: "What is the weather today?" The agent replies with a task. It has an ID and a context ID, its state is completed, and it carries one artifact named "Weather Report". That artifact's text part says, "Today will be sunny", and so on.

MAYA: So the answer comes back as an artifact attached to the task.

BEN: Right. And tasks have a lifecycle. They start as submitted, then working. There are two interrupted states: input required, and auth required. And there are terminal states: completed, failed, canceled and rejected.

MAYA: [thoughtful] The interrupted ones are the interesting part, I think.

BEN: They are. That's how {A2A|A two A} supports the "async first" principle. A remote agent can pause and ask for more input, or for credentials, rather than fail.

MAYA: Like the "which city did you mean?" case.

BEN: Exactly that. The core operations include {SendMessage|Send Message}, {SendStreamingMessage|Send Streaming Message}, which sends updates over Server-Sent Events, {GetTask|Get Task}, {ListTasks|List Tasks}, {CancelTask|Cancel Task} and {SubscribeToTask|Subscribe To Task}. Plus push-notification configuration for long jobs. And they're available over three standard bindings: JSON-RPC, {gRPC|G R P C}, and HTTP plus JSON, also called REST.

MAYA: Can a remote agent pass work on again?

BEN: Yes. A remote agent can itself act as a client, and delegate part of its task to another agent. That gives you a multi-agent workflow in which no agent needs to know how to do everything. Only how to read cards and hand off work. Chapter twenty-three discusses when splitting work across agents pays off.

MAYA: Is there a real-world-style example?

BEN: Google's launch post used hiring. A hiring manager asks their agent, inside a unified interface, which was Google Agentspace in the example, to find candidates matching a job description. That agent works with specialised remote agents to source candidates. Later it helps schedule interviews. And after the interviews, another agent runs background checks.

MAYA: So it's really about hiring?

BEN: [laughs] The point isn't hiring. A single request was handled by several specialised agents, coordinated through one protocol.

MAYA: Okay. Let's do the comparison properly. MCP versus {A2A|A two A}.

BEN: They're different layers, used together. MCP connects an agent to tools and data. {A2A|A two A} connects an agent to other agents. With MCP, the other side is a capability with structured inputs and outputs. With {A2A|A two A}, it's an autonomous agent that reasons, plans and keeps state.

MAYA: And how they describe themselves?

BEN: An MCP tool has a name, a description and an input schema. An {A2A|A two A} agent has its Agent Card: skills, interfaces, media types and auth. And the typical interaction differs. MCP is "call this function with these arguments". {A2A|A two A} is "here is a task, work on it and report back".

MAYA: Is there an example that shows both together?

BEN: The {A2A|A two A} docs use an auto repair shop. A customer, or the customer's assistant agent, talks to a "Shop Manager" agent over {A2A|A two A}. The Mechanic agent uses MCP to drive its diagnostic scanners and lifts. And when it needs a part, it asks a supplier's agent, over {A2A|A two A}: "Do you have part {#12345|number one two three four five} in stock for a Toyota Camry {2018|twenty eighteen}?"

MAYA: [laughs] So the mechanic uses MCP for its tools, and {A2A|A two A} to talk to the supplier.

BEN: That's it. MCP gives each agent its tools, and {A2A|A two A} lets the agents work together. As the docs put it: "{A2A|A two A} connects the agents to each other; MCP connects each agent to its own tools".

MAYA: Do the two ever overlap?

BEN: They can meet. The docs note that an {A2A|A two A} server "can expose some skills as MCP-compatible resources". That works best for well-defined, stateless, tool-like skills. But the docs say {A2A|A two A}'s "main strength is its support for flexible, stateful, collaborative interactions".

MAYA: So, my "just wrap the agent as a tool" idea works for the simple, tool-like parts, but not for the rest.

BEN: That's a fair way to put it. The Agentic AI Foundation's announcement calls MCP the "vertical integration layer", and {A2A|A two A} the "horizontal protocol enabling peer-to-peer collaboration". And the {A2A|A two A} docs' own advice is short: "Use both together".

MAYA: Who actually runs {A2A|A two A} now? Is it still a Google thing?

BEN: Not anymore. Here's the timeline, as of October 2026. Google announced it on April ninth, 2025, with more than fifty technology partners. On June twenty-third, 2025, the Linux Foundation launched the {A2A|A two A} project, with support from more than a hundred companies, and the aim of keeping it vendor-neutral.

MAYA: And then?

BEN: On March twelfth, 2026, version one point oh shipped. The announcement called it "the first stable, production-ready version". It added multiple protocol bindings, version negotiation, multi-tenancy, which means many agents behind one endpoint, and signed Agent Cards. Its technical steering committee includes AWS, Cisco, Google, IBM Research, Microsoft, Salesforce, SAP and ServiceNow.

MAYA: And the foundation you mentioned?

BEN: On August twenty-seventh, 2026, {A2A|A two A} joined the Agentic AI Foundation as a Growth Stage project, alongside MCP. The project reports backing from more than a hundred and fifty organisations. And native support in Google Cloud, AWS Bedrock AgentCore and Microsoft Azure AI Foundry, plus frameworks such as LangGraph, CrewAI and Pydantic AI.

MAYA: If I find an {A2A|A two A} tutorial online, will it match all this?

BEN: Maybe not. Version one point oh changed a lot of names. Slash-style methods, like {message/send|message slash send} and {tasks/get|tasks slash get}, became {SendMessage|Send Message} and {GetTask|Get Task}. Enum values got prefixes, like {TASK_STATE_COMPLETED|task state completed} and {ROLE_USER|role user}. The separate text, file and data part types merged into one part type. And the card's single URL became a list of supported interfaces.

MAYA: So older tutorials will show the old names.

BEN: Tutorials from 2025 will, yes. But a card can advertise support for version zero point three and version one point oh at the same time, so clients can migrate gradually.

MAYA: Before we wrap up, what part of this will still be true in a few years, and what will date?

BEN: The durable part first. Agent-to-tool integration and agent-to-agent collaboration are different problems, with different interfaces and trust needs. Self-description drives delegation. Cards and tool descriptions are read by models, so write them precisely. Opaque execution: share capabilities and results, not internals.

MAYA: Go on.

BEN: Long-running work needs explicit task states, including "waiting for input" and "waiting for auth". And treat a remote agent's card as a claim to verify, not as a fact. That means signatures, TLS and authentication.

MAYA: And what will date?

BEN: The version one point oh field and method names, the bindings, and the well-known path. The adoption figures and the list of platforms with native support, which are as of August 2026. The governance: the Linux Foundation project, the foundation's Growth Stage, and the steering-committee members. And the SDK languages and sample repositories.

MAYA: Let me try to sum up, and you correct me. {A2A|A two A} is an open protocol for agents to delegate work to other agents. MCP connects an agent to its tools, {A2A|A two A} connects agents to each other, and you use both. Each agent publishes an Agent Card with its skills, interfaces, media types and auth, usually at a well-known address.

BEN: Good so far.

MAYA: A client sends a message made of parts. It gets back a message or a task, with a lifecycle state and artifacts. Tasks can pause for input or auth, instead of failing. Agents never show their internals. And the card is what drives delegation, so it has to be precise, and I should verify it rather than just trust it.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your assistant needs to connect to two things: a currency-conversion API, and a travel-booking agent that plans trips and asks follow-up questions. Which protocol fits each, and what test did you use?

PAUSE: 5

MAYA: The conversion API is a tool, so MCP. It has well-defined, structured inputs and outputs. The travel agent is an agent, so {A2A|A two A}. It reasons, plans, keeps state across turns, and may come back with questions. That's my test: function call, or task that someone works on?

BEN: Perfect. Question two. Your delegating agent keeps handing tasks to the wrong remote agent. What do you fix first?

PAUSE: 5

MAYA: The Agent Cards, especially the skill descriptions. The delegating agent decides from the card, so vague skills lead to wrong hand-offs. Make them precise, specific, and honest about limits.

BEN: Exactly. And last one. Halfway through a task, the remote agent needs the user to log in. Does the task fail?

PAUSE: 5

MAYA: No. It moves into the auth-required state, one of the two interrupted states, and waits for credentials. That's the "async first" principle. Then it carries on.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has four exercises. I'm doing the first one. Write an Agent Card by hand for a narrow agent, like an expense-report checker. Then give it, with five sample requests, to a colleague or an LLM, ask which requests the agent should take, and fix the card wherever it misled them.

BEN: And try the others too. Classify ten integrations as tools or agents, and write down your test. Run the official Python sample: the hello-world agent listens on port nine nine nine nine, and you can fetch its card from the well-known path and read it. And compare a 2025 tutorial with the version one point oh spec, and list what changed.

MAYA: And what's next?

BEN: Next time, in chapter sixteen, we start the next track, building AI features, with embeddings and {RAG|rag}. The retrieval pipeline, hybrid search, how it fails, and when {RAG|rag} is the wrong tool.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter sixteen.
