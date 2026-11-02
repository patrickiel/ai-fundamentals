<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 27: Running agents in parallel

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter twenty-seven: running agents in parallel. Isolation, handoffs and orchestration.

MAYA: Last time, in chapter twenty-six, we built a harness with a planner, a generator and a separate evaluator. One agent doing, another judging.

BEN: Right. Today we go wide instead. Not several roles on one task, but several sessions running at the same time, on one codebase.

MAYA: [curious] Which sounds great. Five agents, five times the work, right?

BEN: [laughs] That's the hope. So here's the one sentence. Running several agent sessions at once keeps your own time fully used. But sessions don't share what they know, while they do share files, branches, ports and dependencies. So parallel work holds up only if you plan independent tasks, hand off context on purpose, and isolate each session in layers.

MAYA: So they share all the wrong things, and none of the right things.

BEN: [laughs] That's a fair summary of the problem. Let's start with why you'd do it at all.

MAYA: Because waiting is boring?

BEN: Pretty much. A single agent on a long task leaves you waiting. With several sessions, you read what one just returned, send it the next instruction, and move on while the others keep working. Claude Code's documentation now lists several built-in ways to do this, from subagents inside one conversation to background sessions you dispatch and check on later.

MAYA: And the cost?

BEN: Tokens. In the docs' words: "Running several sessions or subagents at once multiplies token usage."

MAYA: Okay. What goes wrong?

BEN: The catch is that parallel sessions fail quietly. Every session looks busy, and every output reads fine. But underneath, they can overwrite each other's work, and you often don't notice until something breaks.

MAYA: [sighs] The worst kind of failure. Let's list the problems.

BEN: The chapter splits them into general ones, for any kind of work, and software-specific ones. The first general problem is simply losing track. Which session was doing what, and where did the important one go?

MAYA: Guilty. I have terminal tabs I'm afraid to close.

BEN: [laughs] Second: no shared memory. Each session is its own conversation, with its own context window. Instruction files like {CLAUDE.md|CLAUDE dot M D} are loaded when a session starts. And a session doesn't see another session's discoveries unless they're written down or passed on.

MAYA: Wait, really? If one session figures out why a test is flaky, the others don't know?

BEN: Not unless someone writes it down or sends it. As of October 2026, Claude Code has a cross-session messaging feature for passing findings between sessions you run yourself. But nothing flows on its own.

MAYA: Okay. Third?

BEN: Overwrites. Two agents editing the same file clobber each other. Claude Code's agent-teams guidance is blunt: two teammates editing the same file "leads to overwrites". So break the work up so each owns a different set of files.

MAYA: And the software-specific problems?

BEN: Three of them. Branch fights: one session switches branches in the same checkout another session is using. Port collisions: several dev servers try to bind the same port, like {localhost:3000|localhost three thousand}. And shared dependencies and state: one session upgrades a package, changes the build, or migrates the local database, and every other session breaks without knowing why.

MAYA: That last one sounds nasty, because the broken session has no idea what happened.

BEN: Exactly. Now the fixes. Four general ones first. Number one: name, resume and catch up on sessions. Treat sessions as long-lived, named work units you return to, not throwaway chats. Claude Code's guide says to name sessions and "treat them like branches: each workstream gets its own persistent context".

MAYA: How, concretely?

BEN: As of October 2026, a few commands. {claude --continue|claude dash dash continue} reopens the most recent session in that folder. {claude --resume|claude dash dash resume} lets you pick from a list, or by name. You name a session with {claude -n|claude dash N} when you start it, or with {/rename|slash rename} later. {/export|slash export} copies a conversation out, to the clipboard or a text file. And if you want to try another approach without losing this one, there's {/branch|slash branch}, or the {--fork-session|dash dash fork session} flag.

MAYA: Okay, so sessions get names, like branches. Fix number two?

BEN: Plan first, then split into independent tasks. Claude Code's recommended flow is explore, plan, implement, commit. With plan mode for the first two steps, so Claude reads and proposes without editing anything.

MAYA: And for parallel work specifically?

BEN: Ask the plan for tasks that can be done independently, and then check the result. Is there any file two tasks would both write? If so, give that file one owner, and tell the other sessions they may read it but not write it.

MAYA: [thoughtful] One owner per file. That's simple. Does task size matter?

BEN: It does. The agent-teams guidance describes good tasks as "self-contained units that produce a clear deliverable, such as a function, a test file, or a review". Too small, and coordination costs more than the work. Too large, and an agent runs too long without a check-in.

MAYA: Fix number three?

BEN: Hand off with a compact document, not a chat dump. Pasting a whole conversation into a new session carries all its noise, and fills the new context.

MAYA: Which is the context rot problem from chapter four, basically.

BEN: Same family, yes. The better pattern is a handoff document. Claude Code's guide recommends having Claude write a spec, then starting, quote, "a fresh session to execute it. The new session has clean context focused entirely on implementation."

