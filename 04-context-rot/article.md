# Context Rot: Why Bigger Context Windows Don't Fix Long Agent Sessions

> **Core idea in one sentence:** Models use long inputs less reliably as those inputs grow, well before the window is full, so long agent sessions depend on deciding after every task what stays in context: continue, rewind, clear, compact, or hand the noisy part to a subagent.

## TL;DR

- **Context rot** is the drop in output quality as the context window fills. Chroma measured it across 18 models, even on simple retrieval tasks [1]. Anthropic explains it as a finite "attention budget" that every extra token draws on [3].
- There is **no universal threshold**. How fast quality drops depends on the model, the task and what else is in the window [1][4]. A bigger window gives you more room, not immunity.
- Long contexts fail in recognisable ways: an error gets repeated (**poisoning**), history crowds out reasoning (**distraction**), irrelevant material steers the answer (**confusion**), and earlier wrong turns conflict with later information (**clash**) [4].
- Anthropic's Claude Code team treats **every turn as a branching point**: continue, rewind, clear, compact, or delegate to a subagent [5].
- **Compaction is lossy** and happens when the model is least sharp, so compact deliberately with a hint about what comes next. **Clear** for new tasks. **Rewind** instead of correcting a failed attempt. **Delegate** output-heavy steps when you only need the conclusion [5][6].

## What context rot is

Every model works inside a context window: everything it can see at once. In a coding agent that's far more than your messages. Claude Code's docs list what loads before you type anything: the system prompt, your `CLAUDE.md` files, auto memory, environment info, MCP tool names and skill descriptions. Then every file read, command output and tool result is added as the session goes on [6].

Chroma's 2025 technical report tested 18 models from Anthropic, OpenAI, Google and Alibaba on deliberately simple tasks and found that "models do not use their context uniformly" [1]. Its main findings:

- Performance **falls as input length grows**, even when the task stays the same.
- The drop is **faster when the question and the answer are less similar** in wording, which is the normal case in real work.
- **Distractors** (related but wrong statements) hurt. "Even a single distractor reduces performance relative to the baseline," and several compound the damage.
- On a memory benchmark, every model did much better on a **focused ~300-token prompt** than on the full ~113k-token chat history that contained the same answer.

An older result points the same way. *Lost in the Middle* (Liu et al.) found that models use information best when it sits at the **beginning or end** of a long input, and noticeably worse when it's in the middle. That held even for models built for long contexts [2].

Anthropic's context-engineering guide gives the mechanism in plain terms. Transformers relate every token to every other token, which means n² relationships for n tokens, and models see fewer very long sequences in training. The result is a gradual loss of precision rather than a cliff, so context should be treated as "a finite resource with diminishing marginal returns" [3].

### Bigger windows don't remove it

As of 2026-10, Claude Code supports a 1-million-token window on several models (Opus 4.6 and later, Sonnet 4.6, Sonnet 5 and later, Haiku 5.5, and the Fable models) [6]. Anthropic's Thariq Shihipar wrote his session-management guide for exactly this situation: a much larger window gives you room to work longer, but "model performance degrades as context grows" regardless [5]. No fixed token count marks where rot starts. Chroma's data shows it varies by task and model [1], and Drew Breunig's survey cites a Databricks study in which one large open model began losing accuracy around 32k tokens [4].

## How long contexts fail

Breunig groups long-context failures into four patterns, each backed by published experiments [4]:

| Failure | What happens | What it looks like in a coding session |
|---|---|---|
| **Context poisoning** | A hallucination or error enters the context and keeps being referenced | The agent "remembers" a function signature that doesn't exist and builds on it |
| **Context distraction** | The model leans on its long history instead of its training | It repeats earlier actions rather than trying a better approach |
| **Context confusion** | Irrelevant material, such as unused tool definitions, still shapes the output | It calls a tool that has nothing to do with the task |
| **Context clash** | Parts of the context contradict each other | A failed first attempt and your later correction pull in different directions |

The clash evidence matters for daily practice. In a Microsoft and Salesforce study that split benchmark prompts across several turns, average performance dropped 39%, and the researchers found that "when LLMs take a wrong turn in a conversation, they get lost and do not recover" [4]. That is the argument for removing failed attempts instead of arguing past them.

