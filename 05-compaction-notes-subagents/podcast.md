<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 05: Compaction, notes and subagents

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm still the one learning here.

BEN: And I'm Ben. Last time I promised we'd go deeper into the three techniques for work that outlasts one context window. So that's today, chapter five: compaction, structured notes, and subagents.

MAYA: Quick reminder for me first. Chapter four was context rot. The longer the context gets, the less reliably the model uses it. And after every turn, I pick one of five moves: continue, rewind, clear, compact, or hand it to a subagent.

BEN: Exactly. Those were choices you make turn by turn. Today is about tasks that run for hours, and need far more tokens than any window holds.

MAYA: So, the long game.

BEN: The long game. And here's the one sentence for today. A context window is a finite working space, so work that outlasts it needs three techniques. Compress the history, that's compaction. Keep important state in files outside the window, that's structured notes and memory. And split large jobs across agents that each start with a clean window, that's subagents.

MAYA: Compress it, write it down, split it up.

BEN: [laughs] That's a good short version.

MAYA: Let's start with why agents lose track at all.

BEN: A model has no memory between turns. Each turn it sees one block of tokens, the context window. In an agent, that block holds the system prompt, tool definitions, memory files, and every file read and every tool result so far. Anthropic describes the model's attention as a budget that each token draws on. So they argue context should be curated, so the model sees, quote, "the smallest possible set of high-signal tokens."

MAYA: [curious] But windows keep growing. Why not just wait a year until they're big enough for anything?

BEN: Anthropic's context-engineering guide answers that directly. Quote: "context windows of all sizes will be subject to context pollution and information relevance concerns." So waiting for bigger windows won't solve it. Their answer is the three techniques.

MAYA: Okay. Technique one, compaction. We touched on it last time.

BEN: We did, so let's add what's new. Compaction takes a conversation that's nearing the limit, summarises it, and starts a new context window from that summary. In Claude Code, the model keeps, quote, "architectural decisions, unresolved bugs, and implementation details", and drops redundant tool output.

MAYA: The hard part must be deciding what to keep.

BEN: That's exactly the hard part. And Anthropic has clear advice for anyone writing a compaction prompt. First maximise recall, so nothing important is lost. Then improve precision, by cutting what isn't needed.

MAYA: Recall and precision. Translate that for me.

BEN: Recall asks: did everything important make it into the summary? Precision asks: is everything in the summary actually needed? You tune for the first one first.

MAYA: [thoughtful] So, keep everything relevant first, and only then trim.

BEN: That's the order. Recall first, precision second.

MAYA: Claude Code does compaction automatically anyway, right?

BEN: It does, as you approach the limit. But you can also trigger it yourself, and the chapter has a little table comparing the two. Automatic fires near the limit, and you can move that threshold with {/autocompact|slash autocompact}. It keeps what the model guesses matters. And the risk is that it fires mid-task, when rot is worst.

MAYA: And manual?

BEN: Manual happens when you run {/compact|slash compact}. It keeps what you tell it, for example {/compact|slash compact} focus on the API changes. The risk there is just that you have to remember to do it.

MAYA: [laughs] That's the "most tired version of the model writes the summary" problem from last time.

BEN: Right. Anthropic's Claude Code team notes that automatic compaction happens when the model is least sharp, and has to guess where the work is going. So they recommend compacting proactively, with a hint about the next step. And you can put standing rules in {CLAUDE.md|CLAUDE dot M D}, like "When compacting, always preserve the full list of modified files and any test commands."

MAYA: And what survives a compaction?

BEN: Claude Code reloads the system prompt, the project-root {CLAUDE.md|CLAUDE dot M D}, auto memory and the current plan from disk, and it re-reads up to five recently modified files. Instructions you gave only in the conversation get summarised with everything else.

MAYA: Which sounds like a hint that the real answer is files.

BEN: [laughs] You're ahead of me. That's one reason to keep durable state in files, and we'll get there. But first, there's a lighter form of compaction.

MAYA: Lighter than summarising?

BEN: Much lighter. Anthropic calls tool result clearing, quote, "one of the safest lightest touch forms of compaction". The idea is that once a tool was called deep in the history, the agent rarely needs its raw output again.

