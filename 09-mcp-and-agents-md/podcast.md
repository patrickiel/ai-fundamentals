<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 09: MCP and AGENTS.md

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm still the one learning here.

BEN: And I'm Ben. Today is chapter nine, and the start of a new track: tools, protocols and capabilities. The chapter is about standards for agents: {MCP|M C P} for tools, and {AGENTS.md|AGENTS dot M D} for project context.

MAYA: Last time, in chapter eight, we did spec-driven development. Write down what to build, plan it, split it into tasks, then implement. A lot of it was about getting things written down in the right place.

BEN: And today is about agreeing where those things live, across tools. Here's the one sentence. {MCP|M C P} gave AI applications one shared way to connect to tools and data. And {AGENTS.md|AGENTS dot M D} applies the same move to project context: one plain Markdown file that any participating coding agent reads, instead of a separate rules file for every tool.

MAYA: Let's start with {MCP|M C P}. I hear about it constantly. What is it, actually?

BEN: The Model Context Protocol. Anthropic released it on November twenty-fifth, 2024, as, quote, "a new standard for connecting AI assistants to the systems where data lives."

MAYA: And what problem did it solve?

BEN: The announcement names it directly. Quote: "Every new data source requires its own custom implementation, making truly connected systems difficult to scale."

MAYA: What shipped at launch?

BEN: The specification and SDKs. Support for local {MCP|M C P} servers in the Claude Desktop apps. And an open-source repository of ready-made servers for systems like Google Drive, Slack, GitHub, Git and Postgres.

MAYA: [curious] Okay, so this is what taught models to use tools?

BEN: That's the most common misconception, and the chapter is clear about it. {MCP|M C P} is a convention, not a new model ability. Function calling already existed. Models could already call tools. What {MCP|M C P} added was an agreement on how the two sides talk.

MAYA: Hmm. So why does an agreement matter so much?

BEN: Picture it without one. App A needs custom code for tool one, and more custom code for tool two. App B needs its own custom code for tool one, and again for tool two. One integration for every pair. With {MCP|M C P}, every app speaks {MCP|M C P} once, every server speaks it once, and any app can talk to any server.

MAYA: So instead of apps times tools, it's apps plus tools.

BEN: That's a nice way to put it. Each side implements the protocol once. The {MCP|M C P} docs have an analogy: "Think of {MCP|M C P} like a {USB-C|U S B C} port for AI applications."

MAYA: [laughs] One plug for everything.

BEN: Right. And the specification makes a more technical comparison. {MCP|M C P} "takes some inspiration from the Language Server Protocol". That's the protocol that standardised how editors support programming languages. {MCP|M C P} does the same for "context and tools" in AI applications.

MAYA: So what are the moving parts?

BEN: As of October 2026, the current specification revision is from July twenty-eighth, 2026. It defines three roles, which exchange {JSON-RPC 2.0|JSON R P C two point oh} messages. The host is the LLM application that starts connections, like a chat app, an IDE or a coding agent. The client is a connector inside the host, one per server. And the server is a service that offers context and capabilities, like a server exposing your issue tracker.

MAYA: Wait, host and client sound like the same thing.

BEN: The host is the whole application. Inside it, there's one client for each server it connects to. So your coding agent is the host, and its link to your GitHub server is one client.

MAYA: Got it. And what can a server offer?

BEN: Three things. Tools, which are functions the model can call. Resources, which are data for the user or the model. And prompts, which are templated messages and workflows. Chapter ten builds a server and goes into tools in detail.

MAYA: Is the protocol stable now?

BEN: It keeps changing. That July 2026 revision made {MCP|M C P} stateless. It removed the initialize handshake and protocol-level sessions. It moved long-running tasks into an optional extension. And it deprecated the roots, sampling and logging features. So if you read an older tutorial, expect differences.

MAYA: Good to know. And security? A plug that connects to anything sounds a bit scary.

BEN: It's part of the standard. The specification says {MCP|M C P} "enables powerful capabilities through arbitrary data access and code execution paths". It asks hosts to get explicit user consent before exposing data or invoking tools. And to treat tool descriptions and annotations as untrusted, unless they come from a trusted server.

MAYA: So the plug makes it easy to connect things I haven't checked.

BEN: Exactly. A standard plug makes it easy to connect many servers, including ones you haven't reviewed. Chapter thirteen covers what can go wrong when untrusted content and tool access meet.

MAYA: Is {MCP|M C P} still Anthropic's protocol?

BEN: Not on its own anymore. On December ninth, 2025, the Linux Foundation announced the Agentic AI Foundation. Its founding projects were Anthropic's {MCP|M C P}, Block's goose, and OpenAI's {AGENTS.md|AGENTS dot M D}. Platinum members include AWS, Anthropic, Block, Bloomberg, Cloudflare, Google, Microsoft and OpenAI.

MAYA: That's quite a list of competitors in one room.

BEN: [laughs] It is. Anthropic's post on the donation reported more than ten thousand active public {MCP|M C P} servers, and more than ninety-seven million monthly SDK downloads across Python and TypeScript. And adoption by ChatGPT, Cursor, Gemini, Microsoft Copilot and VS Code. It also said {MCP|M C P}'s governance model "will remain unchanged". Those figures are from December 2025.