## Every turn is a branching point

When the agent finishes a step, your next action decides what happens to the context. Thariq's guide lists five options [5]:

| Option | Claude Code (as of 2026-10) | Use it when |
|---|---|---|
| **Continue** | Send another message | Same task, and the context is still relevant |
| **Rewind** | `Esc Esc` or `/rewind` | The agent went down a wrong path |
| **Clear** | `/clear` | You're starting a new task |
| **Compact** | `/compact <hint>` | Mid-task, the session is full of stale exploration |
| **Subagent** | Ask for a subagent | A step will produce lots of output and you only need the conclusion |

Continuing is the default, and often not the best choice.

### Compact: deliberately, with a hint

**Compaction** replaces the conversation with a summary and continues on top of it. It needs no work from you, but it is lossy [5]. Thariq explains why automatic compaction can go badly: it fires when the context is most rotted, so the model is "at its least intelligent point", and it has to guess what you'll do next. His example: after a long debugging session, autocompact summarises the investigation. Your next message is "now fix that other warning we saw in bar.ts", and that warning didn't make it into the summary [5].

How to compact well:

- **Do it yourself, before the automatic pass**, and say what comes next: `/compact focus on the auth bug fix` [6]. A large window gives you room to do this early [5].
- **Put standing compaction rules in `CLAUDE.md`**, such as "When compacting, always preserve the full list of modified files and any test commands" [6].
- **Move the automatic threshold** with `/autocompact` (for example `/autocompact 500k`) if you want it to fire earlier [6].
- **Compact part of the history**: in the `/rewind` menu, "Summarize from here" condenses everything after a chosen message and keeps earlier instructions intact [6].

Know what survives. After compaction, Claude Code re-injects the system prompt, project-root `CLAUDE.md`, auto memory and the plan-mode plan from disk, re-reads up to five recently modified files, and re-injects invoked skills up to a size cap. Instructions you gave only in conversation are summarised with everything else, so put anything that must persist into `CLAUDE.md` [6].

### Clear: a fresh start with a brief

**`/clear`** gives you an empty context. Thariq recommends a new session for each new task, usually "with a brief you've distilled from what you just learned". It's more work than compacting, but "the resulting context is what you decided was relevant" [5]. Claude Code's best-practice guide names the failure this prevents, the **kitchen-sink session**: one task, then something unrelated, then back to the first, until the context is full of irrelevant material [6].

A common pattern is to write the brief to a file before clearing: the goal, current state, decisions and constraints, known issues, and next steps. Then start the new session from that file. A structured format helps. Anthropic's long-running-agent harness kept its feature list in JSON because the model was less likely to wrongly rewrite JSON than Markdown [7].

### Rewind instead of correcting forward

When an attempt fails, the instinct is to reply "no, try another way". That leaves the failed attempt in context, where it can poison or clash with what follows [4]. Thariq's example: Claude reads five files, tries an approach, and it doesn't work. Rewind to just after the file reads and re-prompt with what you learned. You keep the useful reads and lose the dead end [5].

In Claude Code, `Esc Esc` or `/rewind` opens a menu that can restore the conversation, the code, or both, to any earlier prompt [6]. The best-practice guide adds a rule of thumb: if you've corrected Claude more than twice on the same issue, the context is "cluttered with failed approaches", so `/clear` and start again with a better prompt [6]. Note that rewind only restores files changed by Claude's edit tools, not by shell commands, so it doesn't replace git [6].

### Delegate output-heavy steps to subagents

A **subagent** runs in its own fresh context window and returns only its result. Its file reads, searches and logs never enter your main session [5][6]. Thariq's test is a single question:

> Will I need this tool output again, or just the conclusion?

His examples: verifying work against a spec, researching another codebase before implementing, and writing docs from git changes [5]. Claude Code's docs add that you can ask directly, as in "use subagents to investigate X", and that subagent requests still count toward your usage [6]. Topic 05 covers subagent architectures in more depth, and topic 23 compares subagents with agent teams.

### Restate goals in long tasks

A common pattern in long single-task sessions is to ask the agent now and then to restate the goal, constraints and decisions so far. *Lost in the Middle* suggests why this can help: information near the end of the input is used more reliably than information buried in the middle [2]. Treat it as a habit to test, not a guarantee, and prefer `CLAUDE.md` or a plan file for anything that must survive compaction.

