<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 21: The harness matters as much as the model

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Last time, in chapter twenty, we talked about data privacy and token cost. Today we start a new track, agent architecture, with chapter twenty-one: the harness matters as much as the model.

MAYA: The harness. I've heard that word a lot, but I'd struggle to define it.

BEN: We'll get there. First, the one sentence for today. On large codebases, coding agents succeed or fail mainly on whether they find the right context. So results depend as much on the harness around the model, how it searches, which instructions load and when, which rules are enforced, which tools and helpers it has, as on the model itself.

MAYA: [curious] So it's not just "use the smartest model and you're done."

BEN: Not on a big codebase. And it starts with a basic question. How does an agent find the code it needs in the first place?

MAYA: I'd guess it searches somehow. But I don't know how.

BEN: There are two broad ways. The first is retrieval, also called RAG. A retrieval system embeds the codebase into a vector index ahead of time. When you ask something, it finds the chunks most similar to your request and adds them to the model's context. Chapter sixteen covers the mechanics.

MAYA: That sounds sensible. What's the problem?

BEN: Anthropic's context-engineering guide calls this "embedding-based pre-inference time retrieval", and notes that many AI applications use it. But for code that changes daily, the index is a copy, and copies drift. Anthropic's guide for large codebases puts it concretely. A stale index can return, quote, "a function the team renamed two weeks ago".

MAYA: [laughs] Ouch. That's exactly the kind of thing that would confuse me too.

BEN: And there's a second problem. In a very large codebase, the most similar chunk is often not the right one. Similar and right are different things.

MAYA: Okay. And the second way?

BEN: Agentic search. Claude Code, in Anthropic's words, "navigates a codebase the way a software engineer would". It lists and reads files, runs grep, and follows references. It runs locally and needs no index.

MAYA: So it just explores the real files, like I would.

BEN: Right. The context-engineering guide describes it as a hybrid. The {CLAUDE.md|CLAUDE dot M D} files are dropped into context up front, and then tools like glob and grep let the agent find files just in time. That, quote, bypasses "the issues of stale indexing and complex syntax trees".

MAYA: Did they just pick that approach, or did they try the alternatives?

BEN: They tried them. Boris Cherny, who created Claude Code, told The Pragmatic Engineer that the team tried local vector databases and recursive model-based indexing first. Both brought stale indexes and permission complexity. His summary was, quote, "Plain glob and grep, driven by the model, beat everything."

MAYA: Let me check I've got the comparison. Retrieval works from a pre-built index, picks by similarity, and can hand the model near misses. Agentic search works from the live file system, and narrows things down to what the agent has actually confirmed.

BEN: That's the chapter's comparison, yes. And each has a weak spot. Retrieval struggles with fast-changing code and with permissions. Agentic search needs signposts, so it knows where to look.

MAYA: Signposts?

BEN: Because agentic search has a cost too. A vague search across a huge codebase can exhaust the context window. Anthropic's guide says agentic search "works best when Claude has enough starting context to know where to look". So much of the rest of the chapter is about signposts. Giving the search a good starting point, and keeping what it loads relevant.

MAYA: [thoughtful] So is retrieval dead, then?

BEN: No, and that's worth saying. Some advanced teams expose structured code search to Claude as an MCP tool. Retrieval isn't obsolete. It just isn't the default here.

MAYA: Okay, now the definition. What exactly is the harness?

BEN: Everything around the model that shapes its behaviour. Instruction files, tools, hooks, skills, subagents and configuration. Anthropic's guide says this ecosystem, quote, "determines how Claude Code performs more than the model alone".

MAYA: More than the model. That's a strong claim.

BEN: And the reason goes back to chapters three and four. The context-engineering guide calls context "a finite resource with diminishing marginal returns". Good context engineering means "finding the smallest possible set of high-signal tokens" for the task. The harness is how you get those tokens in, and keep the rest out.

MAYA: So can I just copy someone's good setup and be done?

BEN: Built-in defaults suit codebases in general. A large codebase needs a harness tuned to its own structure, conventions and tools. And order matters. Anthropic says "each layer builds on what came before".

MAYA: Where do I start, then? Configure everything on day one?

BEN: The opposite. Claude Code's documentation recommends growing the setup in response to specific triggers. For example, if Claude gets a convention wrong twice, that convention belongs in {CLAUDE.md|CLAUDE dot M D}.

MAYA: I like that. Fix what actually breaks.

BEN: Exactly. Now, the pieces. Anthropic's guide has a table of seven components, with when each loads, what it's best for, and the mistake teams most often make with it. I'll walk you through them.

MAYA: Go.

BEN: {CLAUDE.md|CLAUDE dot M D} loads every session, and it's for conventions and codebase knowledge. The common mistake is putting reusable expertise there instead of in a skill. Hooks run on lifecycle events. The mistake is using prompts for steps that should run automatically. Skills load on demand, for reusable expertise. The mistake is loading everything into {CLAUDE.md|CLAUDE dot M D} instead.

MAYA: Three down.