MAYA: Okay, let me summarise {MCP|M C P}. It's an agreement, not a new ability. Hosts, clients and servers. Servers offer tools, resources and prompts. Connecting a server is a trust decision. And it's now under a neutral foundation.

BEN: [warm] That's exactly it.

MAYA: So now the second half. Why do agents need a context file?

BEN: Because each session starts with an empty context window. Claude Code's docs state it plainly: "Each Claude Code session begins with a fresh context window". A project instruction file is how knowledge carries across sessions. Coding agents load it automatically at startup. It holds what you'd otherwise repeat every time: build and test commands, conventions, project layout, "always do X" rules.

MAYA: That's the context engineering idea from chapter three.

BEN: Right, it's a basic piece of it. And here's the problem. Each agent shipped with its own file name and format. {CLAUDE.md|CLAUDE dot M D} for Claude Code, {GEMINI.md|GEMINI dot M D} by default for Gemini CLI, and rules files in other tools.

MAYA: So if I use two agents on one repository, I keep two files.

BEN: Or more. Several files that should say the same thing.

MAYA: [sighs] That's annoying, but I can copy and paste.

BEN: That's the thing. Duplication is the small problem. Drift is the big one. Once agents start updating their own files, each tool writes only to its own file. A decision recorded by one agent never reaches the others. Nobody remembers to sync the copies by hand. And each agent starts its next session from a different, partial view of the project.

MAYA: [thoughtful] Oh. So the files don't just duplicate, they slowly disagree.

BEN: Exactly. And it's a general rule, not something specific to AI. [slowly] Several copies of the same state, each with its own writer, will diverge. The fix is one source of truth.

MAYA: And that's {AGENTS.md|AGENTS dot M D}.

BEN: That's {AGENTS.md|AGENTS dot M D}. Its site describes the format as a, quote, "README for agents: a dedicated, predictable place to provide the context and instructions to help AI coding agents work on your project." READMEs stay focused on people. {AGENTS.md|AGENTS dot M D} holds the extra detail agents need.

MAYA: What are the rules of the format?

BEN: Deliberately minimal. Plain Markdown, no required fields, use whatever headings help. Nested files for monorepos: agents read the nearest {AGENTS.md|AGENTS dot M D} in the directory tree. The site notes that OpenAI's main repository has eighty-eight of them.

MAYA: [laughs] Eighty-eight! And if two of them disagree?

BEN: There's a precedence rule. Quote: "The closest {AGENTS.md|AGENTS dot M D} to the edited file wins; explicit user chat prompts override everything."

MAYA: Okay. What else?

BEN: Commands get run. If you list test commands, agents try to run the relevant checks and fix failures before they finish. And migration is simple: rename your existing file to {AGENTS.md|AGENTS dot M D}, and leave a symbolic link under the old name.

MAYA: What does a typical one look like?

BEN: The chapter has a minimal example in the style of the site's samples. A setup commands section: install dependencies, start the dev server, run the tests, all with {pnpm|P N P M}. A code style section: TypeScript strict mode, single quotes, no semicolons. And pull request instructions: run lint and tests before committing.

MAYA: That's it? It's so simple.

BEN: And that's the point. What makes it work isn't the content. It's the agreement. Every participating agent looks in the same place, so you write the file once, and every tool reads and updates the same copy.

MAYA: Who actually reads it?

BEN: As of October 2026, the site lists twenty-three compatible agents and tools. The format came out of a collaboration between OpenAI Codex, Amp, Google's Jules, Cursor and Factory. And the list includes Codex, Jules, Gemini CLI, Cursor, GitHub Copilot's coding agent, VS Code, Zed, Warp, Aider, goose, opencode, Devin, Windsurf, Junie, Amp, RooCode and Augment Code. The site reports use by more than sixty thousand open-source projects.

MAYA: Does it just work in all of them?

BEN: Some need one setting. Aider and Gemini CLI, for example, each need a line in their own config file pointing at {AGENTS.md|AGENTS dot M D}.

MAYA: And Claude Code? We said in chapter six it reads {AGENTS.md|AGENTS dot M D} too.

BEN: It does. Claude Code keeps its own {CLAUDE.md|CLAUDE dot M D}. But its documentation says it "can read {AGENTS.md|AGENTS dot M D} as your project instructions, so a repository already set up for other coding agents works without adding a {CLAUDE.md|CLAUDE dot M D}, an import, or a setting." That's from version {2.1.277|two point one point two seven seven} onward.

MAYA: What if I have both files?

BEN: The defaults are worth knowing. If the repository has {AGENTS.md|AGENTS dot M D}, and no {CLAUDE.md|CLAUDE dot M D} or {CLAUDE.local.md|CLAUDE dot local dot M D}, Claude Code reads {AGENTS.md|AGENTS dot M D}. If it has both {AGENTS.md|AGENTS dot M D} and a {CLAUDE.md|CLAUDE dot M D}, it reads only {CLAUDE.md|CLAUDE dot M D}. And if your {CLAUDE.md|CLAUDE dot M D} imports {@AGENTS.md|at AGENTS dot M D}, it reads {CLAUDE.md|CLAUDE dot M D} with {AGENTS.md|AGENTS dot M D} pulled in.

