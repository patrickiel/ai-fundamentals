<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 06: Agent instruction files

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm still the one learning here.

BEN: And I'm Ben. Today is chapter six: agent instruction files. How to write a {CLAUDE.md|CLAUDE dot M D} or an {AGENTS.md|AGENTS dot M D} that actually changes how the agent behaves.

MAYA: Last time, in chapter five, we talked about work that outlasts one context window. And {CLAUDE.md|CLAUDE dot M D} kept coming up, as the file that loads every session and comes back after compaction.

BEN: Right. So today we ask what should actually go in it. Here's the one sentence. An instruction file should hold only what changes the agent's default behaviour: how to handle ambiguity, how much to change, how to prove the work is done, and which tools to use. Keep it short and scoped. And anything that must never happen is enforced by permissions and hooks, not by prose.

MAYA: [curious] Okay, so not a manual for the whole project.

BEN: Not at all. Anthropic has a test for every single line, and I love how short it is. [slowly] "Would removing this cause Claude to make mistakes?" If the answer is no, cut it.

MAYA: Let's start with the basics. Which agents read which file?

BEN: Most coding agents read a project-level instruction file at the start of each session. Claude Code reads {CLAUDE.md|CLAUDE dot M D}. Codex, Cursor, Gemini CLI, GitHub Copilot's coding agent and many others read {AGENTS.md|AGENTS dot M D}. That's an open format, now stewarded by the Agentic AI Foundation under the Linux Foundation, and it's described as "a README for agents".

MAYA: And if my repository only has an {AGENTS.md|AGENTS dot M D}, does Claude Code ignore it?

BEN: As of October 2026, no. Claude Code also reads {AGENTS.md|AGENTS dot M D} when a repository has no {CLAUDE.md|CLAUDE dot M D}.

MAYA: So why does this one file matter so much?

BEN: Because it loads every time, so it shapes every task. And remember the last two chapters: performance degrades as the context fills. Anthropic's best-practices guide says that's the main constraint. So a bloated file hurts twice. It takes up space, and it buries the rules that matter. In the guide's words: "Bloated {CLAUDE.md|CLAUDE dot M D} files cause Claude to ignore your actual instructions!"

MAYA: With an exclamation mark.

BEN: [laughs] With an exclamation mark. They mean it.

MAYA: Okay, but if I write a rule in there, the agent follows it, right? That's the point of the file.

BEN: That's the misconception I want to clear up early. The file has a ceiling. Claude Code delivers {CLAUDE.md|CLAUDE dot M D} content as a user message after the system prompt. And the docs say, quote, "there's no guarantee of strict compliance, especially for vague or conflicting instructions."

MAYA: Hmm. So it's more like advice than a law.

BEN: Exactly. Even Andrej Karpathy noted that the failure modes he saw persisted, quote, "despite a few simple attempts to fix it via instructions in {CLAUDE.md|CLAUDE dot M D}." A good file reduces how often you have to correct the agent. It doesn't replace reviewing the diff.

MAYA: You mentioned failure modes. What does the agent get wrong by default?

BEN: Karpathy wrote a post in January 2026, after weeks of mostly agent-driven coding. He said the mistakes weren't syntax errors. They were, quote, "subtle conceptual errors that a slightly sloppy, hasty junior dev might do."

MAYA: [laughs] Ouch. Okay, what kind of errors?

BEN: He lists four. First: "The models make wrong assumptions on your behalf and just run along with them without checking."

MAYA: Number two?

BEN: Confusion. Quote: "They don't manage their confusion, they don't seek clarifications, they don't surface inconsistencies, they don't present tradeoffs, they don't push back when they should."

MAYA: That's a long list of don'ts.

BEN: It is. Third, overcomplication. "They really like to overcomplicate code and APIs, they bloat abstractions, they don't clean up dead code after themselves." He describes one construction of a thousand lines that got cut down to a hundred, once he asked whether a simpler approach would work.

MAYA: Ten times smaller. Just from asking.

BEN: Just from asking. And fourth, side effects. They "sometimes {change/remove|change or remove} comments and code they don't like or don't sufficiently understand as side effects, even if it is orthogonal to the task at hand."

