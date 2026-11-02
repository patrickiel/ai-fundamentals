# Running Agents in Parallel: Isolation, Handoffs and Orchestration

> **Core idea in one sentence:** Running several agent sessions at once keeps your own time fully used, but sessions don't share what they know while they do share files, branches, ports and dependencies, so parallel work holds up only if you plan independent tasks, hand off context on purpose, and isolate each session in layers.

## TL;DR

- Parallel sessions mean you are not waiting on one agent. You review one session's output while others work. The cost is tokens: "Running several sessions or subagents at once multiplies token usage" [1].
- Each session has **its own context**. Nothing one session learns reaches another unless it is written to a file or sent as a message [1][6].
- General fixes: **name and resume** sessions [4], **plan first and split into independent tasks** [6], give each file **one owner** [1], and start new sessions from a **compact, self-contained spec** rather than a pasted chat [6].
- Code conflicts need **git worktrees**: one folder and branch per session, sharing one repository [2][3].
- Environment conflicts (ports, installed dependencies, running services) need **sandboxes**: an isolated filesystem, network and runtime per agent [8].
- Task hand-out can be automated. OpenAI's open-source **Symphony** turns an issue tracker into a queue that launches one agent per issue in its own workspace [7].

## Why run sessions in parallel at all

A single agent on a long task leaves you waiting. With several sessions you can read what one just returned, send it the next instruction, and move on while the others keep working. Claude Code's documentation now lists several built-in ways to do this, from subagents inside one conversation to background sessions you dispatch and check on later [1].

The catch is that parallel sessions fail quietly. Every session looks busy and every output reads fine, but underneath they can overwrite each other's work, and you often don't notice until something breaks.

## The problems

**General (any kind of work):**

- **Losing track.** Which session was doing what, and where did the important one go?
- **No shared memory.** Each session is its own conversation with its own context window. Instruction files such as `CLAUDE.md` are loaded when a session starts [6], and a session doesn't see another session's discoveries unless they are written down or passed on. As of 2026-10, Claude Code has a cross-session messaging feature for passing findings between sessions you run yourself [1].
- **Overwrites.** Two agents editing the same file clobber each other. Claude Code's agent-teams guidance is blunt: two teammates editing the same file "leads to overwrites", so break the work up so each owns a different set of files [1].

**Software-specific:**

- **Branch fights.** One session switches branches in the same checkout another is using.
- **Port collisions.** Several dev servers try to bind the same port, such as `localhost:3000`.
- **Shared dependencies and state.** One session upgrades a package, changes the build or migrates the local database, and every other session breaks without knowing why.

## General fixes

### 1. Name, resume and catch up on sessions

Treat sessions as long-lived, named work units you return to, not throwaway chats. Claude Code's guide says to name sessions and "treat them like branches: each workstream gets its own persistent context" [6]. The mechanics (as of 2026-10) [4]:

| Need | Mechanism |
|---|---|
| Reopen the most recent session here | `claude --continue` |
| Pick from a list, or by name | `claude --resume`, `claude --resume <name>` |
| Name a session | `claude -n <name>` at start, or `/rename` |
| Copy a conversation out | `/export` to clipboard or a text file |
| Try another approach without losing this one | `/branch`, or `--fork-session` |

### 2. Plan first, then split into independent tasks

Use a planning pass before going parallel. Claude Code's recommended flow is explore → plan → implement → commit, with **plan mode** for the first two steps, so Claude reads and proposes without editing [6]. For parallel work, ask the plan for **tasks that can be done independently** and check the result: is there any file two tasks would both write? If so, give that file one owner and tell the other sessions they may read it but not write it [1].

The size of each task matters too. The agent-teams guidance describes good tasks as "self-contained units that produce a clear deliverable, such as a function, a test file, or a review". Too small and coordination costs more than the work. Too large and an agent runs too long without a check-in [1].

### 3. Hand off with a compact document, not a chat dump

Pasting a whole conversation into a new session carries all its noise and fills the new context. The better pattern is a **handoff document**. Claude Code's guide recommends having Claude write a spec, then starting "a fresh session to execute it. The new session has clean context focused entirely on implementation" [6]. It describes what makes a spec useful: it is self-contained, names the files and interfaces involved, states what is out of scope, and ends with an end-to-end verification step [6].

For parallel work, a planning session can write **one handoff per task**, each pointing to the shared plan file rather than copying it. Every worker starts clean, with the same shared ground truth.

### 4. Launch and monitor background sessions

