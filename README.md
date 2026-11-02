<img src="logo.svg" alt="AI Fundamentals logo" width="72">

# AI Fundamentals

A hype-free course on the durable principles of working with and building on large language models:
how to talk to models, how to engineer their context, the tools and protocols agents use, how to build
and evaluate AI features, and how agent systems are designed and kept safe.

Compiled October 2026 from primary sources: provider documentation, specifications, research papers,
original engineering posts, and established practitioner writing. Every chapter lists its sources.

## How to use this course

Start with the [introduction and Track 0](00-introduction/article.md), a short primer on how LLMs work.
The [introduction slides](00-introduction/slides.md) also cover why the course exists, how chapters are
built, the map of all tracks, and suggested reading paths.

Each chapter folder contains:

| File | What it is |
|---|---|
| `article.md` | The chapter: TL;DR, explanation, what lasts vs. what will date, exercises, key terms, sources |
| `slides.md` | The chapter as a [Slidev](https://sli.dev) slide deck |
| `podcast.md` | The script of the chapter's podcast episode: Maya is the learner, Ben the expert |

Work through the tracks in order. Track 0 gives the mental model, Tracks 1–3 cover what you need on day
one, Tracks 4–5 add tools and the basics of building AI features, Track 6 is advanced agent design,
and Track 7 can be read at any time.

Facts that change quickly (product names, versions, prices, flags) are marked "as of 2026-10" in the text.

---

## Track 0: How LLMs work

Integrated into the introduction: [read the primer](00-introduction/article.md) ·
[slides](00-introduction/slides.md). Covers tokens and attention, training vs. inference, the context
window, hallucinations, and how tools turn a model into part of an agent system.

The original Karpathy, 3Blue1Brown and Anthropic resources are listed as
[sources and optional deeper reading](00-introduction/article.md#sources-and-optional-deeper-reading).

## Track 1: Using models

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 01 | [Getting better LLM answers](01-better-llm-answers/article.md) · [slides](01-better-llm-answers/slides.md) | Clear instructions, real material to work from, managing the context window, and when to use thinking, projects, memory and tools. | [Anthropic prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) · [OpenAI prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering) |
| 02 | [Reasoning models](02-reasoning-models/article.md) · [slides](02-reasoning-models/slides.md) | Why reasoning models want short, goal-focused prompts, when they are worth the cost, and the planner/workhorse split. | [OpenAI reasoning best practices](https://developers.openai.com/api/docs/guides/reasoning-best-practices) · [OpenAI reasoning models](https://developers.openai.com/api/docs/guides/reasoning) |

## Track 2: Context engineering

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 03 | [Context engineering](03-context-engineering/article.md) · [slides](03-context-engineering/slides.md) | Designing the information an agent works from (requirements, plans, conventions, known issues) instead of crafting one clever prompt. | [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · [Tobi Lütke](https://x.com/tobi/status/1935533422589399127) · [Andrej Karpathy](https://x.com/karpathy/status/1937902205765607626) |
| 04 | [Context rot](04-context-rot/article.md) · [slides](04-context-rot/slides.md) | Why models use long inputs less reliably as they grow, and when to compact, clear, delegate or rewind. | [Chroma: Context Rot](https://www.trychroma.com/research/context-rot) · [Lost in the Middle](https://arxiv.org/abs/2307.03172) |
| 05 | [Compaction, notes and subagents](05-compaction-notes-subagents/article.md) · [slides](05-compaction-notes-subagents/slides.md) | The three techniques for work that outlasts one context window: compaction, structured notes, and sub-agents. | [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · [Managing context on the Claude Developer Platform](https://claude.com/resources/articles/context-management) |

## Track 3: Working with coding agents

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 06 | [Agent instruction files](06-coding-guidelines/article.md) · [slides](06-coding-guidelines/slides.md) | What belongs in `CLAUDE.md`/`AGENTS.md`: only what changes the agent's default behaviour, kept short, with hard rules enforced by permissions and hooks. | [Karpathy's notes on coding with agents](https://x.com/karpathy/status/2015883857489522876) · [Claude Code best practices](https://code.claude.com/docs/en/best-practices) |
| 07 | [Verification first](07-verification-first/article.md) · [slides](07-verification-first/slides.md) | The single biggest improvement: give the agent a check it can run (tests, build, browser, linter). | [Boris Cherny: How I use Claude Code](https://x.com/bcherny/status/2007179832300581177) · [Claude Code best practices](https://code.claude.com/docs/en/best-practices) |
| 08 | [Spec-driven development](08-spec-driven/article.md) · [slides](08-spec-driven/slides.md) | Spec → plan → tasks → implementation, so you and the agent agree on what to build before it builds it. | [GitHub Spec Kit](https://github.com/github/spec-kit) · [spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md) |

## Track 4: Tools, protocols and capabilities

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 09 | [MCP and AGENTS.md](09-mcp-and-agents-md/article.md) · [slides](09-mcp-and-agents-md/slides.md) | Why shared standards matter: MCP for tools and data, AGENTS.md for project context. | [Introducing MCP](https://www.anthropic.com/news/model-context-protocol) · [What is MCP?](https://modelcontextprotocol.io/docs/getting-started/intro) |
| 10 | [How MCP servers work](10-build-mcp-server/article.md) · [slides](10-build-mcp-server/slides.md) | Tools as described functions: names, descriptions and input schemas, and how to build and test a server. | [Build an MCP server](https://modelcontextprotocol.io/docs/develop/build-server) · [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) |
| 11 | [Agent Skills](11-skills/article.md) · [slides](11-skills/slides.md) | Packaging know-how as folders an agent loads in stages (progressive disclosure), and how skills differ from MCP. | [Equipping agents with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) · [Introducing Agent Skills](https://www.claude.com/blog/skills) |
| 12 | [Skill supply-chain security](12-skill-supply-chain/article.md) · [slides](12-skill-supply-chain/slides.md) | Installed skills and plugins are third-party code plus instructions: how they attack and how to vet them. | [Agent Skills in the Wild](https://arxiv.org/abs/2601.10338) · ["Do Not Mention This to the User"](https://arxiv.org/abs/2602.06547) |
| 13 | [Prompt injection](13-prompt-injection/article.md) · [slides](13-prompt-injection/slides.md) | Why models obey instructions hidden in content, the "lethal trifecta", and designing systems that limit the damage. | [OWASP LLM01 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) · [Simon Willison: injection vs. jailbreaking](https://simonwillison.net/2024/Mar/5/prompt-injection-jailbreaking/) |
| 14 | [Dynamic context discovery](14-dynamic-context-discovery/article.md) · [slides](14-dynamic-context-discovery/slides.md) | Keep always-loaded context small and let the agent pull bulky material (tool output, logs, tool definitions) on demand. | [Cursor: Dynamic context discovery](https://cursor.com/blog/dynamic-context-discovery) · [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
| 15 | [A2A protocol](15-a2a-protocol/article.md) · [slides](15-a2a-protocol/slides.md) | How agents discover and delegate to other agents (Agent Cards, tasks), and how A2A complements MCP. | [Announcing A2A](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/) · [A2A specification v1.0](https://a2a-protocol.org/latest/specification/) |

## Track 5: Building AI features

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 16 | [Embeddings and RAG](16-embeddings-and-rag/article.md) · [slides](16-embeddings-and-rag/slides.md) | Embeddings, the retrieval pipeline, hybrid search, failure modes, measuring retrieval, and when RAG is the wrong tool. | [RAG paper (Lewis et al.)](https://arxiv.org/abs/2005.11401) · [OpenAI embeddings guide](https://developers.openai.com/api/docs/guides/embeddings) |
| 17 | [Prompting vs. RAG vs. fine-tuning](17-prompting-vs-rag-vs-fine-tuning/article.md) · [slides](17-prompting-vs-rag-vs-fine-tuning/slides.md) | Which lever changes what (instructions, knowledge access or weights), and how to choose. | [OpenAI model optimization](https://developers.openai.com/api/docs/guides/model-optimization) |
| 18 | [Evals](18-evals/article.md) · [slides](18-evals/slides.md) | Measuring whether an AI feature works: error analysis, small datasets, code checks vs. LLM-as-judge, regression testing, evals for agents. | [Hamel Husain: Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) · [Using LLM-as-a-Judge](https://hamel.dev/blog/posts/llm-judge/) |
| 19 | [Open-weight and local models](19-open-weight-and-local-models/article.md) · [slides](19-open-weight-and-local-models/slides.md) | Open-weight vs. open source, quantization, running models locally, licenses, and testing on your own task. | [OSI Open Source AI Definition](https://opensource.org/ai/open-source-ai-definition) |
| 20 | [Data privacy and token cost](20-data-privacy-and-cost/article.md) · [slides](20-data-privacy-and-cost/slides.md) | What you send and where it goes, data minimization, and why agents multiply cost; caching, batching and routing. | [Anthropic API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention) · [Claude Code data usage](https://code.claude.com/docs/en/data-usage) |

## Track 6: Agent architecture

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 21 | [The harness matters as much as the model](21-harness-over-model/article.md) · [slides](21-harness-over-model/slides.md) | On large codebases, results depend on finding the right context: instruction files, hooks, skills, LSP, MCP and subagents. | [How Claude Code works in large codebases](https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start) · [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
| 22 | [Agent loops](22-agent-loops/article.md) · [slides](22-agent-loops/slides.md) | Goal, action, check, stop: the shared anatomy of agent loops, and how loops differ in what checks and what persists. | [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) · [ReAct](https://arxiv.org/abs/2210.03629) |
| 23 | [Subagents vs. agent teams](23-subagents-vs-teams/article.md) · [slides](23-subagents-vs-teams/slides.md) | Workers that report back vs. sessions that coordinate with each other, and when peer communication is worth the cost. | [Claude Code agent teams](https://code.claude.com/docs/en/agent-teams) · [Claude Code subagents](https://code.claude.com/docs/en/sub-agents) |
| 24 | [The advisor strategy](24-advisor-strategy/article.md) · [slides](24-advisor-strategy/slides.md) | A cheaper executor that consults a stronger model only at decisive points: a general cost/quality pattern. | [The advisor strategy](https://claude.com/blog/the-advisor-strategy) · [Advisor tool docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) |
| 25 | [Graphs and verification](25-graphs-and-verification/article.md) · [slides](25-graphs-and-verification/slides.md) | From loops to multi-agent graphs, common workflow shapes, and why every node needs its own check. | [Building verification loops with skills](https://claude.com/blog/building-verification-loops-in-claude-code-with-skills) · [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) |
| 26 | [Agent harness design](26-agent-harness/article.md) · [slides](26-agent-harness/slides.md) | Planner, generator and evaluator: each harness component encodes an assumption about the model, so keep doing separate from judging. | [Harness design for long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) · [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) |
| 27 | [Running agents in parallel](27-parallel-agents/article.md) · [slides](27-parallel-agents/slides.md) | Isolation (worktrees, sandboxes), handoffs, and orchestration when several sessions share one codebase. | [Claude Code: run agents in parallel](https://code.claude.com/docs/en/agents) · [Parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees) |

## Track 7: The bigger picture

| # | Chapter | What you'll learn | Key sources |
|---|---|---|---|
| 28 | [A lifecycle for agentic systems](28-adlc/article.md) · [slides](28-adlc/slides.md) | Why building agents needs extra lifecycle work (evaluation, staged rollout, monitoring), synthesized from established guidance. | [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) · [OpenAI: A practical guide to building agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| 29 | [Reward hacking and emergent misalignment](29-reward-hacking/article.md) · [slides](29-reward-hacking/slides.md) | What reward hacking is, how learning to cheat generalized to worse behaviour in an Anthropic study, and which mitigations helped. | [Anthropic: From shortcuts to sabotage](https://www.anthropic.com/research/emergent-misalignment-reward-hacking) · [Paper (arXiv 2511.18397)](https://arxiv.org/abs/2511.18397) |
| 30 | [The forward deployed engineer](30-forward-deployed-engineer/article.md) · [slides](30-forward-deployed-engineer/slides.md) | Fitting AI into how work really gets done: the FDE role, its origins, and the skills it needs. | [Palantir: Dev versus Delta](https://blog.palantir.com/dev-versus-delta-demystifying-engineering-roles-at-palantir-ad44c2a6e87) |

---

## Read, watch and listen

The course is online at <https://ai-fundamentals.kwise.app/>: every chapter as an article, a slide deck and a
podcast episode with a transcript that follows the audio. PDFs of every article and deck, and the podcast audio,
are attached to the [releases](../../releases).

The `slides.md` files are [Slidev](https://sli.dev) sources. They use the course theme, which is not part of
this repository, so read them here or watch the rendered decks online.

## License

© 2026 Patrick Demichiel. The course text, slides, podcast scripts and podcast episodes are licensed under
[CC BY-NC-SA 4.0](LICENSE): you may share and adapt them for non-commercial purposes if you credit
"AI Fundamentals by Patrick Demichiel" with a link to this repository and release adaptations under the same
license. Commercial use is reserved to the author.

Not covered by this license:

- Quoted passages, which keep the copyright of the sources they come from.
- The AI Fundamentals name, logo and wordmark.

The podcast voices are synthetic. Episodes 05 to 30 were rendered with Fish Audio Drama 3: Built with Fish Audio.
Episodes 00 to 04 were rendered with ElevenLabs.

Corrections and suggestions are welcome as issues. Pull requests are not accepted, so that the whole course stays
with one copyright holder.
