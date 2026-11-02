<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 23: Subagents vs. agent teams

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Last time, in chapter twenty-two, we looked at agent loops: goal, action, check, stop. One of those loops had critics that could either report to a central orchestrator, or message each other. Today, chapter twenty-three, is that choice in detail: subagents versus agent teams.

MAYA: We already met subagents back in chapter five, right? Their own clean window, a short summary back.

BEN: Right, so we won't re-teach them. Today is about when parallel workers should talk to each other. Here's the one sentence. Subagents are focused workers that report results back to the agent that started them, while an agent team is a set of full sessions that share a task list and message each other directly. That helps when workers must react to each other's findings, but it costs far more tokens and adds coordination risk.

MAYA: Report back, versus talk to each other. Okay.

BEN: Let's start with the two underlying patterns. "Building effective agents" names them. The first is parallelization: "breaking a task into independent subtasks run in parallel", which they call sectioning, or running the same task several times for diverse outputs, which they call voting.

MAYA: And the second?

BEN: Orchestrator-workers. Quote, "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results". It's useful when "you can't predict the subtasks needed".

MAYA: So where do subagents and teams fit?

BEN: Subagents are the orchestrator-workers pattern, built into a coding agent. Agent teams go a step further. The workers also coordinate with each other, not only through the orchestrator.

MAYA: Okay. Quick refresher on subagents, then.

BEN: In Claude Code, "each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions". It doesn't see your conversation. Claude writes a delegation message, the subagent works, and "only the summary" comes back to the main context.

MAYA: And that's good for keeping the main context clean.

BEN: That's one job. Test runs, log searches and documentation lookups produce verbose output the parent doesn't need. The other job is running separable work in parallel, like several subagents exploring different parts of a codebase at once.

MAYA: How do I make my own?

BEN: A Markdown file with YAML front matter. A name, a description, and optionally tools and a model. It goes in the {.claude/agents|dot claude slash agents} folder of your project, or the same folder in your home directory. Restricting the tools is how you make a read-only reviewer. And setting the model to Haiku on a simple subagent is one of Anthropic's cost tips.

MAYA: So far, subagents sound great. What's the catch?

BEN: The coordination limit. Subagents coordinate through their caller. When one subagent's result matters to another, either the parent relays it, or the subagents leave notes in files.

MAYA: [curious] Notes in files. That sounds a bit clunky.

BEN: Anthropic's research system does it on purpose. Subagents "store their work in external systems, then pass lightweight references back to the coordinator". That avoids a "game of telephone", where details get lost in each retelling.

MAYA: Okay, that makes sense.

BEN: But the same post names the cost of central coordination. Its lead agent runs subagents synchronously. So "the lead agent can't steer subagents, subagents can't coordinate", and the whole system can stall while one subagent finishes.

MAYA: So the boss becomes the bottleneck.

BEN: Exactly. Now, lately the line has blurred a little. Claude Code can name a subagent, and named agents can be resumed or messaged with a {SendMessage|send message} tool. A subagent with that tool sees a roster of other named agents.

MAYA: Wait, so subagents can talk to each other now? Isn't that a team?

BEN: [laughs] Good pushback. Not quite. There's still no shared task list, and the main agent still manages all the work.

MAYA: Okay. So what is an agent team, exactly?

BEN: Claude Code's docs describe it as multiple Claude Code instances, where "one session acts as the team lead, coordinating work, assigning tasks, and synthesizing results. Teammates work independently, each in its own context window, and communicate directly with each other." And you can message any teammate yourself, without going through the lead.

MAYA: What are the moving parts?

BEN: Four. The team lead is the main session. It spawns teammates, creates tasks, coordinates and synthesises results. And it's fixed for the session's lifetime.

MAYA: Then the teammates.

BEN: Separate Claude Code instances. Each loads the project's {CLAUDE.md|CLAUDE dot M D}, MCP servers and skills, plus its spawn prompt. But they don't inherit the lead's conversation.

MAYA: And the other two?

BEN: The task list. Shared work items, each pending, in progress or completed. Tasks can depend on others. Teammates claim tasks themselves, with file locking so two can't claim the same one. And the mailbox: messages between agents, delivered automatically. When a teammate finishes, it notifies the lead.

MAYA: Where does all that live? In my project?

BEN: No, and that surprises people. It lives in your home directory. The team config, the mailboxes and the tasks are all in folders under {~/.claude|dot claude in your home directory}. The docs say explicitly that a teams file inside your project's {.claude|dot claude} folder is not recognised as configuration.