MAYA: Like a long search result from an hour ago. The agent already used it.

BEN: Right. On the Claude API, this is available as context editing, in beta as of October 2026. There's a strategy called {clear_tool_uses_20250919|clear tool uses}, with a version number on the end. Once the input passes a trigger, it replaces the oldest tool results with placeholders.

MAYA: What's the trigger?

BEN: By default, a hundred thousand input tokens. It keeps the three most recent tool uses, and you can exempt specific tools. The API also offers server-side compaction, which summarises older turns for you. That's also in beta as of October 2026.

MAYA: Does clearing actually help, or is it just tidying up?

BEN: When Anthropic launched context editing with Claude Sonnet 4.5 in September 2025, they reported a hundred-turn web-search evaluation. Context editing let agents finish workflows that would otherwise run out of context, while cutting token use by eighty-four percent.

MAYA: Eighty-four percent. Okay, that's more than tidying.

BEN: Now, compaction works within a session. Across sessions, the agent starts each one with a fresh window and no memory of the last.

MAYA: So every new session is the agent's first day at work.

BEN: [laughs] Without help, yes. That's where the second idea comes in: keeping state in files. It comes in two flavours. Memory files that load at the start of every session, and structured notes the agent writes as it works. Memory files first.

MAYA: Go.

BEN: Claude Code has two mechanisms, both loaded at the start of every session. {CLAUDE.md|CLAUDE dot M D} is written by you. It holds instructions and rules: build commands, conventions, architecture notes. And it's loaded in full.

MAYA: And the other one?

BEN: Auto memory. That one is written by Claude. It holds learnings: corrections, preferences, patterns it noticed. Only the first two hundred lines or twenty-five kilobytes of {MEMORY.md|MEMORY dot M D} are loaded. Topic files are read only on demand.

MAYA: [curious] So if I tell Claude to remember that we use tabs, where does that go?

BEN: Into auto memory. If you want it in {CLAUDE.md|CLAUDE dot M D}, you have to ask explicitly.

MAYA: Then I'll just put everything in {CLAUDE.md|CLAUDE dot M D}. It's loaded in full, so nothing gets lost.

BEN: That's the trap. The docs advise keeping each {CLAUDE.md|CLAUDE dot M D} under about two hundred lines, because, quote, "longer files consume more context and reduce adherence." Workflow-specific instructions should move into skills, which load only when used. Chapter eleven covers skills.

MAYA: Right. Loaded in full means it costs context in full. Every single session.

BEN: Exactly.

MAYA: Okay, so memory files are what the agent knows when it wakes up. What are structured notes?

BEN: Structured note-taking means the agent regularly writes notes to files outside the context window, and reads them back when it needs them. Anthropic's examples range from a simple to-do list to Claude playing Pokémon.

MAYA: [laughs] Pokémon?

BEN: Pokémon. The agent kept notes such as, quote, "for the last {1,234|twelve hundred and thirty-four} steps I've been training my Pokémon in Route {1|one}". And it kept them across thousands of steps and context resets.

MAYA: So it's like a diary it leaves for a future self who won't remember anything.

BEN: That's a nice analogy. And there's a concrete version for coding. Anthropic's article on harnesses for long-running agents, by Justin Young, gives a setup for projects that span many sessions.

MAYA: Walk me through it.

BEN: There are two kinds of session. First, an initializer session creates the scaffolding. A feature list, over two hundred features in their example, all marked failing. A progress file, {claude-progress.txt|claude progress dot T X T}. A script, {init.sh|init dot S H}, that starts the app. And a first git commit.

MAYA: And then?

BEN: Then every later coding session follows a routine. It gets its bearings by reading the progress file and the git log. It picks the highest-priority unfinished feature, runs a basic end-to-end test, and works on one feature at a time. And it ends by committing and updating the progress notes.

MAYA: One feature at a time sounds slow.

BEN: It answers two failures the authors saw. Agents that tried to do too much at once, and ran out of context mid-change. And later agents that saw partial progress, and declared the job done.

MAYA: [laughs] The second one is very human. And the feature list, is that Markdown?

BEN: No, JSON. Because the model was less likely to change or overwrite JSON inappropriately.

