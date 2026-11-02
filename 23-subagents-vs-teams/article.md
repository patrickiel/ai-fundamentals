# Subagents vs. Agent Teams: When Parallel Workers Should Talk to Each Other

> **Core idea in one sentence:** Subagents are focused workers that report results back to the agent that started them, while an agent team is a set of full sessions that share a task list and message each other directly, which helps when workers must react to each other's findings but costs far more tokens and adds coordination risk.

## TL;DR

- A **subagent** runs in its own context window, does one delegated task and returns a summary to its caller. The main agent manages all the work [2].
- An **agent team** in Claude Code is several independent Claude Code sessions: a **lead** that spawns and coordinates, and **teammates** that claim work from a **shared task list** and message each other through **mailboxes** [1].
- Anthropic's rule: use subagents "when you need quick, focused workers that report back", and agent teams "when teammates need to share findings, challenge each other, and coordinate on their own" [1].
- Teams cost much more. Each teammate is a separate Claude instance, and token use scales with team size [1][3].
- As of 2026-10, agent teams are **experimental and off by default**. Enable them with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` [1]. They launched as a research preview with Claude Opus 4.6 in February 2026 [4].
- Anthropic's own research system warns that agents "are not yet great at coordinating and delegating to other agents in real time" [5]. Start with read-heavy work such as review and research.

## Two ways to split work

"Building effective agents" names the two underlying patterns [6]:

- **Parallelization**: "breaking a task into independent subtasks run in parallel" (sectioning) or running the same task several times for diverse outputs (voting).
- **Orchestrator-workers**: "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results", useful when "you can't predict the subtasks needed".

Subagents are the orchestrator-workers pattern built into a coding agent. Agent teams go a step further: workers also coordinate *with each other*, not only through the orchestrator.

## Subagents

In Claude Code, "each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions" [2]. It doesn't see your conversation. Claude writes a delegation message, the subagent works, and "only the summary" comes back to the main context [2]. That makes subagents good for two jobs:

1. **Keeping the main context clean.** Test runs, log searches and documentation lookups produce verbose output the parent doesn't need (topic 05 covers this).
2. **Running separable work in parallel.** Several subagents explore different parts of a codebase at once.

You define a custom subagent as a Markdown file with YAML front matter (name, description, and optionally `tools` and `model`) in `.claude/agents/` or `~/.claude/agents/` [2]. Restricting `tools` is how you make a read-only reviewer. Setting `model: haiku` on a simple subagent is one of Anthropic's cost tips [3].

### The coordination limit

Subagents coordinate through their caller. When one subagent's result matters to another, either the parent relays it, or the subagents leave notes in files. Anthropic's research system uses the second approach on purpose: subagents "store their work in external systems, then pass lightweight references back to the coordinator", which avoids a "game of telephone" [5].

The same post names the cost of central coordination. Its lead agent runs subagents synchronously, so "the lead agent can't steer subagents, subagents can't coordinate", and the whole system can stall while one subagent finishes [5].

As of 2026-10 the line has blurred a little. Claude Code can **name** a subagent, and named agents can be resumed or messaged with a `SendMessage` tool. A subagent with that tool sees a roster of other named agents [2]. But there is still no shared task list, and the main agent still manages all the work [1].

## Agent teams

Claude Code's docs describe an agent team as multiple Claude Code instances in which "one session acts as the team lead, coordinating work, assigning tasks, and synthesizing results. Teammates work independently, each in its own context window, and communicate directly with each other" [1]. You can also message any teammate yourself without going through the lead.

| Component | Role |
|---|---|
| **Team lead** | The main session. Spawns teammates, creates tasks, coordinates, synthesises results. Fixed for the session's lifetime. |
| **Teammates** | Separate Claude Code instances. Each loads the project's CLAUDE.md, MCP servers and skills, plus the spawn prompt. They don't inherit the lead's conversation. |
| **Task list** | Shared work items: pending, in progress, completed. Tasks can depend on others. Teammates self-claim, with file locking to prevent two claiming the same task. |
| **Mailbox** | Messages between agents, delivered automatically. When a teammate finishes, it notifies the lead. |

As of 2026-10, this state lives in your **home directory**, not your project: team config at `~/.claude/teams/{team-name}/config.json`, mailboxes under `~/.claude/teams/{team-name}/inboxes/`, and tasks under `~/.claude/tasks/{team-name}/`. The docs state that a file like `.claude/teams/teams.json` in your project is *not* recognised as configuration [1].

### Subagents and teams side by side

From the Claude Code docs [1], lightly condensed:

| | Subagents | Agent teams |
|---|---|---|
| **Context** | Own window; results return to the caller | Own window; fully independent |
| **Communication** | Return a result; named subagents can also message | Teammates message each other directly |
| **Coordination** | Main agent manages all work | Self-coordination plus a shared task list |
| **Best for** | Focused tasks where only the result matters | Work that needs discussion and collaboration |
| **Token cost** | Lower: results summarised back | Higher: each teammate is a separate instance |

## Turning teams on (as of 2026-10)

- Set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` to `1` in your shell or in `settings.json` under `"env"` [1].
- Teams need an **interactive** session. In non-interactive mode (`-p`) and the Agent SDK, Claude doesn't spawn teammates [1].
- Ask for a team in plain language, naming the roles. Claude may use subagents instead, so ask explicitly for an agent team if it does [1].
- With teams enabled, a subagent that Claude *names* launches as a teammate, so teams can form when you didn't ask for one. Set the variable to `0` to switch back [1].
- The default **in-process** display works in any terminal. Split panes need tmux or iTerm2, and the docs list Windows Terminal, VS Code's terminal and Ghostty as unsupported for split panes [1].
- **Plan approval**: if the lead is in plan mode, a teammate it spawns plans read-only before implementing. **Hooks** such as `TeammateIdle` and `TaskCompleted` can enforce quality gates [1].
- **Shutdown**: ask the lead to shut a teammate down. The teammate can approve or reject with an explanation, and it finishes its current tool call first [1].