MAYA: Can you put the two side by side for me?

BEN: The docs have a comparison table. Context: both get their own window, but subagent results return to the caller, while teammates are fully independent. Communication: subagents return a result, and named subagents can also message. Teammates message each other directly.

MAYA: And coordination?

BEN: With subagents, the main agent manages all the work. Teams have self-coordination plus a shared task list. Subagents are best for focused tasks where only the result matters. Teams for work that needs discussion and collaboration. And token cost: lower for subagents, because results are summarised back. Higher for teams, because each teammate is a separate instance.

MAYA: Okay. How do I turn teams on?

BEN: As of October 2026 they're experimental and off by default. They launched as a research preview with Claude Opus 4.6, in February 2026. You set {CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS|Claude code experimental agent teams} to one, either in your shell, or in your settings file under env.

MAYA: Anything else I need to know before I try?

BEN: A few things. Teams need an interactive session. In non-interactive mode, with {-p|dash P}, and in the Agent SDK, Claude doesn't spawn teammates. You ask for a team in plain language, naming the roles. Claude may use subagents instead, so ask explicitly for an agent team if it does.

MAYA: Okay.

BEN: And one surprise. With teams enabled, a subagent that Claude names launches as a teammate. So teams can form when you didn't ask for one. Set the variable to zero to switch back.

MAYA: [laughs] Accidental teams. What does it look like on screen?

BEN: The default in-process display works in any terminal. Split panes need {tmux|T mux} or {iTerm2|eye term two}. The docs list Windows Terminal, VS Code's terminal and Ghostty as unsupported for split panes.

MAYA: Can I keep some control over what teammates do?

BEN: Yes. Plan approval: if the lead is in plan mode, a teammate it spawns plans read-only before implementing. And hooks like {TeammateIdle|teammate idle} and {TaskCompleted|task completed} can enforce quality gates.

MAYA: And when I'm done?

BEN: Ask the lead to shut a teammate down. The teammate can approve or reject, with an explanation, and it finishes its current tool call first.

MAYA: [laughs] It can reject being shut down?

BEN: With an explanation, yes. That's how the docs describe it.

MAYA: Okay, so when are teams actually worth it?

BEN: Anthropic's launch note says teams are "best for tasks that split into independent, read-heavy work like codebase reviews". And the docs list four strong cases. First, research and review. The docs say "a single reviewer tends to gravitate toward one type of issue at a time." So you give teammates distinct lenses, like security, performance and test coverage, and the lead synthesises.

MAYA: Second?

BEN: Debugging with competing hypotheses. A single agent "tends to find one plausible explanation and stop looking". Teammates who try to disprove each other's theories counter that anchoring. And, quote, "the theory that survives is much more likely to be the actual root cause".

MAYA: [thoughtful] I like that one. It's like having colleagues who argue with you.

BEN: That's the spirit of it. Third, new modules or features, where each teammate owns a separate piece. And fourth, cross-layer changes across frontend, backend and tests, each owned by a different teammate.

MAYA: Anything that really needs the live messaging?

BEN: A common pattern is find-and-fix. One teammate reviews and sends findings one at a time, while another fixes them. So review and repair overlap, instead of running one after the other. But it only works if the fixer's edits don't collide with other work.

MAYA: And the task dependencies we mentioned?

BEN: They matter too. A teammate that sets up packages can block the page builders until it finishes. Claude Code unblocks the dependent tasks automatically when the blocking task completes.

MAYA: And when do teams not pay off?

BEN: The docs are direct. Quote, "For sequential tasks, same-file edits, or work with many dependencies, a single session or subagents are more effective." Anthropic's research team reached the same conclusion for multi-agent systems in general. Domains that require "all agents to share the same context", or that involve "many dependencies between agents", are poor fits. And "most coding tasks involve fewer truly parallelizable tasks than research".

MAYA: Let's talk about the cost. How much more are we talking?

BEN: Two numbers. Anthropic's multi-agent research system used about fifteen times the tokens of a chat. And token usage alone explained eighty percent of the performance variance on its browsing benchmark. They say multi-agent designs pay off only for "tasks where the value of the task is high enough".

MAYA: And for Claude Code teams specifically?

BEN: The cost guide says teams use "approximately 7x more tokens than standard sessions when teammates run in plan mode". Its tips: use Sonnet for teammates, keep teams small, keep spawn prompts focused, and shut teammates down when they're done.