MAYA: So it tidies up things I never asked it to touch.

BEN: Right. But Karpathy also names the main lever. Quote: "LLMs are exceptionally good at looping until they meet specific goals." And then: "Don't tell it what to do, give it success criteria and watch it go."

MAYA: Let me summarise. Wrong assumptions, hidden confusion, bloat, and changes to unrelated code. And the fix is giving it goals it can check, not step-by-step orders.

BEN: That's a good summary. And someone turned those observations into rules. There's a community repository called andrej karpathy skills. It packages them as a single {CLAUDE.md|CLAUDE dot M D}, also published as a Claude Code plugin. One important detail: it's derived from Karpathy's post. It wasn't written by him.

MAYA: Good to know. And it has four rules?

BEN: Four rules, each targeting one of those habits. Rule one: think before coding. The wording is, "Don't assume. Don't hide confusion. Surface tradeoffs." State your assumptions. When a request has several readings, present them instead of silently picking one. If there's a simpler approach, say so, and push back where it's warranted. And if something is unclear, stop and ask.

MAYA: Why is that the first rule?

BEN: Because misunderstandings are cheapest to fix before any code exists. Claude Code's own guidance points the same way. For larger features, have the agent interview you and write a spec before it implements anything. That's chapter eight.

MAYA: Rule two?

BEN: Simplicity first. "Minimum code that solves the problem. Nothing speculative." No features beyond the request. No abstractions for code that's used once. No configurability nobody asked for. No error handling for impossible scenarios.

MAYA: [thoughtful] But isn't extra error handling just being careful?

BEN: For scenarios that can actually happen, sure. The rule is about impossible ones. The file adds a heuristic: "If you write two hundred lines and it could be fifty, rewrite it." And a check: "Would a senior engineer say this is overcomplicated?"

MAYA: And the reason?

BEN: Every extra line has to be read, reviewed and worked around by every later change. By you, and by the agent.

MAYA: Rule three.

BEN: Surgical changes. "Touch only what you must. Clean up only your own mess." Don't "improve" nearby code, comments or formatting, and match the existing style. If the agent notices unrelated dead code, it should mention it, not delete it. And it removes only the imports, variables or functions that its own change made unused.

MAYA: Is there a test for that one too?

BEN: There is. "Every changed line should trace directly to the user's request."

MAYA: [curious] But if the agent spots something genuinely broken next door, isn't it a waste not to fix it?

BEN: Here's the reasoning. Small diffs are reviewable. And code that looks wrong may exist for reasons nobody wrote down. You don't lose the information, though. You still hear about the issue, and you decide later.

MAYA: Okay, that's fair. And rule four?

BEN: Goal-driven execution. "Define success criteria. Loop until verified." You turn imperative tasks into checkable goals. "Add validation" becomes "write tests for invalid inputs, then make them pass." "Fix the bug" becomes "write a test that reproduces it, then make it pass." And for multi-step work, state a short plan with a verification check for each step.

MAYA: That's Karpathy's lever again. Success criteria.

BEN: Exactly. A passing check is an objective stopping point. Anthropic's guide puts it well: "Claude stops when the work looks done. Without a check it can run, 'looks done' is the only signal available."

MAYA: What about UI work? A unit test can't tell me if a page looks right.

BEN: Right. For that, give the agent a browser it can drive and take screenshots with. Karpathy suggests putting the agent, quote, "in the loop with a browser {MCP|M C P}." Chapter seven covers verification in depth.

MAYA: Any downside to these four rules?

BEN: The repository is honest about it. The rules "bias toward caution over speed." For trivial tasks, you should use judgement.

MAYA: So: think first, keep it simple, change only what's needed, and define done as a check.

BEN: That's the behaviour half. The other half of the file is project-specific facts. Things the agent simply can't guess.

MAYA: Like what?

BEN: Let's start with how you get a file at all. Running an init command produces a starting file. Anthropic suggests you then, quote, "refine from there with instructions Claude wouldn't discover on its own." And the best-practices guide has a list of what to include and what to exclude.

MAYA: Go on.

BEN: Include: bash commands Claude can't guess. Code style rules that differ from the defaults. Testing instructions and your preferred test runners. Repository etiquette, like branch naming and pull request conventions. Architectural decisions specific to the project. Environment quirks, such as required environment variables. And common gotchas.