MAYA: Interesting. Okay, that's the Claude Code way. Is there something like this on the API?

BEN: Yes, the memory tool. It gives the agent a {/memories|slash memories} directory, where it can view, create, edit, rename and delete files.

MAYA: So Anthropic stores my agent's memories somewhere on their side?

BEN: No, and that's an important detail. It runs client-side. Claude requests the operation, and your application performs it, on storage you control.

MAYA: Oh. So I write the code that actually touches the files.

BEN: You do. And when the tool is enabled, the API adds a protocol to the system prompt. It tells Claude to check its memory first, and to, quote, "ASSUME INTERRUPTION", because its context window "might be reset at any moment".

MAYA: [laughs] That's a slightly anxious instruction.

BEN: But a useful one. Now, since you're writing the handler, here's the security warning. If the handler doesn't validate paths, a path such as {/memories/../../secrets.env|slash memories, dot dot, dot dot, secrets dot env} can climb out and reach files outside the directory. So the handler must reject anything that resolves outside {/memories|slash memories}.

MAYA: Noted. And how does the memory tool fit with the clearing we talked about?

BEN: They pair up. As the context nears the clearing threshold, Claude gets a warning, so it can save important information before the old tool results are cleared.

MAYA: Write it down before it's wiped.

BEN: Exactly. And in Anthropic's internal agentic-search eval, memory plus context editing improved performance by thirty-nine percent over baseline. Context editing alone gave twenty-nine percent.

MAYA: Okay, but here's my pushback. Notes are just more text. If the agent reads them all back in, aren't we filling the context again?

BEN: Good objection. The chapter gives three reasons notes work. First, state survives compaction, clearing and restarts, because it lives on disk. Second, the window stays lean, because notes are read when needed, not kept in context.

MAYA: Read when needed. But how does it know what to read?

BEN: That's Anthropic's just-in-time approach. You keep lightweight identifiers, such as file paths, and load the content with tools at runtime. Folder names, file names and timestamps tell the agent what a file is for before it opens it.

MAYA: Like labels on boxes in a storage room. You don't unpack every box to find the winter coats.

BEN: [laughs] Good analogy. And the third reason: structure resists drift. A fixed format with required fields is harder to quietly corrupt than free prose.

MAYA: Let me summarise the files part. Memory files load at the start of each session, with standing rules and learnings. Notes get written during the work and read back just in time. And both survive compaction, because they're on disk.

BEN: That's right. And keep {CLAUDE.md|CLAUDE dot M D} short.

MAYA: Okay. Last technique: subagents.

BEN: Instead of one agent holding everything, a lead agent plans and coordinates, while subagents do focused work, each in its own clean context window. A subagent may use tens of thousands of tokens exploring, but it returns, quote, "a condensed, distilled summary of its work", often {1,000-2,000|one to two thousand} tokens.

MAYA: The chapter has a diagram for this, right? Describe it for me.

BEN: A user request goes to the lead agent, which plans and delegates. It fans out to three subagents, A, B and C. Each has its own context and tools, and each sends back a short summary. Then the lead combines the summaries into the answer.

MAYA: And does that actually beat one agent?

BEN: Anthropic's research system uses exactly this orchestrator-worker design. With Claude Opus 4 as the lead and Sonnet 4 subagents, it outperformed a single Opus 4 agent by ninety point two percent on an internal research eval.

MAYA: Wow.

BEN: And two refinements. Subagents can write their output to external storage and pass back only, quote, "lightweight references". And the lead saves its plan to memory, in case its own context gets truncated.

MAYA: So even the boss keeps notes.

BEN: [laughs] Even the boss. In Claude Code, subagents are built in. Explore, which is read-only search, Plan, and general-purpose ship by default. And you can define your own as Markdown files in {.claude/agents/|dot claude slash agents}, each with its own prompt, tool list and model.

MAYA: Okay, so my takeaway is: always use subagents. Ninety percent better!

BEN: [sighs] And that's the misconception the chapter is most careful about. Anthropic is direct about the costs. First, tokens. Agents used about four times the tokens of chat, and multi-agent systems about fifteen times. That only pays off on high-value tasks.