MAYA: Seven times. Okay, that makes me want to use them carefully. What are the best practices?

BEN: Seven of them, from the docs and the research lessons. One: give enough context in the spawn prompt. Teammates don't see the lead's history. Each needs, quote, "an objective, an output format, guidance on the tools and sources to use, and clear task boundaries".

MAYA: Two?

BEN: Choose a small team. Start with three to five teammates. "Three focused teammates often outperform five scattered ones." Three: size tasks sensibly. Too small, and coordination costs more than it saves. Too large, and teammates work too long without check-ins. About five to six tasks per teammate keeps everyone busy.

MAYA: Four?

BEN: Avoid file conflicts. "Two teammates editing the same file leads to overwrites." So split work by file ownership. Five: tell the lead to wait. The lead sometimes starts doing tasks itself. So you say, "Wait for your teammates to complete their tasks before proceeding."

MAYA: [laughs] The manager who can't stop doing the work.

BEN: [laughs] Exactly. Six: monitor and steer. Redirect teammates that drift. A team left alone too long wastes effort. And seven: start with research and review, which have clear boundaries and no write conflicts, before trying parallel implementation.

MAYA: Any known limitations?

BEN: A few. In-process teammates aren't restored on {/resume|slash resume}. Task status can lag. Shutdown can be slow. One team per session, no nested teams, and the lead can't be changed.

MAYA: Okay, give me the decision rule.

BEN: A single agent when the task is small or sequential, or most steps touch the same files. Subagents when the subtasks are separable: each can be done and checked without the others, and the parent only needs the results. And an agent team when workers need each other's findings while they work. Competing hypotheses, multi-lens review, or modules with clear owners that must stay consistent.

MAYA: So, does the worker need to hear from the others while it's working? If yes, team. If it just needs to hand in a result, subagent.

BEN: That's the heart of it. And remember: each extra worker is a full context window you pay for. So parallelism trades tokens for time and breadth.

MAYA: What lasts, and what will date?

BEN: The durable part. Workers that can only report upward make the coordinator a bottleneck. Direct messaging removes it, but adds cost and collision risk. Parallel work helps only when tasks are independent, clearly scoped and sensibly sized. Independent investigators who try to disprove each other resist anchoring better than one agent. And token use scales with the number of workers, so use multiple agents where the task's value justifies it.

MAYA: And what will date?

BEN: As of October 2026: agent teams as an experimental Claude Code feature, with the enabling variable, the display modes and the limitations. Named subagents and {SendMessage|send message}. And the token figures, about seven times and about fifteen times, plus the ninety point two percent research-eval result from Anthropic's research system, which we met in chapter five.

MAYA: Let me recap, and you correct me. Subagents work in their own window and report a summary back to their caller. The caller manages everything, which keeps things cheap but can make it a bottleneck. Agent teams are full sessions with a lead, a shared task list and mailboxes, so teammates can message each other directly.

BEN: Good so far.

MAYA: Teams are best for read-heavy work like multi-lens review, or competing hypotheses in debugging. They're bad for sequential work, same-file edits, and lots of dependencies. And they cost a lot more, so keep them small, brief them well, and split by file ownership.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You have a bug with an unclear cause. Why might a team beat a single agent here?

PAUSE: 5

MAYA: Because a single agent tends to find one plausible explanation and stop looking. Teammates who try to disprove each other's theories counter that anchoring, and the theory that survives is much more likely to be the real root cause.

BEN: Perfect. Question two. You need to run the test suite, search some logs and look up documentation, and you only need the conclusions. Subagents or a team?

PAUSE: 5

MAYA: Subagents. The tasks are separable, nobody needs the others' findings while working, and the parent only needs the results. A team would just cost more tokens.

BEN: Exactly. And last one. You're writing a spawn prompt for a teammate. What does it need, and why?

PAUSE: 5

MAYA: An objective, an output format, guidance on the tools and sources to use, and clear task boundaries. Because teammates don't see the lead's conversation history.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the first: list five recent multi-step tasks, and label each one single agent, subagents or team, with one sentence of reasoning.

BEN: And then write a proper delegation brief, run a three-lens review with a team and with subagents and compare tokens with {/usage|slash usage}, try competing hypotheses on a bug, and draw file ownership for a planned feature. Next time, in chapter twenty-four, we look at the advisor strategy. A cheaper model that does the work, and consults a stronger one only at the decisive points.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-four.