MAYA: [thoughtful] Okay, so having both quietly hides {AGENTS.md|AGENTS dot M D}, unless I import it.

BEN: Right. A project instructions setting changes the default, for example to read both. And there's one trap. If you add a personal {CLAUDE.local.md|CLAUDE dot local dot M D} to a repository that relies on {AGENTS.md|AGENTS dot M D}, Claude Code stops reading {AGENTS.md|AGENTS dot M D}, unless you change that setting.

MAYA: Ouch. That's a sneaky one. So does {AGENTS.md|AGENTS dot M D} solve everything?

BEN: No. It covers instructions only. {MCP|M C P} server configuration is still set per client. Claude Code keeps project servers in {.mcp.json|dot M C P dot JSON}, under an {mcpServers|M C P servers} key, and personal ones in {~/.claude.json|dot claude dot JSON} in your home folder. VS Code has its own {mcp.json|M C P dot JSON} inside the {.vscode|dot vscode} folder, with a key called servers.

MAYA: Of course it does.

BEN: [laughs] But VS Code now also reads a portable {.mcp.json|dot M C P dot JSON} with {mcpServers|M C P servers}, and recommends the portable formats for new servers. That's convergence by imitation, though, not a formal standard. So check each client's docs.

MAYA: Okay, step back. What's the bigger lesson?

BEN: The chapter lines up three layers. Tool access was fragmented, with custom glue for every app and tool pair. The shared convention is {MCP|M C P}. Project instructions were one rules file per agent, drifting apart. The convention is {AGENTS.md|AGENTS dot M D}. And tool configuration is one config format per client, only partly converging on {.mcp.json|dot M C P dot JSON}.

MAYA: And in each case...

BEN: The value comes from agreement, not new technology. When many producers and consumers need the same thing, a simple shared format beats a better proprietary one. And when state lives in several copies with different writers, keep one source of truth.

MAYA: What lasts, and what will date?

BEN: The durable part. A shared protocol turns many-to-many integration into "implement once" on each side. That's {MCP|M C P}'s real contribution. Agents start each session blank, so project context must be written down and loaded automatically. Keep one source of truth for shared state, because copies with different writers drift. A convention only helps where your tools honour it, so check adoption before relying on it. And connecting a tool is a trust decision. Consent and review still apply.

MAYA: And what will date?

BEN: The spec revision, and its stateless design, deprecated features and extensions. The adoption figures: ten thousand servers and ninety-seven million monthly downloads as of December 2025, and sixty thousand projects for {AGENTS.md|AGENTS dot M D}. The list of agents that read {AGENTS.md|AGENTS dot M D}, and Claude Code's version requirement and loading rules. And the {MCP|M C P} config file names, keys and locations for each client.

MAYA: Let me try to sum up, and you correct me. {MCP|M C P} didn't give models a new ability. It's an agreement that turns one integration per pair into one implementation per side. Hosts contain clients, clients connect to servers, and servers offer tools, resources and prompts.

BEN: Good so far.

MAYA: Agents start every session blank, so they need a context file. But every tool invented its own, and copies with separate writers drift. {AGENTS.md|AGENTS dot M D} is one plain Markdown file for all agents, nearest file wins, chat prompts override. Claude Code reads it when there's no {CLAUDE.md|CLAUDE dot M D}. And {MCP|M C P} server config is still per client.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. A colleague says {MCP|M C P} is what finally taught models to use tools. What do you tell them?

PAUSE: 5

MAYA: That function calling already existed. {MCP|M C P} is a convention for how apps discover and call tools and data, so each side implements it once instead of writing glue code for every pair.

BEN: Perfect. Question two. You use two coding agents on one repository, and each keeps its own rules file. What goes wrong over time, and what's the fix?

PAUSE: 5

MAYA: Drift. Each agent writes only to its own file, so decisions don't reach the other one, and the copies diverge. The fix is one source of truth, one {AGENTS.md|AGENTS dot M D} that both read and update.

BEN: Exactly. And last one. Your repository relies on {AGENTS.md|AGENTS dot M D}, and you add a personal {CLAUDE.local.md|CLAUDE dot local dot M D}. What happens in Claude Code?

PAUSE: 5

MAYA: It stops reading {AGENTS.md|AGENTS dot M D}, unless I change the project instructions setting.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the inventory: list every agent-specific rules file in one repository, diff them, and note the drift.

BEN: Then consolidate them into one {AGENTS.md|AGENTS dot M D} and ask two agents how to run the tests. Check how Claude Code resolves it with {/context|slash context}, test whether two agents write back to the same file, and map where each client keeps its {MCP|M C P} config. Next time, in chapter ten, we look at how {MCP|M C P} servers work. Tools as described functions, and how to build and test a server.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter ten.