MAYA: Fifteen times. Okay, that cools me down.

BEN: Second, coordination failures. Early versions spawned fifty subagents for simple queries. And vague task descriptions led subagents to duplicate each other's work.

MAYA: [laughs] Fifty subagents for a simple question. How did they fix that?

BEN: Explicit delegation. Each subagent gets an objective, an output format, tools and task boundaries.

MAYA: And the third cost?

BEN: Poor fits. Tasks where all agents need the same context, or that have many dependencies. The authors note that, quote, "most coding tasks involve fewer truly parallelizable tasks than research."

MAYA: [thoughtful] So for coding, there's often less to split than I'd think.

BEN: Fewer truly parallel pieces, yes. And Claude Code's docs add two things. Subagent requests count toward the same usage limits. And many subagents, each returning detailed results, can still fill the main context. Chapter twenty-three compares subagents with agent teams.

MAYA: So how do I choose between all of this?

BEN: Anthropic's guide matches each technique to a task shape. A long back-and-forth that needs to keep flowing: compaction. Iterative development with clear milestones: structured notes, plus memory files. Complex research or analysis that can run in parallel: subagents.

MAYA: And can I mix them?

BEN: They combine well. A long project typically has a {CLAUDE.md|CLAUDE dot M D} for standing rules, a progress file and feature list for evolving state, compaction at milestones, and a subagent for the occasional large investigation.

MAYA: Okay. What part of this will last, and what will date?

BEN: The durable principles first. Context is a working budget, not storage, and larger windows don't remove the need to curate it. Summarise deliberately, recall first, precision second. Keep durable state in files outside the window, in a structured format, and load it just in time. Give subtasks clean contexts and pass back short summaries or references. And every extra agent costs tokens and coordination, so use them where work really separates.

MAYA: And what will date?

BEN: API feature names, versions and beta status: the clear tool uses strategy, the memory tool, server-side compaction. Claude Code commands and limits: {/compact|slash compact}, {/autocompact|slash autocompact}, auto memory's two hundred lines or twenty-five kilobytes, and the built-in subagent names. All of that as of October 2026. And the reported numbers, thirty-nine, eighty-four and ninety point two percent, and fifteen times, come from Anthropic's own evaluations of 2025 models.

MAYA: Let me try to sum up, and you correct me. The window is finite, and bigger windows won't fix context pollution. So for long work there are three techniques. Compaction compresses the history. Tune it for recall first, then precision, and do it yourself with a hint instead of waiting for the automatic pass. Clearing old tool results is the lightest form.

BEN: Good so far.

MAYA: Second, files. {CLAUDE.md|CLAUDE dot M D} and auto memory load every session, so the agent doesn't start from zero. Structured notes, like a progress file and a JSON feature list, survive compaction and restarts, and get read back just in time. And third, subagents work in clean windows and return short summaries. They're great for parallel research, but cost about fifteen times the tokens of chat.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You're writing a compaction prompt. What do you tune for first, and what second?

PAUSE: 5

MAYA: Recall first, so nothing important is lost. Then precision, cutting what isn't needed.

BEN: Perfect. Question two. You're building an app over many sessions, with clear milestones. Which technique fits, and what would you set up?

PAUSE: 5

MAYA: Structured notes, plus memory files. A progress file and a JSON feature list, updated before each session ends, so the next session gets its bearings from them and the git log. And {CLAUDE.md|CLAUDE dot M D} for the standing rules.

BEN: Exactly. And last one. You give an API agent the memory tool. What must your handler do?

PAUSE: 5

MAYA: Validate paths. Reject anything that resolves outside {/memories|slash memories}, because otherwise a path with dot dot in it can reach files outside, like a secrets file.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the progress file one: have the agent keep {progress.md|progress dot M D} with done, in progress and next, plus a JSON feature list. Then start a fresh session and let it get its bearings from those files alone.

BEN: And try compacting with an explicit focus, trimming your {CLAUDE.md|CLAUDE dot M D}, the memory tool on the API, and one task run with and without subagents. Compare quality, time and token use. Next time, in chapter six, we look at agent instruction files. After how often {CLAUDE.md|CLAUDE dot M D} came up today, that's a good place to go.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter six.
