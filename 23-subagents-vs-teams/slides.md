---
theme: ../theme
chapter: '23'
header: 'AI Fundamentals · 23 · Subagents vs. agent teams'
layout: cover
---

# Subagents vs. Agent Teams

Subagents **report back**; teammates **talk to each other**. Use a team only when workers need each other's findings while they work.

---

## Two ways to split work

- **Parallelization**: independent subtasks side by side, or the same task several times (voting)
- **Orchestrator-workers**: a central model breaks work down, delegates, synthesises
- **Subagents** = orchestrator-workers inside a coding agent
- **Agent teams** add coordination **between workers**

<!-- Both patterns come from Anthropic's "Building effective agents" (Dec 2024). -->

---

## Subagents

- Own **context window**, own tools and permissions
- Get a delegation message, not your conversation
- Return **only a summary** to the caller
- Good for: keeping main context clean, separable parallel work
- Defined as Markdown + YAML in `.claude/agents/`; restrict `tools`, set `model`

<!-- Restricting tools is how you build a read-only reviewer. model: haiku on simple subagents is one of Anthropic's own cost tips. Topic 05 covers context isolation. -->

---

## The coordination limit

- Subagents coordinate **through the caller**, or via files
- Anthropic's research system: "the lead agent can't steer subagents, subagents can't coordinate"
- Their fix: write outputs to the filesystem, pass **lightweight references**
- As of 2026-10, **named** subagents can message via `SendMessage`
- Still no shared task list: the main agent manages all work

<!-- The filesystem hand-off avoids the "game of telephone" where results get re-summarised at every hop. -->

---

## Agent teams

| Component | Role |
|---|---|
| **Lead** | Spawns teammates, creates tasks, synthesises; fixed for the session |
| **Teammates** | Separate Claude Code instances; load CLAUDE.md, MCP, skills + spawn prompt |
| **Task list** | Pending / in progress / done; dependencies; self-claim with file locks |
| **Mailbox** | Direct messages between agents, delivered automatically |

State lives in `~/.claude/teams/` and `~/.claude/tasks/`, not your project

<!-- Teammates don't inherit the lead's conversation, so the spawn prompt must carry task context. A project-level .claude/teams file is not recognised as configuration. -->

---

## Side by side

| | Subagents | Agent teams |
|---|---|---|
| Context | Own window; results to caller | Own window; fully independent |
| Communication | Return a result (named ones can message) | Message each other directly |
| Coordination | Main agent manages all | Self-coordination + task list |
| Best for | Only the result matters | Discussion and collaboration |
| Tokens | Lower | **Higher**: each is a full instance |

<!-- Condensed from the Claude Code agent-teams docs comparison table. -->

---

## Turning teams on (as of 2026-10)

- **Experimental**, off by default; research preview since Opus 4.6 (Feb 2026)
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in shell or `settings.json` `env`
- **Interactive sessions only**: not with `-p` or the Agent SDK
- Ask for a team explicitly and name the roles
- Split panes need tmux or iTerm2; in-process works in any terminal

<!-- With teams on, a subagent Claude names launches as a teammate, so teams can form unasked. Windows Terminal and VS Code's terminal don't support split panes; use in-process mode there. -->

---

## Where teams pay off

- **Multi-lens review**: "a single reviewer tends to gravitate toward one type of issue"
- **Competing hypotheses**: teammates try to **disprove** each other, countering anchoring
- **New modules** with clear owners
- **Cross-layer** changes: frontend, backend, tests
- Launch note: best for "independent, **read-heavy** work like codebase reviews"

<!-- Find-and-fix (reviewer messages findings, fixer acts immediately) is a common pattern that uses live messaging, as long as edits don't collide. Task dependencies let setup work block builders until it's done. -->

---

## Where they don't

- Sequential tasks, **same-file edits**, many dependencies → single session or subagents
- Poor fit when all agents need **the same context**
- "Most coding tasks involve fewer truly parallelizable tasks than research"
- Agents "are not yet great at coordinating and delegating … in real time"

<!-- Lessons from Anthropic's multi-agent research system (June 2025), echoed in the Claude Code docs. -->

---

## The cost

- Multi-agent research: about **15× the tokens** of a chat
- Claude Code teams: about **7×** a standard session when teammates run in plan mode
- Token use scales with **team size × run time**
- Tips: Sonnet for teammates, small teams, focused spawn prompts, **shut down** when done

<!-- Anthropic: multi-agent systems need tasks whose value is high enough to pay for the extra tokens. -->

---

## Best practices

1. **Brief each worker**: objective, output format, tools, boundaries
2. **Start with 3–5 teammates**; about 5–6 tasks each
3. **Own files**: no two teammates edit the same file
4. Tell the lead: "**Wait for your teammates** to complete their tasks"
5. **Monitor and steer**; start with research and review

<!-- "Three focused teammates often outperform five scattered ones." Known limits as of 2026-10: no /resume for in-process teammates, task status lag, slow shutdown, one team per session, no nested teams. -->

---

## A decision rule

- **Single agent**: small, sequential, same files
- **Subagents**: separable tasks; the parent only needs results
- **Agent team**: workers need **each other's findings while working**

Every extra worker is a full context window you pay for.

<!-- Parallelism trades tokens for time and breadth. Make sure the task's value justifies it. -->

---

## What lasts vs. what will date

**Lasts**
- Report-only workers make the coordinator a bottleneck
- Parallelism needs independent, scoped, sized tasks
- Investigators who challenge each other resist anchoring

**Will date**
- The experimental flag, display modes, limitations (as of 2026-10)
- `SendMessage` for named subagents; the 7× and 15× figures

---

## Try it yourself

1. Label five recent tasks: single / subagents / team
2. Write a full delegation brief for one subagent
3. Three-lens PR review: team vs. subagents; compare `/usage`
4. Give one unclear bug to three teammates told to disprove each other
5. Draw file ownership for a planned feature

<!-- Exercise 3 makes the token cost concrete. -->

---
layout: end
---

# Key takeaways

- **Subagents report back; teammates coordinate** through a task list and mailboxes
- **Teams fit read-heavy, multi-perspective work**, not same-file or sequential edits
- **Tokens scale with workers**: brief well, keep teams small, shut them down

code.claude.com/docs/en/agent-teams\
code.claude.com/docs/en/sub-agents\
anthropic.com/engineering/multi-agent-research-system