## What lasts vs. what will date

- **Durable principles:**
  - Models use long inputs unevenly. Quality falls as context grows, and the rate depends on task and model.
  - Everything in the window competes for attention: tool output, old attempts, unused tool definitions.
  - Summaries lose information. Choose when to summarise and say what matters next.
  - Start unrelated work in a fresh context with a deliberate brief.
  - Remove failed attempts rather than correcting on top of them.
  - Push output-heavy steps into separate contexts and keep only the conclusion.
- **Time-sensitive details:**
  - Which models have 1M-token windows in Claude Code (as of 2026-10).
  - Claude Code commands and menus: `/compact`, `/autocompact`, `/clear`, `/rewind`, `Esc Esc`, Summarize from here, and what is re-injected after compaction (as of 2026-10).
  - The specific benchmark numbers in Chroma's report (July 2025 models) and the studies Breunig cites.

## Try it yourself

1. **Look at your starting context.** Run `/context` at the start of a session and note how much is used before your first prompt.
2. **Compact with a hint.** Next time you compact, add what comes next, for example `/compact keep the failing test names and the API contract; next I'll fix the retry logic`. Then ask about one specific earlier detail to see if it survived.
3. **Rewind once.** When an attempt fails, rewind to before it and re-prompt with what you learned, instead of correcting. Compare the next result with your usual approach.
4. **Clear with a brief.** At the end of a task, have the agent write a short handoff file (goal, state, decisions, open issues, next steps). Run `/clear` and start the next session from that file alone.
5. **Apply the subagent test.** Before a big search or test run, ask "will I need this output again, or just the conclusion?" If just the conclusion, delegate it.

## Key terms

- **Context window**: everything the model sees at once, including system prompt, memory files, tool definitions, files read, tool outputs and conversation.
- **Context rot**: the decline in output quality as the context grows, starting well before the window is full.
- **Attention budget**: Anthropic's term for the model's limited capacity to attend to tokens, which every added token draws on.
- **Context poisoning / distraction / confusion / clash**: Breunig's four ways long contexts fail.
- **Compaction**: replacing conversation history with a summary to free space. Lossy.
- **Clear**: starting a fresh session, ideally with a brief you wrote.
- **Rewind**: rolling the conversation (and optionally code) back to an earlier prompt so a failed attempt leaves the context.
- **Subagent**: a separate agent with its own context window that returns only its result.

## Sources

1. [Context Rot: How Increasing Input Tokens Impacts LLM Performance](https://www.trychroma.com/research/context-rot) (Kelly Hong, Anton Troynikov, Jeff Huber, Chroma, 2025-07-14). Experiments across 18 models showing performance falls with input length, and the effects of similarity, distractors and focused prompts.
2. [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172) (Nelson F. Liu et al., TACL, arXiv 2023-07, rev. 2023-11). The U-shaped finding that information in the middle of long inputs is used least reliably.
3. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Anthropic Applied AI team, 2025-09-29). Context rot, the attention budget, and the n² explanation.
4. [How Long Contexts Fail](https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html) (Drew Breunig, 2025-06-22). The poisoning/distraction/confusion/clash taxonomy and the studies behind it, including the multi-turn "get lost and do not recover" result.
5. [Using Claude Code: session management and 1M context](https://claude.com/resources/articles/using-claude-code-session-management-and-1m-context) (Thariq Shihipar, Anthropic, 2026-04-15). Every turn as a branching point, the bad-autocompact explanation, rewind over correction, clear with a brief, and the subagent test.
6. Claude Code documentation: [Best practices](https://code.claude.com/docs/en/best-practices), [Explore the context window](https://code.claude.com/docs/en/context-window), [Checkpointing](https://code.claude.com/docs/en/checkpointing) and [Subagents](https://code.claude.com/docs/en/sub-agents) (Anthropic, accessed 2026-10-14). Commands, what loads and what survives compaction, rewind behaviour and limits, and subagent context isolation.
7. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Justin Young, Anthropic, 2025-11-26). Structured handoff artifacts between sessions, and why JSON was preferred over Markdown for the feature list.