BEN: Plugins, for distributing a working setup. The mistake is letting good setups stay tribal. LSP, for symbol navigation and errors in typed languages. The mistake is assuming it works automatically. MCP servers, for internal tools Claude can't otherwise reach. The mistake is building MCP before the basics work.

MAYA: And the seventh?

BEN: Subagents, which load when invoked, for separating exploration from editing, and for parallel work. The mistake is exploring and editing in one session.

MAYA: Okay, let's go one by one. {CLAUDE.md|CLAUDE dot M D} first.

BEN: It loads at the start of every session. Anthropic's advice for large repositories is that "the root file should be pointers and critical gotchas only". Local conventions go in subdirectory files.

MAYA: How short is short?

BEN: As of October 2026, Claude Code's docs say, quote, "target under {200|two hundred} lines per {CLAUDE.md|CLAUDE dot M D} file. Longer files consume more context and reduce adherence."

MAYA: Under two hundred lines. Can I cheat by splitting it into imported files?

BEN: [laughs] That's the obvious trick, and it doesn't work. Splitting with {@imports|at imports} helps organisation, but it saves nothing, because imports also load at launch.

MAYA: Hmm. So how do I keep things out of every session?

BEN: Layering. Files above your working directory load at launch. Files in subdirectories load on demand, when Claude reads or edits files there. That's why Anthropic recommends starting a session in the relevant subdirectory. Claude walks up the tree and loads every {CLAUDE.md|CLAUDE dot M D} on the way.

MAYA: And for rules that only apply to certain files?

BEN: Path-scoped rules. Markdown files in {.claude/rules|dot claude slash rules}, with a paths field in their frontmatter, load only when Claude works on matching files.

MAYA: Nice. And if I write "never do X" in there, that's that?

BEN: No, and this is the big limit. The docs call {CLAUDE.md|CLAUDE dot M D} "context, not enforced configuration". If you need to block an action regardless of what the model decides, use a hook.

MAYA: Which is the second piece.

BEN: Right. Hooks are commands that Claude Code runs at lifecycle events. The docs say they give "deterministic control: certain actions always happen rather than relying on the LLM to choose to run them". And they put the contrast bluntly. Quote, "An instruction like 'never edit {.env|dot env}' in {CLAUDE.md|CLAUDE dot M D} or a skill is a request, not a guarantee."

MAYA: A request, not a guarantee. What does a hook look like in practice?

BEN: There are several events. {PreToolUse|Pre tool use} runs before a tool call. If your script exits with code two, the action is blocked, and whatever it wrote to standard error goes back to Claude as feedback. The docs' example blocks edits to {.env|dot env}, lock files and the {.git|dot git} folder.

MAYA: So the model gets told why it was stopped.

BEN: Exactly. Then {SessionStart|session start} can add context, for example re-injecting critical notes after compaction. {PostToolUse|Post tool use} can run formatters or linters after each edit. And the stop hook runs when Claude finishes responding.

MAYA: What would I use a stop hook for?

BEN: Anthropic suggests one that will, quote, "reflect on what happened during a session and propose {CLAUDE.md|CLAUDE dot M D} updates while the context is fresh".

MAYA: [curious] Oh, so the harness learns from each session. That's clever.

BEN: It is, with two warnings. Stop hooks fire after every response, not only when the task is done. And they need a guard, a flag called {stop_hook_active|stop hook active}, to avoid loops. Also, review the proposed updates before you accept them.

MAYA: Got it. Third piece, skills.

BEN: Skills load their full content only when relevant. That's chapter eleven. Anthropic recommends moving specialised workflows out of {CLAUDE.md|CLAUDE dot M D} and into skills, and scoping skills to paths. Their example is a payments team's deployment skill, bound to its directory, so it "never auto-loads" elsewhere in the monorepo.

MAYA: And Claude Code supports that?

BEN: With the paths field in the skill's frontmatter, yes.

MAYA: Plugins next.

BEN: A plugin bundles skills, hooks, subagents and MCP servers into one installable unit. For teams, that stops good setups from staying "tribal". Updates can be distributed through managed marketplaces. But plugins run code with your privileges, so vet third-party ones. That's chapter twelve.

MAYA: And LSP. I know it from my editor, but what does it do for an agent?

BEN: A language server gives Claude IDE-style navigation. Go to definition, find references, type errors after edits. Anthropic's guide says, "Without it, Claude pattern-matches on text and can land on the wrong symbol."

MAYA: Because grep just matches strings.

BEN: Right. A grep for a common function name can return thousands of matches, while "LSP returns only the references that point to the same symbol".

MAYA: Does that just work out of the box?

BEN: That's the common confusion. In Claude Code, the LSP tool stays inactive until you install a code intelligence plugin for your language. And one company in Anthropic's guide deployed LSP across the whole organisation before rollout, so C and {C++|C plus plus} navigation would be reliable.

MAYA: Okay. MCP. We had two chapters on it.

BEN: Chapters nine and ten. MCP servers give Claude access to internal tools, data and APIs it can't otherwise reach. Anthropic lists "building MCP connections before the basics work" as a common mistake. Get instructions, hooks and navigation right first.

MAYA: And the last piece, subagents. We met those in chapter five.