As of 2026-10, Claude Code's **agent view** (research preview) is one screen for background sessions: "what's running, what needs your input, and what's done" [5]. You open it with `claude agents`, type a prompt to dispatch a new background session, start one from the shell with `claude --bg "<task>"`, or move the current session to the background with `/bg`. Background sessions "consume your subscription usage the same as interactive sessions" [5].

## Development-specific fixes

### Git worktrees: isolate the code

A **git worktree** lets one repository "support multiple working trees, allowing you to check out more than one branch at a time" [3]. Each linked worktree has its own folder, `HEAD` and index, and shares everything else, including history and refs, with the main repository. You don't clone the project repeatedly, and git refuses by default to check out a branch already checked out in another worktree [3].

```bash
# one folder and branch per parallel task
git worktree add ../myapp-auth    -b feature/auth
git worktree add ../myapp-billing -b feature/billing
git worktree list
git worktree remove ../myapp-auth   # when the work is merged
```

Give each session its own worktree, and they can't overwrite each other's files or fight over the checked-out branch. Claude Code builds this in (as of 2026-10) [2]:

- `claude --worktree <name>` (or `-w`) creates a worktree under `.claude/worktrees/<name>/` on a new branch and starts the session there.
- Subagents can each get a temporary worktree (`isolation: worktree` in their definition), and sessions dispatched from agent view move into their own worktree before editing files [1][2][5].
- `/batch` splits one large change across 5 to 30 worktree-isolated subagents [1].

Two practical notes from the docs. A worktree is a fresh checkout, so you have to install dependencies there, and gitignored files such as `.env` are missing unless you list them in a `.worktreeinclude` file [2]. And worktrees isolate **files only**. A dev server in one worktree can still collide on a port with a server in another, and anything installed globally is shared.

### Issue-driven orchestration: automate the hand-out

Worktrees isolate the code, but you are still the one assigning tasks. An **orchestrator** automates that step. OpenAI's **Symphony** is one open-source example, described in its repository as "a low-key engineering preview for testing in trusted environments" (as of 2026-10) [7]:

- It **watches an issue tracker** (Linear in the reference implementation) and starts a coding agent for each eligible issue.
- Its specification describes a long-running daemon with an orchestrator, a tracker poller, a **workspace per issue**, concurrency limits (default 10 agents) and retries with exponential backoff [7].
- Agents report "proof of work" such as CI status and PR review feedback, and land the PR once the work is accepted [7].
- The spec is explicit about what it doesn't do: it does not mandate "strong sandbox controls beyond what the coding agent and host OS provide". Each implementation "defines its own trust boundary" [7].

The pattern travels beyond any one tool. Turn the backlog into a queue, use a tracker state or label as the trigger, give each task its own branch and workspace, and use the pull request as the review gate.

### Sandboxes: isolate the whole environment

Worktrees and orchestrators leave the environment shared: installed packages, running services, ports. Fixing that takes a **sandbox**, a disposable, isolated environment in which the agent can install, build and run without touching your machine or the other sessions.

Docker Sandboxes is one implementation (as of 2026-10). It runs coding agents in microVMs, and "each sandbox gets its own Docker daemon, filesystem, and network" [8]. A sandbox is started from a project directory with the `sbx` CLI, and network access follows a policy you choose (open, balanced, or locked down). By default the project directory is mounted read-write, so the agent's edits appear in your working tree as they happen. For running several agents on one repository, **clone mode** gives each agent a private clone inside its sandbox instead [8].

Containers, VMs and cloud sessions provide the same kind of isolation. The vendor matters less than the property: **each parallel task gets its own dependencies, processes and ports.**

### Putting it together: one planner, many isolated workers

A common end state is to let one session orchestrate. It plans the feature, starts one isolated worker per subtask (a worktree, a sandbox or a cloud session), monitors them, reviews and merges each result, and cleans up the workspace once a task is confirmed done. This is the orchestrator-workers shape from topic 25. Claude Code's own `/batch` and dynamic workflows are packaged versions of it, for example migrating many files while "working on each file in its own isolated copy" and verifying each result [1]. As with any orchestration, review the merged output yourself, ideally with a fresh-context reviewer [6].

## The layered picture

| Problem | Layer that fixes it |
|---|---|
| Losing track of sessions | Names, resume, a dashboard such as agent view |
| Stale or noisy context | Plan first; compact, self-contained handoff specs |
| Two sessions editing one file | One owner per file, or worktrees |
| Branch fights | Worktrees (one branch per folder) |
| Manual task assignment | Tracker-driven orchestrator |
| Port collisions, dependency changes | Sandboxes, containers, VMs |

