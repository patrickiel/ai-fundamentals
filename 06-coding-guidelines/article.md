# Writing Agent Instruction Files That Change Behaviour

> **Core idea in one sentence:** A coding agent's instruction file (`CLAUDE.md`, `AGENTS.md` and similar) should hold only what changes the agent's default behaviour (how to handle ambiguity, how much to change, how to prove the work is done, which tools to use), kept short and scoped, while anything that must never happen is enforced by permissions and hooks rather than by prose.

## TL;DR

- An instruction file is loaded into every session, so every line costs context. Anthropic's test for each line: *"Would removing this cause Claude to make mistakes?"* If not, cut it [3].
- Andrej Karpathy described the typical failure modes of coding agents in January 2026: wrong assumptions carried forward without checking, hidden confusion, overcomplicated code and bloated abstractions, and edits to code unrelated to the task [1].
- A widely copied community `CLAUDE.md` turns those observations into four rules: **think before coding**, **simplicity first**, **surgical changes** and **goal-driven execution** [2].
- Add project-specific facts the agent can't guess: non-default tools and commands, environment quirks, repository etiquette [3][5].
- Instruction files are **advisory**. Claude Code's docs say they are "context, not enforced configuration". Hard safety rules belong in permission deny rules and hooks [4][7].
- Keep files under about **200 lines** (Anthropic's target), move area-specific rules into path-scoped files, and let nested files serve each package in a monorepo [4][5].

## Why the instruction file matters, and its limits

Most coding agents read a project-level instruction file at the start of each session. Claude Code reads `CLAUDE.md`. Codex, Cursor, Gemini CLI, GitHub Copilot's coding agent and many others read `AGENTS.md`, an open format now stewarded by the Agentic AI Foundation under the Linux Foundation and described as "a README for agents" [5]. As of 2026-10, Claude Code also reads `AGENTS.md` when a repository has no `CLAUDE.md` [4].

Because the file loads every time, it shapes every task. Anthropic's best-practices guide frames the main constraint as the context window: performance degrades as it fills, so a bloated file hurts twice. It takes space, and it buries the rules that matter. In the guide's words, "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" [3]

The file also has a ceiling. Claude Code delivers `CLAUDE.md` content as a user message after the system prompt, and "there's no guarantee of strict compliance, especially for vague or conflicting instructions" [4]. Karpathy noted that the failure modes he listed persisted "despite a few simple attempts to fix it via instructions in CLAUDE.md" [1]. A good file reduces how often you correct the agent. It doesn't replace reviewing the diff.

## What the agent gets wrong by default

Karpathy's post, written after weeks of mostly agent-driven coding, describes the mistakes as "subtle conceptual errors that a slightly sloppy, hasty junior dev might do" rather than syntax errors [1]:

- "The models make wrong assumptions on your behalf and just run along with them without checking."
- "They don't manage their confusion, they don't seek clarifications, they don't surface inconsistencies, they don't present tradeoffs, they don't push back when they should."
- "They really like to overcomplicate code and APIs, they bloat abstractions, they don't clean up dead code after themselves." He describes a 1,000-line construction cut to 100 lines once he asked whether a simpler approach would work.
- They "sometimes change/remove comments and code they don't like or don't sufficiently understand as side effects, even if it is orthogonal to the task at hand."

He also names the main lever: "LLMs are exceptionally good at looping until they meet specific goals... Don't tell it what to do, give it success criteria and watch it go." [1]

## Four behaviour rules

The community repository `andrej-karpathy-skills` packages these observations as a single `CLAUDE.md` (also published as a Claude Code plugin). It is derived from Karpathy's post, not written by him [2]. Each rule targets one of the habits above.

### 1. Think before coding

**Rule:** "Don't assume. Don't hide confusion. Surface tradeoffs." State assumptions, and when a request has several readings, present them instead of picking one silently. If a simpler approach exists, say so and push back where warranted. If something is unclear, stop and ask [2].

**Why:** misunderstandings are cheapest to fix before code exists. Claude Code's own guidance points the same way: for larger features, have the agent interview you and write a spec before implementing (topic 08) [3].

### 2. Simplicity first

**Rule:** "Minimum code that solves the problem. Nothing speculative." No features beyond the request, no abstractions for single-use code, no unrequested configurability, no error handling for impossible scenarios. The file adds a heuristic: "If you write 200 lines and it could be 50, rewrite it," and a check: "Would a senior engineer say this is overcomplicated?" [2]

**Why:** every extra line has to be read, reviewed and worked around by every later change, by you and by the agent.

### 3. Surgical changes

**Rule:** "Touch only what you must. Clean up only your own mess." Don't "improve" adjacent code, comments or formatting, and match the existing style. If you notice unrelated dead code, mention it rather than deleting it. Remove only the imports, variables or functions that *your* change made unused. The test: "Every changed line should trace directly to the user's request." [2]

**Why:** small diffs are reviewable, and code that looks wrong may exist for reasons nobody wrote down. You still hear about the issues and decide later.

### 4. Goal-driven execution

**Rule:** "Define success criteria. Loop until verified." Turn imperative tasks into checkable goals: "Add validation" becomes "write tests for invalid inputs, then make them pass"; "Fix the bug" becomes "write a test that reproduces it, then make it pass". For multi-step work, state a short plan with a verification check per step [2].

**Why:** a passing check is an objective stopping point. Anthropic's guide puts it this way: "Claude stops when the work looks done. Without a check it can run, 'looks done' is the only signal available" [3]. For UI work, where unit tests can't judge the result, give the agent a browser it can drive and screenshot. Karpathy suggests putting the agent "in the loop with a browser MCP" [1][3]. Topic 07 covers verification in depth.

The repository states the trade-off openly: these rules "bias toward caution over speed", and for trivial tasks you should use judgement [2].

## Project-specific content: what the agent can't guess

Running an `init` command produces a starting file, which Anthropic suggests you then "refine from there with instructions Claude wouldn't discover on its own" [4]. The best-practices guide gives an include/exclude list [3]:

| Include | Exclude |
|---|---|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions |
| Testing instructions and preferred test runners | Detailed API documentation (link instead) |
| Repository etiquette (branch naming, PR conventions) | Information that changes frequently |
| Architectural decisions specific to the project | Long explanations or tutorials |
| Environment quirks (required env vars) | File-by-file descriptions of the codebase |
| Common gotchas | Self-evident practices like "write clean code" |

Two practical consequences:

- **List tools only where they differ from the default.** "This project uses `pnpm`, not `npm`" earns its line. So does telling the agent to use a CLI such as `gh` for GitHub work, which the guide calls the most context-efficient way to reach external services [3].
- **Write instructions concrete enough to verify.** "Use 2-space indentation" instead of "format code properly"; "Run `npm test` before committing" instead of "test your changes" [4].

AGENTS.md suggests a similar set of sections: project overview, build and test commands, code style, testing instructions, security considerations, and PR guidelines [5].

## Safety rules belong in configuration, not prose

It's tempting to write "never force-push or run `rm -rf` without asking" into the instruction file. That line helps, but it's advisory. Claude Code's docs are explicit: "To block an action regardless of what Claude decides, use a PreToolUse hook" [4], and "Unlike CLAUDE.md instructions which are advisory, hooks are deterministic" [3]. Permission rules work the same way. Deny rules block in every mode, and `ask` rules force a confirmation prompt [7].

A sound split:

- **Settings (enforced):** deny or ask rules for destructive commands (force-push, hard reset, recursive delete, production deploys), shared through the project's committed settings file.
- **Instruction file (guidance):** the intent behind those rules and the judgement calls a rule list can't capture, for example "if you're unsure whether a command is reversible, ask."

## Keep it current: turn corrections into rules

Boris Cherny, who created Claude Code, describes how his team maintains its file: "Anytime we see Claude do something incorrectly we add it to the CLAUDE.md, so Claude knows not to do it next time." The file is checked into git and the team adds to it several times a week. During code review he tags Claude on colleagues' pull requests so the lesson is added as part of the PR [6].

Anthropic's memory docs give the triggers for adding a line: Claude makes the same mistake a second time, a code review catches something Claude should have known, or you type the same correction you typed last session [4]. As of 2026-10, Claude Code also keeps an **auto memory** where Claude records corrections and preferences on its own [4].

Adding is only half of it. The guide's advice is to "treat CLAUDE.md like code: review it when things go wrong, prune it regularly, and test changes by observing whether Claude's behavior actually shifts" [3].

## Structure: short, scoped, unambiguous

**Length.** Anthropic's target is under 200 lines per `CLAUDE.md`, because "longer files consume more context and reduce adherence." Imports (`@path`) help organise a long file but don't reduce its cost, because imported files also load at launch [4]. If the agent keeps ignoring a rule, the file is probably too long and the rule is getting lost [3].

**Path-scoped rules.** Claude Code loads markdown files in `.claude/rules/` and, if a rule file has a `paths` frontmatter field with glob patterns, loads it only when Claude works with matching files [4]:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API rules
- All endpoints validate input and use the standard error format.
```

**Monorepos.** Nested instruction files serve each package. In Claude Code, files in subdirectories load on demand when Claude works in that directory, and `claudeMdExcludes` can skip other teams' files [4]. With AGENTS.md, "the closest one takes precedence" [5]. Keep only system-wide rules at the root.

**Structure and emphasis.** Group instructions under headers and bullets. Remove contradictions, because when two instructions conflict the agent "may pick one arbitrarily" [4]. If one rule keeps being skipped, mark that line alone as "IMPORTANT". If you emphasise many lines, none stands out [3]. A common pattern is to open with a two- or three-line project overview, then the rules that matter most.

### A skeleton

```markdown
# Acme Notes
Shared-notes web app. Next.js frontend (apps/web), Node API (apps/api), Postgres.

## Commands
- pnpm (not npm). Dev: `pnpm dev`. Tests: `pnpm test`. Types: `pnpm typecheck`.
- Use the `gh` CLI for GitHub operations.

## How to work
- State assumptions. If the request is ambiguous, list readings and ask.
- Minimum code for the task; no speculative features or abstractions.
- Change only what the task needs; mention unrelated issues, don't fix them.
- Turn each task into a check (usually a test) and iterate until it passes.
- Done = tests, typecheck and lint pass; UI changes checked in the browser.

## Gotchas
- Migrations are generated, never hand-edited.
- Area rules live in .claude/rules/ (api.md, ui.md).
```

Destructive-command limits live in `.claude/settings.json`, not here. (This example is illustrative, not taken from a source.)

## What lasts vs. what will date

- **Durable principles:**
  - Instructions should correct defaults the agent gets wrong, not repeat what it can infer.
  - Clarify before building, make the smallest change that solves the problem, and define "done" as something checkable.
  - Prose is advisory. Enforce must-never-happen rules with deterministic controls.
  - Load rules only where they apply. Irrelevant instructions cost attention.
  - Grow the file from observed mistakes and prune it like code.
- **Time-sensitive details (as of 2026-10):** file names and loading rules (`CLAUDE.md`, `AGENTS.md`, `.claude/rules/` with `paths`), Claude Code's AGENTS.md fallback, the 200-line target, auto memory, the plugin packaging of the community guidelines, and the list of tools that read AGENTS.md.

## Try it yourself

1. **Prune.** For each line of an existing instruction file, ask "would removing this cause mistakes?" Delete the rest and count what went.
2. **A/B the four rules.** Add them in your own words. Run the same ambiguous request before and after, and compare clarifying questions and diff size.
3. **Move one safety rule into settings.** Take a "never do X" line and implement it as a deny or ask permission rule. Check that it holds even when you ask the agent to do X.
4. **Scope one area.** Move API- or UI-specific rules into a path-scoped rule file and confirm it loads only when you work there.
5. **Log corrections for a week.** Each time you correct the agent, decide whether the lesson belongs in the file. At the end of the week, prune again.

## Key terms

- **Instruction file:** a project file such as `CLAUDE.md` or `AGENTS.md` that the agent reads at the start of each session.
- **Surgical change:** an edit limited to what the task requires, where every changed line traces to the request.
- **Goal-driven execution:** turning a task into checkable success criteria, usually tests, and iterating until they pass.
- **Path-scoped rule:** a rule file that loads only when the agent works on files matching its glob patterns.
- **Advisory vs. enforced:** instruction-file text influences the model. Permission rules and hooks are applied by the client regardless of what the model decides.
- **Hook:** a script the agent harness runs at a fixed lifecycle point (for example before a tool call) that can block or modify actions.
- **Auto memory:** notes Claude Code writes for itself from your corrections and preferences.

## Sources

1. [A few random notes from claude coding quite a bit last few weeks](https://x.com/karpathy/status/2015883857489522876) (Andrej Karpathy, X, 2026-01-26). The original description of agent failure modes (wrong assumptions, unmanaged confusion, bloat, orthogonal edits), the note that CLAUDE.md instructions only partly fix them, and the "give it success criteria" lever.
2. [andrej-karpathy-skills: CLAUDE.md](https://github.com/forrestchang/andrej-karpathy-skills) (Jiayuan Zhang, GitHub user forrestchang, and contributors, 2026-01; now hosted at github.com/multica-ai/andrej-karpathy-skills, MIT). The four rules derived from Karpathy's post, their exact wording, the 200-to-50-lines heuristic, and the caution-over-speed trade-off.
3. [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (Anthropic, accessed 2026-10-14). Context as the main constraint, the "would removing this cause mistakes?" test, the include/exclude table, advisory CLAUDE.md vs. deterministic hooks, emphasis guidance, and verification.
4. [How Claude remembers your project (CLAUDE.md and memory)](https://code.claude.com/docs/en/memory) (Anthropic, accessed 2026-10-14). Load order, the 200-line target, writing concrete instructions, `.claude/rules/` with `paths`, nested files and monorepos, AGENTS.md support, auto memory, and the "context, not enforced configuration" caveat.
5. [AGENTS.md](https://agents.md/) (Agentic AI Foundation / Linux Foundation, accessed 2026-10-14). The open instruction-file format, recommended sections, nested files where the closest one wins, and the list of supporting tools.
6. [How I use Claude Code (thread)](https://x.com/bcherny/status/2007179832300581177) (Boris Cherny, X, 2026-01-02). How the Claude Code team grows a shared CLAUDE.md from observed mistakes and adds lessons during code review.
7. [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) (Anthropic, accessed 2026-10-14). Permission modes and the fact that deny rules block in every mode, the enforcement layer for safety rules.