MAYA: And exclude?

BEN: Anything Claude can figure out by reading the code. Standard language conventions. Detailed API documentation, link to it instead. Information that changes often. Long explanations or tutorials. File-by-file descriptions of the codebase. And self-evident practices, like "write clean code".

MAYA: [laughs] I've definitely written "write clean code" in a prompt.

BEN: Everyone has. Two practical consequences. First, list tools only where they differ from the default. "This project uses {pnpm|P N P M}, not {npm|N P M}" earns its line. So does telling the agent to use a command-line tool like {gh|G H} for GitHub work. The guide calls that the most context-efficient way to reach external services.

MAYA: And the second?

BEN: Write instructions concrete enough to verify. "Use two-space indentation" instead of "format code properly". "Run {npm test|N P M test} before committing" instead of "test your changes".

MAYA: Does {AGENTS.md|AGENTS dot M D} suggest anything similar?

BEN: A similar set of sections. A project overview, build and test commands, code style, testing instructions, security considerations, and pull request guidelines.

MAYA: Okay. Now the safety part. If I write "never force-push" in the file, I'm covered?

BEN: [sighs] That's the tempting one. Something like "never force-push or run {rm -rf|R M dash R F} without asking." That line helps, but it's advisory. Claude Code's docs are explicit. Quote: "To block an action regardless of what Claude decides, use a {PreToolUse|pre tool use} hook." And: "Unlike {CLAUDE.md|CLAUDE dot M D} instructions which are advisory, hooks are deterministic."

MAYA: Deterministic meaning it always happens, no matter what the model thinks.

BEN: Exactly. Permission rules work the same way. Deny rules block in every mode. And ask rules force a confirmation prompt.

MAYA: So how do I split it?

BEN: Settings are the enforced layer. Deny or ask rules for destructive commands: force-push, hard reset, recursive delete, production deploys. And you share them through the project's committed settings file. The instruction file is the guidance layer. It explains the intent behind those rules, and the judgement calls a rule list can't capture. For example: "if you're unsure whether a command is reversible, ask."

MAYA: Rules in settings, reasons in the file.

BEN: Nicely put.

MAYA: Next question. Once I have a good file, do I just leave it?

BEN: No. It should grow from real mistakes. Boris Cherny, who created Claude Code, describes how his team does it. Quote: "Anytime we see Claude do something incorrectly we add it to the {CLAUDE.md|CLAUDE dot M D}, so Claude knows not to do it next time."

MAYA: Is that file shared, or is it just his?

BEN: Shared. It's checked into git, and the team adds to it several times a week. And during code review, he tags Claude on colleagues' pull requests, so the lesson gets added as part of the pull request.

MAYA: [laughs] That's neat. The review itself updates the rules.

BEN: Right. Anthropic's memory docs also give triggers for adding a line. Claude makes the same mistake a second time. A code review catches something Claude should have known. Or you type the same correction you typed last session.

MAYA: That last one is very relatable.

BEN: [laughs] It is. And Claude Code also keeps an auto memory, where Claude records corrections and preferences on its own.

MAYA: So the file only ever grows?

BEN: Adding is only half of it. The guide says to "treat {CLAUDE.md|CLAUDE dot M D} like code: review it when things go wrong, prune it regularly, and test changes by observing whether Claude's behavior actually shifts."

MAYA: Let's talk length. How long is too long?

BEN: Anthropic's target is under two hundred lines per {CLAUDE.md|CLAUDE dot M D}, because, quote, "longer files consume more context and reduce adherence."

MAYA: Can I split a long file into smaller ones with imports?

BEN: Imports help you organise a long file, but they don't reduce its cost, because imported files also load at launch. And here's a useful signal. If the agent keeps ignoring a rule, the file is probably too long, and the rule is getting lost.

MAYA: So what do I do with rules that only matter for part of the code?

BEN: Path-scoped rules. Claude Code loads markdown files from a rules folder inside the {.claude|dot claude} directory. If a rule file has a paths field in its frontmatter, with glob patterns, it loads only when Claude works with matching files.

MAYA: Give me an example.