## What lasts vs. what will date

- **Durable principles:**
  - Parallel agents share nothing live, so share context deliberately through plans, files and handoff specs.
  - Plan and decompose into independent, well-sized tasks *before* going parallel.
  - Isolate in layers: ownership rules, then separate branches and folders, then separate environments.
  - Pass context as compact references to files, not raw transcripts.
  - Keep one coordinator, one source of truth for task status, and the pull request as the review gate.
  - Parallelism multiplies cost. Use it where waiting is the bottleneck.
- **Time-sensitive details (as of 2026-10):** Claude Code flags and commands (`--continue`, `--resume`, `-n`, `/export`, `--worktree`, `.worktreeinclude`, `claude agents`, `--bg`, `/bg`, `/batch`), agent view's research-preview status, cross-session messaging, Symphony's preview status and spec details, and Docker Sandboxes' `sbx` CLI, policies and clone mode.

## Try it yourself

1. **Collide, then isolate.** Run two agent sessions on the same repo, each on a small task, *without* isolation, and note what collides. Repeat with each in its own `git worktree`.
2. **Plan for parallelism.** Use plan mode to break a feature into 3–4 independent tasks. Find any file two tasks would both write and assign it an owner.
3. **Write a handoff spec by hand.** Goal, files and interfaces, out of scope, done-criteria with an end-to-end check, links to the plan. Start a fresh session from it and compare with pasting the full chat.
4. **Fix a port collision twice.** Run two copies of a web app in parallel worktrees. Resolve the clash first with configuration (a per-worktree port), then with a container or sandbox.
5. **Label-driven flow.** Label one issue for an agent, give it its own branch, and make the pull request the only way its work gets merged.

## Key terms

- **Parallel sessions:** several independent agent conversations working at the same time, each with its own context.
- **File ownership:** a rule that only one session may write a given file; others may read it.
- **Handoff document:** a compact, self-contained spec that references files instead of copying them, used to start a fresh session.
- **Git worktree:** a linked working tree that checks out another branch of the same repository in its own folder.
- **Orchestrator:** a process or agent that assigns tasks, tracks their status and merges results.
- **Daemon:** a long-running background process, for example one polling an issue tracker.
- **Sandbox:** a disposable, isolated environment (container or microVM) where an agent can install, build and run without affecting the host or other sessions.

## Sources

1. [Run agents in parallel](https://code.claude.com/docs/en/agents) (Claude Code docs, read 2026-10), with the linked [agent teams](https://code.claude.com/docs/en/agent-teams) and [dynamic workflows](https://code.claude.com/docs/en/workflows) pages. Approaches to parallel work, the token-cost warning, file partitioning, task sizing, cross-session messaging, and `/batch`.
2. [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees) (Claude Code docs, read 2026-10). `--worktree`, subagent isolation, environment setup in a fresh checkout, `.worktreeinclude`, and cleanup.
3. [git-worktree documentation](https://git-scm.com/docs/git-worktree) (Git project, read 2026-10). Main vs. linked worktrees, shared vs. per-worktree state, the add/list/remove/prune/lock commands, and the checked-out-branch safeguard.
4. [Manage sessions](https://code.claude.com/docs/en/sessions) (Claude Code docs, read 2026-10). `--continue`, `--resume`, naming, `/export`, and branching sessions.
5. [Manage agents with agent view](https://code.claude.com/docs/en/agent-view) (Claude Code docs, read 2026-10). `claude agents`, `--bg`, `/bg`, session states, worktree isolation for background sessions, and usage.
6. [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (Claude Code docs, read 2026-10). Explore-plan-implement with plan mode, `CLAUDE.md` loading, writing a spec and executing it in a fresh session, named sessions as branches, and fresh-context review.
7. [openai/symphony](https://github.com/openai/symphony) and its [SPEC.md](https://github.com/openai/symphony/blob/main/SPEC.md) (OpenAI, GitHub, read 2026-10). Tracker-driven orchestration, per-issue workspaces, proof of work, concurrency and retries, preview status, and the explicit trust-boundary caveat.
8. [Get started with Docker Sandboxes](https://docs.docker.com/ai/sandboxes/get-started/) (Docker docs, read 2026-10). MicroVM isolation per agent, the `sbx` CLI, network policies, direct-mount vs. clone mode.