## Where teams pay off

Anthropic's launch note says teams are "best for tasks that split into independent, read-heavy work like codebase reviews" [4]. The docs list four strong cases [1]:

1. **Research and review.** "A single reviewer tends to gravitate toward one type of issue at a time." Give teammates distinct lenses, such as security, performance and test coverage, and have the lead synthesise.
2. **Debugging with competing hypotheses.** A single agent "tends to find one plausible explanation and stop looking". Teammates who try to *disprove* each other's theories counter that anchoring, and "the theory that survives is much more likely to be the actual root cause".
3. **New modules or features**, where each teammate owns a separate piece.
4. **Cross-layer changes** across frontend, backend and tests, each owned by a different teammate.

A common pattern that uses live messaging is **find-and-fix**: one teammate reviews and sends findings one at a time while another fixes them, so review and repair overlap instead of running one after the other. It only works if the fixer's edits don't collide with other work.

Task dependencies matter too. A teammate that sets up packages can block page builders until it finishes, and Claude Code unblocks dependent tasks automatically when the blocking task completes [1].

## Where teams don't pay off

The docs are direct: "For sequential tasks, same-file edits, or work with many dependencies, a single session or subagents are more effective" [1]. Anthropic's research team reached the same conclusion for multi-agent systems in general. Domains that require "all agents to share the same context" or that involve "many dependencies between agents" are poor fits, and "most coding tasks involve fewer truly parallelizable tasks than research" [5].

### The cost

- Anthropic's multi-agent research system used about **15× the tokens of a chat**, and token usage alone explained 80% of performance variance on its browsing benchmark. Multi-agent designs pay off only for "tasks where the value of the task is high enough" [5].
- Claude Code's cost guide says teams use "approximately 7x more tokens than standard sessions when teammates run in plan mode" [3].
- Cost tips from the same guide: use Sonnet for teammates, keep teams small, keep spawn prompts focused, and shut teammates down when they're done [3].

## Best practices

From the Claude Code docs [1], with the multi-agent research lessons [5]:

1. **Give enough context in the spawn prompt.** Teammates don't see the lead's history. Each needs "an objective, an output format, guidance on the tools and sources to use, and clear task boundaries" [5].
2. **Choose a small team.** Start with three to five teammates. "Three focused teammates often outperform five scattered ones." [1]
3. **Size tasks sensibly.** Too small and coordination costs more than it saves. Too large and teammates work too long without check-ins. About five to six tasks per teammate keeps everyone busy [1].
4. **Avoid file conflicts.** "Two teammates editing the same file leads to overwrites." Split work by file ownership [1].
5. **Tell the lead to wait.** The lead sometimes starts doing tasks itself. Say: "Wait for your teammates to complete their tasks before proceeding." [1]
6. **Monitor and steer.** Redirect teammates that drift. A team left alone too long wastes effort [1].
7. **Start with research and review**, which have clear boundaries and no write conflicts, before trying parallel implementation [1].

