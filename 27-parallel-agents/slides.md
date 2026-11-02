---
theme: ../theme
chapter: '27'
header: 'AI Fundamentals · 27 · Parallel agents'
layout: cover
---

# Running Agents in Parallel

Sessions **share nothing they know** but **share files, branches, ports and dependencies**, so plan, hand off on purpose, and isolate in layers.

---

## Why parallel, and the catch

- You review one session's output while **others keep working**
- No more waiting on a single agent
- Cost: running several sessions **multiplies token usage**
- Parallel sessions **fail quietly**: each looks busy, each output reads fine

<!-- The token warning is from Claude Code's "Run agents in parallel" page. Use parallelism where waiting is the real bottleneck. -->

---

## What goes wrong

**Any work**
- Losing track of which session is which
- **No shared memory**: each session has its own context
- Two sessions editing one file → **overwrites**

**Software**
- Branch fights in one checkout
- **Port collisions** (`localhost:3000`)
- One session's dependency or database change breaks the rest

<!-- Instruction files like CLAUDE.md load at session start; discoveries in one session don't reach another unless written down or sent. -->

---

## Fix 1: name and resume sessions

| Need | Mechanism |
|---|---|
| Most recent session here | `claude --continue` |
| Pick from a list, or by name | `claude --resume [name]` |
| Name it | `claude -n <name>`, `/rename` |
| Copy it out | `/export` |
| Try another approach | `/branch`, `--fork-session` |

- Treat sessions "like branches": one workstream, one context

<!-- Commands as of 2026-10, from Claude Code's session docs and best-practices guide. -->

---

## Fix 2: plan first, then split

- **Plan mode**: explore and propose without editing
- Ask for tasks that can run **independently**
- Check: does any file appear in **two** tasks?
- Give it **one owner**; others read, never write
- Good task size: "self-contained units that produce a **clear deliverable**"

<!-- Too small and coordination costs more than the work; too large and an agent runs too long without a check-in (agent-teams guidance). -->

---

## Fix 3: hand off with a spec, not a chat dump

- A pasted chat carries **all its noise** into the new context
- Write a spec, then run it in a **fresh session** with clean context
- Good spec: **self-contained**, names files and interfaces, states what's **out of scope**
- Ends with an **end-to-end verification** step
- One handoff per task, each **pointing to** the shared plan

<!-- From Claude Code's best-practices guide ("Let Claude interview you" section). -->

---

## Fix 4: launch and watch from one place

- **Agent view** (research preview): `claude agents`
- "What's running, what needs your input, and what's done"
- Dispatch from the prompt, or `claude --bg "<task>"`
- Send the current session to the background: `/bg`
- Background sessions use your plan the **same as interactive** ones

<!-- As of 2026-10. Sessions dispatched from agent view move into their own worktree before editing files. -->

---

## Git worktrees: isolate the code

```bash
git worktree add ../myapp-auth    -b feature/auth
git worktree add ../myapp-billing -b feature/billing
git worktree list
git worktree remove ../myapp-auth
```

- One repository, **many working trees**, each with its own branch, `HEAD` and index
- Git refuses to check out a branch **already checked out** elsewhere

<!-- From the git-worktree documentation. No repeated clones, no branch switching in a shared folder. -->

---

## Worktrees in Claude Code

- `claude --worktree <name>` → `.claude/worktrees/<name>/` on a new branch
- Subagents: `isolation: worktree`; `/batch` splits a change across 5–30 isolated subagents
- Fresh checkout: **install dependencies** there
- `.worktreeinclude` copies gitignored files such as `.env`
- Worktrees isolate **files only**: ports and global installs are still shared

<!-- Features as of 2026-10, from Claude Code's worktrees and parallel-agents docs. -->

---

## Orchestration: automate the hand-out

- **Symphony** (OpenAI, open source, engineering preview)
- Watches an issue tracker; **one agent per issue**, each in its **own workspace**
- Concurrency limits, retries with backoff, **proof of work**, PR as the gate
- Spec caveat: no built-in sandbox; each implementation **defines its own trust boundary**
- Portable pattern: backlog → queue, label → trigger, PR → review

<!-- From the openai/symphony repository and SPEC.md (read 2026-10). Linear is the tracker in the reference implementation. -->

---

## Sandboxes: isolate the environment

- Worktrees leave **ports, packages and services** shared
- Sandbox: a disposable environment per agent
- Docker Sandboxes, for example: microVM with its own **daemon, filesystem and network**
- Default mounts your project read-write; **clone mode** gives each agent a private copy
- Containers, VMs or cloud sessions give the same property

<!-- Docker Sandboxes details as of 2026-10. The vendor matters less than the property: each task gets its own dependencies, processes and ports. -->

---

## One planner, many isolated workers

- One session **plans** the feature
- Starts one isolated worker per subtask (worktree, sandbox, cloud)
- **Monitors**, reviews and merges each result
- Cleans up once each task is confirmed done
- Same shape as orchestrator-workers (topic 25). Review the merge with a **fresh context**

<!-- Claude Code's /batch and dynamic workflows are packaged versions of this pattern. -->

---

## The layered picture

| Problem | Layer |
|---|---|
| Losing track | Names, resume, agent view |
| Stale or noisy context | Plan first, compact handoff specs |
| Same-file edits | File ownership, worktrees |
| Branch fights | Worktrees |
| Manual assignment | Tracker-driven orchestrator |
| Ports, dependencies | Sandboxes, containers, VMs |

<!-- Add a layer only when you hit the problem it solves. -->

---

## What lasts vs. what will date

**Lasts**
- Share context deliberately: plans, files, specs
- Decompose into **independent, well-sized** tasks first
- Isolate in layers: ownership → branches → environments
- One coordinator, PR as the review gate

**Will date**
- Claude Code commands and flags, agent view, `/batch`
- Symphony's spec and status, Docker's `sbx` CLI and clone mode

<!-- All tool details as of 2026-10. -->

---

## Try it yourself

1. Run two sessions on one repo **without** isolation, then with worktrees
2. Plan a feature into 3–4 independent tasks; assign owners to shared files
3. Write a handoff spec by hand; compare with pasting the full chat
4. Fix a port collision with config, then with a container
5. One labelled issue → its own branch → PR as the only way in

<!-- Exercise 1 makes the case for isolation faster than any explanation. -->

---
layout: end
---

# Key takeaways

- Parallel sessions **share nothing they know**: hand off context on purpose
- **Plan independent tasks** before you go parallel
- Isolate in layers: **ownership → worktrees → sandboxes**

code.claude.com/docs/en/worktrees · git-scm.com/docs/git-worktree · github.com/openai/symphony