BEN: The chapter shows a rule file scoped to the TypeScript files under {src/api|source slash API}. It has one rule: all endpoints validate input and use the standard error format. That rule only shows up when the agent is working on the API.

MAYA: And for a monorepo, with lots of packages?

BEN: Nested instruction files serve each package. In Claude Code, files in subdirectories load on demand when Claude works in that directory. And a setting called {claudeMdExcludes|claude M D excludes} can skip other teams' files. With {AGENTS.md|AGENTS dot M D}, quote, "the closest one takes precedence." Keep only system-wide rules at the root.

MAYA: Any tips on how to write the rules themselves?

BEN: Group them under headers and bullets. Remove contradictions, because when two instructions conflict, the agent "may pick one arbitrarily". If one rule keeps being skipped, mark that one line as important. But if you emphasise many lines, none of them stands out.

MAYA: [laughs] Like highlighting a whole textbook page.

BEN: Exactly like that. A common pattern is to open with a two- or three-line project overview, then the rules that matter most.

MAYA: The chapter has a skeleton file. Can you walk me through it?

BEN: Sure. It's an illustrative example, not taken from a source. A shared-notes web app called Acme Notes, with a Next.js frontend, a Node API and Postgres. That's the overview. Then a commands section: use {pnpm|P N P M}, not {npm|N P M}, plus the commands for dev, tests and type checking, and use the {gh|G H} tool for GitHub.

MAYA: Then the behaviour rules?

BEN: A "how to work" section that's basically the four rules in five lines. State assumptions, and if the request is ambiguous, list the readings and ask. Minimum code, no speculative features. Change only what the task needs, and mention unrelated issues instead of fixing them. Turn each task into a check and iterate until it passes. And "done" means tests, type check and lint pass, with UI changes checked in the browser.

MAYA: And gotchas?

BEN: Two. Migrations are generated, never edited by hand. And the area rules live in the rules folder, one for the API and one for the UI. And the destructive-command limits live in the settings file, not in the instruction file.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Instructions should correct defaults the agent gets wrong, not repeat what it can infer. Clarify before building, make the smallest change that solves the problem, and define "done" as something checkable. Prose is advisory, so enforce must-never-happen rules with deterministic controls. Load rules only where they apply, because irrelevant instructions cost attention. And grow the file from observed mistakes, and prune it like code.

MAYA: And what will date?

BEN: As of October 2026: the file names and loading rules, including the rules folder with paths. Claude Code's fallback to {AGENTS.md|AGENTS dot M D}. The two-hundred-line target. Auto memory. The plugin packaging of the community guidelines. And the list of tools that read {AGENTS.md|AGENTS dot M D}.

MAYA: Let me try to sum up, and you correct me. The instruction file loads every session, so every line costs context. Keep only lines whose removal would cause mistakes, and aim for under two hundred. The behaviour rules are think before coding, simplicity first, surgical changes, and goal-driven execution.

BEN: Good so far.

MAYA: Then add the project facts the agent can't guess, like non-default tools and gotchas. The file is advisory, so hard safety rules go into permission rules and hooks. Grow it from real mistakes, prune it, and scope area rules to the paths where they apply.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You're pruning an old instruction file. What question do you ask about each line?

PAUSE: 5

MAYA: Would removing this cause Claude to make mistakes? If not, delete it.

BEN: Perfect. Question two. You never want the agent to force-push. Where should that rule live, and why?

PAUSE: 5

MAYA: In the settings, as a deny or ask permission rule, or in a hook. Because the instruction file is advisory, and those are enforced no matter what the model decides. The file can still explain the intent.

BEN: Exactly. And last one. Your task for the agent is "fix the bug". How do you turn that into goal-driven execution?

PAUSE: 5

MAYA: Write a test that reproduces the bug, then make it pass. The passing test is the stopping point.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with pruning: go through an existing file, ask that question for every line, delete the rest, and count what went.

BEN: Then try the four rules before and after on the same ambiguous request, move one "never do X" line into a permission rule, scope one area into a path-scoped rule file, and log your corrections for a week. Next time, in chapter seven, we look at verification first. Giving the agent a check it can run, which might be the single biggest improvement of all.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter seven.