BEN: We did, so just the large-codebase angle. A subagent "runs in its own context window" and "returns only the summary". The context-engineering guide notes that such summaries are often one to two thousand tokens, however much the subagent read.

MAYA: Anything specific for big repositories?

BEN: Three things. Claude Code's built-in Explore agent is read-only and built for code search. A project or user subagent named Explore overrides it, so you can write one that already knows your repository's layout.

MAYA: Oh, that's handy.

BEN: Second, a documented pattern is a read-only subagent that maps a subsystem and writes its findings to a file. The main agent then uses that file for editing. And third, independent research can run in parallel. But many detailed results still consume the main context, so ask for concise returns.

MAYA: Let me sum up the pieces. {CLAUDE.md|CLAUDE dot M D} is the short, always-on core. Hooks enforce. Skills bring expertise on demand. Plugins share the setup. LSP navigates symbols instead of strings. MCP reaches internal systems. And subagents keep exploration out of the main session.

BEN: [warm] That's a very good summary.

MAYA: The chapter also had a list of large-codebase practices, right?

BEN: Yes, all from Anthropic's guide. First, scope test and lint commands per subdirectory. Running the full suite after changing one service, quote, "causes timeouts and wastes context". That's harder in compiled monorepos with deep cross-directory dependencies.

MAYA: Second?

BEN: Exclude what the agent shouldn't touch. Commit deny rules in the project's settings file for generated files, build artifacts and third-party code, so the whole team gets them.

MAYA: Third?

BEN: Write a codebase map when the directory structure doesn't explain the code. Anthropic describes it as "a lightweight markdown file at the repo root listing each top-level folder with a one-line description". If you have hundreds of top-level folders, layer the maps.

MAYA: That's a signpost again.

BEN: Exactly. Fourth, use LSP for symbol search in typed languages. And fifth, review the setup "every three to six months", and after major model releases.

MAYA: [curious] Why would a good setup go bad?

BEN: Because instructions written for one model's weaknesses can hold back a later model. Anthropic's example is a rule forcing single-file refactors, which would block coordinated cross-file edits. Hooks can become redundant too. One team had a hook enforcing {p4 edit|P four edit} for Perforce. It became unnecessary once Claude Code added native Perforce support.

MAYA: So the harness needs pruning, not just adding.

BEN: Right. And the last practice is to assign an owner. The fastest rollouts had someone responsible for the configuration, at minimum one directly responsible individual. Regulated organisations started with approved skills, required code review and limited access.

MAYA: Does this all scale forever?

BEN: The guide names its own limits. Hierarchical {CLAUDE.md|CLAUDE dot M D} can break down at hundreds of thousands of folders, or on version control other than git.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Agent performance on large codebases is mostly a context-finding problem. Search the live source of truth, and give the search signposts, like maps, scoped instructions and symbol navigation. Layer context: a short always-on core, scoped files per area, on-demand skills and isolated subagents. Enforce critical rules mechanically, because instructions are requests. Package and share the harness, and give it an owner. And re-evaluate it as models improve, removing what no longer helps.

MAYA: And what will date?

BEN: As of October 2026, the Claude Code file names, hook event names, exit-code conventions, the paths field and how plugins work. The "under two hundred lines" guidance, and the subagent override rules. And whether a given tool uses an index, agentic search or both. That varies by product, and it changes.

MAYA: Let me recap the whole thing, and you correct me. Agents find code either by retrieval from an index, or by agentic search through the live files. Claude Code uses agentic search, because indexes go stale. But that search needs signposts, and that's a big part of what the harness provides.

BEN: Good so far.

MAYA: The harness is everything around the model, and on big codebases it matters as much as the model. Keep {CLAUDE.md|CLAUDE dot M D} short and layered, put procedures in skills, enforce real rules with hooks, and review it all every three to six months.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your {CLAUDE.md|CLAUDE dot M D} says "never edit generated files", and the agent sometimes does it anyway. What do you do?

PAUSE: 5

MAYA: Turn it into a hook. A {PreToolUse|pre tool use} hook that exits with code two and a clear message, so the edit is blocked and Claude gets told why. Because the instruction is a request, not a guarantee.

BEN: Perfect. Question two. Why does Claude Code use agentic search instead of a pre-built index, and what does agentic search need to work well?

PAUSE: 5

MAYA: Because an index is a copy that goes stale, and can return a function the team renamed two weeks ago. Agentic search uses the live files, but it needs enough starting context to know where to look. Signposts, like a codebase map, scoped instructions and LSP.

BEN: Exactly. And last one. Your harness was set up a year ago and it still works. Why review it anyway?

PAUSE: 5

MAYA: Because rules written for an older model's weaknesses can hold back a newer one, like forcing single-file refactors. And some hooks become redundant once the tool adds native support.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the first: count the lines in my instruction file, and move directory-specific parts and procedures out of it.

BEN: And then turn one instruction into a hook, add a stop hook that proposes {CLAUDE.md|CLAUDE dot M D} updates, compare text search with LSP, and write a codebase map for a repository you don't know well. Next time, in chapter twenty-two, we look at agent loops. Goal, action, check, stop: what every agent loop has in common.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-two.