Known limitations as of 2026-10 include: in-process teammates aren't restored on `/resume`, task status can lag, shutdown can be slow, one team per session, no nested teams, and the lead can't be changed [1].

## A decision rule

- **Single agent** when the task is small or sequential, or most steps touch the same files.
- **Subagents** when the subtasks are separable: each can be done and checked without the others, and the parent only needs the results.
- **Agent team** when workers need **each other's findings while they work**: competing hypotheses, multi-lens review, or modules with clear owners that must stay consistent.

Each extra worker is a full context window you pay for, so parallelism trades tokens for time and breadth.

## What lasts vs. what will date

- **Durable principles:**
  - Workers that can only report upward make the coordinator a bottleneck. Direct messaging removes it but adds cost and collision risk.
  - Parallel work helps only when tasks are independent, clearly scoped and sensibly sized.
  - Independent investigators who try to disprove each other resist anchoring better than one agent.
  - Token use scales with the number of workers. Use multiple agents where the task's value justifies it.
- **Time-sensitive details:**
  - Agent teams as an experimental Claude Code feature, the enabling variable, display modes and limitations (as of 2026-10).
  - Named subagents and `SendMessage` (as of 2026-10).
  - The ~7× and ~15× token figures and the 90.2% research-eval result.

## Try it yourself

1. **Sort your tasks.** List five recent multi-step tasks. Label each single agent, subagents or team, with one sentence of reasoning.
2. **Write a delegation brief.** For one subagent task, write the objective, output format, tools and boundaries. Compare the result with a one-line delegation.
3. **Run a three-lens review.** With teams enabled, ask for three teammates to review one PR for security, performance and tests. Without teams, do the same with three subagents. Compare findings and tokens with `/usage`.
4. **Competing hypotheses.** Give a bug with an unclear cause to three teammates told to disprove each other. Note whether they converge.
5. **Draw file ownership.** For a planned feature, assign each worker the files it owns and check that no two share a file.

## Key terms

- **Subagent:** a worker with its own context window that does a delegated task and returns a summary to its caller.
- **Orchestrator-workers:** a pattern where a central model splits work, delegates it and synthesises results.
- **Agent team:** several independent Claude Code sessions under a lead, sharing a task list and messaging each other.
- **Team lead:** the session that spawns teammates, assigns tasks, synthesises results and shuts teammates down.
- **Shared task list:** work items teammates claim, with states and dependencies.
- **Mailbox:** the per-agent message store teammates use to reach each other.
- **Blocked task:** a task that can't be claimed until the tasks it depends on are complete.
- **Anchoring:** the tendency to stick with the first plausible explanation, which competing investigators help counter.

## Sources

1. [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams) (Claude Code docs, Anthropic, accessed 2026-10-14). Architecture, the subagents comparison table, enabling, display modes, task claiming and dependencies, use cases, best practices and limitations.
2. [Create custom subagents](https://code.claude.com/docs/en/sub-agents) (Claude Code docs, Anthropic, accessed 2026-10-14). Isolated context windows, summary-only results, definitions and tool restrictions, named subagents and `SendMessage`.
3. [Manage costs effectively](https://code.claude.com/docs/en/costs) (Claude Code docs, Anthropic, accessed 2026-10-14). Agent team token costs (~7× in plan mode) and cost-control tips for teammates and subagents.
4. [Introducing Claude Opus 4.6](https://www.anthropic.com/news/claude-opus-4-6) (Anthropic, 2026-02-05). Launch of agent teams as a research preview, best for independent, read-heavy work.
5. [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (Hadfield et al., Anthropic, 2025-06-13). Orchestrator-worker design, token economics, delegation briefs, filesystem hand-offs, synchronous bottlenecks, and when multi-agent is a poor fit.
6. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Erik Schluntz and Barry Zhang, Anthropic, 2024-12-19). The parallelization and orchestrator-workers patterns.