MAYA: What makes a spec good enough to hand off?

BEN: The guide lists it. It's self-contained. It names the files and interfaces involved. It states what's out of scope. And it ends with an end-to-end verification step.

MAYA: And with several parallel workers, does each get a copy of the whole plan?

BEN: Better not to copy it. A planning session can write one handoff per task, each pointing to the shared plan file instead of copying it. Every worker starts clean, with the same shared ground truth.

MAYA: Point to it, don't paste it. Got it. Fix four?

BEN: Launch and monitor background sessions. As of October 2026, Claude Code's agent view, which is a research preview, is one screen for background sessions. The docs describe it as "what's running, what needs your input, and what's done."

MAYA: How do I get there?

BEN: You open it with {claude agents|claude agents}, and type a prompt to dispatch a new background session. Or start one from the shell with {claude --bg|claude dash dash B G} and the task. Or move the current session to the background with {/bg|slash B G}. And note: background sessions "consume your subscription usage the same as interactive sessions".

MAYA: Let me summarise the general part. Name sessions so I can find them. Plan first, split into independent tasks, one owner per file. Hand off with a short spec, not a chat dump. And use a dashboard to watch the background ones.

BEN: Exactly right. Now the development-specific fixes. The first one is git worktrees.

MAYA: I've heard the word, but I've never used one. What is it?

BEN: The git documentation says a worktree lets one repository "support multiple working trees, allowing you to check out more than one branch at a time". Each linked worktree has its own folder, its own {HEAD|head} and its own index. And it shares everything else with the main repository, including history and refs.

MAYA: So it's like a clone, but lighter?

BEN: That's the practical effect. You don't clone the project over and over. And there's a nice safeguard: by default, git refuses to check out a branch that's already checked out in another worktree.

MAYA: What does it look like to use?

BEN: The chapter shows a short example. You run git worktree add, with a sibling folder and a new branch name. Say, one folder for an auth feature on its own branch, and another for billing on its own branch. Git worktree list shows them all. And git worktree remove cleans one up once the work is merged.

MAYA: And then each session gets its own folder.

BEN: Right. Give each session its own worktree, and they can't overwrite each other's files or fight over the checked-out branch. And Claude Code builds this in, as of October 2026. {claude --worktree|claude dash dash worktree}, or {-w|dash W}, with a name, creates a worktree under {.claude/worktrees|dot claude slash worktrees} on a new branch, and starts the session there.

MAYA: What about subagents?

BEN: Subagents can each get a temporary worktree, by setting {isolation: worktree|isolation colon worktree} in their definition. Sessions dispatched from agent view move into their own worktree before editing files. And {/batch|slash batch} splits one large change across five to thirty worktree-isolated subagents.

MAYA: [curious] Okay, so worktrees solve everything?

BEN: No, and the docs have two practical notes. First, a worktree is a fresh checkout. So you have to install dependencies there. And gitignored files, like {.env|dot env}, are missing, unless you list them in a {.worktreeinclude|dot worktree include} file.

MAYA: Oh, that would trip me up for sure. My app wouldn't start, and I'd have no idea why.

BEN: [laughs] Very common. And second, worktrees isolate files only. A dev server in one worktree can still collide on a port with a server in another. And anything installed globally is shared.

MAYA: So worktrees fix the code conflicts, but not the environment conflicts.

BEN: Exactly. Hold that thought. First, another layer: automating who gets which task. Worktrees isolate the code, but you're still the one assigning tasks. An orchestrator automates that step.

MAYA: Is there a real example?

BEN: OpenAI's Symphony is one open-source example. Its repository describes it as, quote, "a low-key engineering preview for testing in trusted environments", as of October 2026. It watches an issue tracker, Linear in the reference implementation, and starts a coding agent for each eligible issue.

MAYA: So I label an issue, and an agent just picks it up?

BEN: That's the idea. Its specification describes a long-running daemon, meaning a background process, with an orchestrator, a tracker poller, a workspace per issue, concurrency limits, ten agents by default, and retries with exponential backoff.

MAYA: How does it know the work is done?

BEN: Agents report "proof of work", such as CI status and pull request review feedback. And they land the pull request once the work is accepted.

MAYA: And security? An agent per issue sounds like a lot of trust.

BEN: Good instinct, and the spec is explicit about what it doesn't do. It does not mandate, quote, "strong sandbox controls beyond what the coding agent and host OS provide". Each implementation "defines its own trust boundary".

MAYA: So it's on me to isolate it properly.

BEN: Yes. And the pattern travels beyond any one tool. Turn the backlog into a queue. Use a tracker state or a label as the trigger. Give each task its own branch and workspace. And use the pull request as the review gate.

MAYA: Okay, now back to the thought I was holding. The environment.

