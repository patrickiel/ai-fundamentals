---
theme: ../theme
chapter: '04'
header: 'AI Fundamentals · 04 · Context rot'
layout: cover
---

# Context Rot

**Quality drops as context grows, long before the window is full, so decide after every task what stays in context**

---

## What context rot is

- Output quality **falls as input length grows**, even on simple tasks
- Chroma tested **18 models**: "models do not use their context uniformly"
- Distractors hurt; even **one** lowers accuracy
- Focused ~300-token prompt beat the full ~113k history on the same question

<!-- Chroma Context Rot report, July 2025. The point is not one model or one benchmark: the pattern held across vendors. -->

---

## Why it happens

- *Lost in the Middle*: info at the **start or end** is used best, the **middle** worst
- Anthropic: an **attention budget** that every token draws on
  - n tokens → n² relationships
  - fewer very long sequences in training
- A gradual slope, not a cliff

<!-- Liu et al. 2023 and Anthropic's "Effective context engineering" (Sept 2025). -->

---

## Bigger windows don't fix it

- Claude Code offers **1M-token** windows on several models (as of 2026-10)
- More room to work, but performance still "degrades as context grows"
- **No universal threshold**: depends on model, task, and what else is loaded
- Before you type: system prompt, `CLAUDE.md`, memory, tool names, skill descriptions

<!-- Thariq Shihipar's guide gives no fixed token number. Chroma's data shows the rate varies. Run /context to see your own starting load. -->

---

## How long contexts fail

| Failure | In a coding session |
|---|---|
| **Poisoning** | A hallucinated fact keeps being reused |
| **Distraction** | Repeats past actions instead of reasoning |
| **Confusion** | Irrelevant tools or text steer the answer |
| **Clash** | A failed attempt contradicts your correction |

> "When LLMs take a wrong turn in a conversation, they get lost and do not recover."

<!-- Drew Breunig's taxonomy, June 2025. The quote is from the Microsoft/Salesforce multi-turn study he cites (average drop of 39%). -->

---

## Every turn is a branching point

| Option | Claude Code | When |
|---|---|---|
| Continue | next message | Same task, context still relevant |
| **Rewind** | `Esc Esc`, `/rewind` | Went down a wrong path |
| **Clear** | `/clear` | New task |
| **Compact** | `/compact <hint>` | Mid-task, bloated with stale exploration |
| **Subagent** | ask for one | Only the conclusion matters |

<!-- This is the closing table from Thariq Shihipar's Anthropic article (April 2026). Continuing is the default, and often not the best. -->

---

## Why autocompact goes wrong

- Compaction **summarises** the history, which loses detail
- It fires when rot is worst: the model is at its **least sharp**
- It must guess where you're going next
- Example: after a long debug, "now fix that other warning in bar.ts", and the warning didn't survive the summary

<!-- From Thariq's "what causes a bad autocompact" section. The fix is to compact before the automatic pass, with a hint. -->

---

## Compact deliberately

- `/compact focus on the auth bug fix`: say what comes next
- Standing rules in `CLAUDE.md`: "when compacting, preserve modified files and test commands"
- `/autocompact 500k` to move the automatic threshold
- `/rewind` → **Summarize from here** to condense only part of the session
- Re-injected after compaction: system prompt, root `CLAUDE.md`, auto memory, plan

<!-- Instructions given only in conversation get summarised. Anything that must persist belongs in CLAUDE.md. All command names as of 2026-10. -->

---

## Clear for new tasks, with a brief

- New task → **new session**
- Bring a brief "distilled from what you just learned"
- Avoid the **kitchen-sink session**: unrelated tasks piling up in one context
- Common pattern: write the brief to a file, `/clear`, start from the file
  - goal · state · decisions · open issues · next steps
  - structured formats (e.g. JSON) are harder for the model to rewrite carelessly

<!-- Anthropic's long-running-agent harness kept its feature list in JSON because the model was less likely to wrongly edit it than Markdown. -->

---

## Rewind, don't correct forward

- Correcting leaves the failed attempt in context → **clash, poisoning**
- Example: reads 5 files, approach fails → rewind to **after the reads**, re-prompt with what you learned
- Corrected more than twice? `/clear` and write a better prompt
- Rewind restores Claude's file edits, **not** shell-command changes: keep using git

<!-- Sources: Thariq's article for the example, Claude Code best practices for the "more than twice" rule, checkpointing docs for the limitation. -->

---

## Delegate output-heavy steps

```
Main session ──task──► Subagent (fresh context)
                         reads · searches · logs
Main session ◄─result── conclusion only
```

> Will I need this tool output again, or just the conclusion?

- Verify against a spec · research another codebase · write docs from git changes
- Subagent work still counts toward usage

<!-- Topic 05 covers subagent architecture; topic 23 compares subagents with agent teams. -->

---

## Restate goals in long tasks

- Ask now and then for: goal · constraints · decisions so far
- Puts key facts near the **end** of the context, where they're used more reliably
- A habit to test, not a guarantee
- Anything that must survive compaction → `CLAUDE.md` or a plan file

<!-- This is a common practice, not a vendor feature. The rationale borrows from the Lost in the Middle positional finding. -->

---

## What lasts vs. what will date

**Lasts**
- Models use long inputs unevenly; quality falls as context grows
- Summaries lose information: choose when, and say what's next
- Fresh contexts for new work; remove failed attempts
- Isolate output-heavy steps, keep conclusions

**Will date**
- Which models have 1M windows
- Claude Code commands, menus, re-injection rules
- Specific benchmark numbers

<!-- The habits transfer to any agent. The command names will change. -->

---

## Try it yourself

1. Run `/context` before your first prompt
2. Compact with a hint about what's next; check an early detail survived
3. Next failed attempt: rewind instead of correcting
4. End a task with a handoff file, `/clear`, resume from the file
5. Before a big search, ask the subagent question

<!-- Each exercise isolates one of the five options from the branching-point table. -->

---
layout: end
---

# Key takeaways

- **Bigger windows don't stop rot**: manage context long before it's full
- **Compact with a hint, clear with a brief, rewind instead of correcting**
- **Delegate** steps whose output you won't need again

[Thariq Shihipar: Session management and 1M context](https://claude.com/resources/articles/using-claude-code-session-management-and-1m-context)\
[Chroma: Context Rot](https://www.trychroma.com/research/context-rot)\
[Drew Breunig: How Long Contexts Fail](https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html)