BEN: Worktrees and orchestrators leave the environment shared. Installed packages, running services, ports. Fixing that takes a sandbox: a disposable, isolated environment where the agent can install, build and run, without touching your machine or the other sessions.

MAYA: Like a container?

BEN: Containers, VMs and cloud sessions all provide that kind of isolation. The chapter's example is Docker Sandboxes, as one implementation. It runs coding agents in {microVMs|micro V Ms}, and "each sandbox gets its own Docker daemon, filesystem, and network".

MAYA: How do you use it?

BEN: You start a sandbox from a project directory with the {sbx|S B X} command-line tool. Network access follows a policy you choose: open, balanced, or locked down. By default the project directory is mounted read-write, so the agent's edits appear in your working tree as they happen.

MAYA: [thoughtful] Hmm, but then several agents on one repository would be editing the same files again.

BEN: Exactly the right worry. For that case there's clone mode. It gives each agent a private clone inside its sandbox instead.

MAYA: Which product should I use?

BEN: The chapter's point is that the vendor matters less than the property. Each parallel task gets its own dependencies, processes and ports.

MAYA: Okay. How does it all fit together in the end?

BEN: A common end state is to let one session orchestrate. It plans the feature, starts one isolated worker per subtask, in a worktree, a sandbox or a cloud session. It monitors them, reviews and merges each result, and cleans up the workspace once a task is confirmed done.

MAYA: That sounds familiar.

BEN: It's the orchestrator-workers shape from chapter twenty-five. Claude Code's own {/batch|slash batch} and dynamic workflows are packaged versions of it. For example, migrating many files while, quote, "working on each file in its own isolated copy", and verifying each result.

MAYA: And then I just trust the merged result?

BEN: [laughs] No. As with any orchestration, review the merged output yourself. Ideally with a fresh-context reviewer.

MAYA: The doer and the judge again, from last time.

BEN: Same principle. And the chapter sums all of this up as a layered picture. Each problem has a layer that fixes it. Losing track of sessions: names, resume, and a dashboard like agent view. Stale or noisy context: plan first, and use compact, self-contained handoff specs. Two sessions editing one file: one owner per file, or worktrees.

MAYA: And the rest?

BEN: Branch fights: worktrees, one branch per folder. Manual task assignment: a tracker-driven orchestrator. And port collisions and dependency changes: sandboxes, containers or VMs.

MAYA: Okay. What lasts, and what will date?

BEN: The durable principles. Parallel agents share nothing live, so share context deliberately, through plans, files and handoff specs. Plan and decompose into independent, well-sized tasks before going parallel. Isolate in layers: ownership rules, then separate branches and folders, then separate environments.

MAYA: Keep going.

BEN: Pass context as compact references to files, not raw transcripts. Keep one coordinator, one source of truth for task status, and the pull request as the review gate. And parallelism multiplies cost, so use it where waiting is the bottleneck.

MAYA: And what will date?

BEN: As of October 2026: all the specific Claude Code flags and commands we mentioned, from continue and resume to worktree, background sessions and batch. Agent view's research-preview status. Cross-session messaging. Symphony's preview status and spec details. And Docker Sandboxes' command-line tool, policies and clone mode.

MAYA: Let me try to sum up, and you correct me. Parallel sessions save me waiting, but cost more tokens. They don't share knowledge, but they do share files, branches, ports and dependencies. So I name sessions, plan independent tasks, give each file one owner, and hand off with a short spec that points to the plan.

BEN: Good so far.

MAYA: Then worktrees isolate the code, an orchestrator can hand out tasks from an issue tracker, and sandboxes isolate the whole environment. And one coordinator merges, with the pull request as the gate.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Two sessions, each in its own git worktree, both start the web app's dev server, and one fails. What happened, and how do you fix it?

PAUSE: 5

MAYA: They collided on the same port, because worktrees only isolate files. Fix it with configuration, like a port per worktree, or isolate the environment with a container or sandbox.

BEN: Exactly. Question two. Your planning session split a feature into four tasks, and two of them would write the same file. What do you do?

PAUSE: 5

MAYA: Give that file one owner. The other session may read it, but not write it.

BEN: Perfect. And last one. You want to start a fresh worker session on one task. Why not paste in the whole planning chat, and what should you give it instead?

PAUSE: 5

MAYA: Because the chat carries all its noise and fills the new context. Instead, a compact handoff spec: self-contained, naming the files and interfaces, stating what's out of scope, with an end-to-end check, and pointing to the shared plan instead of copying it.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with "collide, then isolate": two sessions on one repo without isolation, note what collides, then repeat with a worktree each.

BEN: And try planning for parallelism with file owners, writing a handoff spec by hand, fixing a port collision twice, and a label-driven flow where the pull request is the only way in. Next time, in chapter twenty-eight, we start Track seven, The bigger picture, with A lifecycle for agentic systems: why building agents needs extra lifecycle work.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-eight.
