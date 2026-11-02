# Context Engineering for AI Coding: Give the Model the Right Information at the Right Time

> **Core idea in one sentence:** Instead of crafting one clever prompt, you design the information an AI coding agent works from (requirements, plans, conventions, known issues) and arrange it so a small core is always loaded and everything else is pulled in only when the current step needs it.

## TL;DR

- **Context engineering** is "the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference" [3]. Prompt engineering, the wording of instructions, is one part of it [3][4].
- The term spread in June 2025 after Shopify CEO Tobi Lütke [1] and Andrej Karpathy [2] argued it described the real skill better than "prompt engineering".
- **More context is not better.** Recall degrades as context grows ("context rot"), so the goal is the "smallest possible set of high-signal tokens" for the task [3].
- A practical pattern: write a **requirements doc**, turn it into a **written plan** and a few **focused reference files**, keep the **always-on rules short**, and load the rest **just in time** [3][5][6].
- Because the plan lives in files, a **fresh session can pick up the work** [6].
- Agents follow written instructions literally. **Contradictory instructions give unpredictable results**, so review what you feed the agent and what it generates [6].
- **Parallel subagents** help with independent tasks, not with steps that depend on each other [6].

## From prompt engineering to context engineering

On 19 June 2025, Tobi Lütke wrote that he preferred "context engineering" over prompt engineering because it describes the core skill better: "the art of providing all the context for the task to be plausibly solvable by the LLM" [1]. A week later Andrej Karpathy added "+1", arguing that people associate prompts with short task descriptions, whereas in real LLM applications context engineering is "the delicate art and science of filling the context window with just the right information for the next step" [2]. Neither invented the practice. They gave a name to something builders were already doing.

Karpathy listed what goes into that context: task descriptions and explanations, few-shot examples, retrieved data (RAG), related and possibly multimodal data, tools, state and history, and compaction. Too little context hurts performance, and too much raises cost and can also hurt it [2]. Philipp Schmid's breakdown is a useful checklist of the same idea: system instructions, the user prompt, conversation state, long-term memory, retrieved information, available tools, and the required output format [4]. The prompt is one item on the list.

Karpathy's second point is that the **application around the model** matters. A serious LLM app also has to split problems into control flows, route calls to suitable models, handle verification and guardrails, and more. He rejected "ChatGPT wrapper" as a description of such systems [2]. Coding agents such as Cursor and Claude Code are examples: they supply file access, tools, task lists and rule systems that make context engineering practical.

Anthropic's engineering team gave the most precise definition. Prompt engineering is about "writing and organizing LLM instructions for optimal outcomes". Context engineering covers everything else that can land in the context window: system instructions, tools, external data, message history and more. It is iterative, because curation happens every time the model is called [3].

## The central tension: complete context vs. a limited window

It's tempting to give the model everything. Anthropic explains why that fails. As the number of tokens grows, the model's ability to recall information from the context decreases, which it calls **context rot**. Models have an "attention budget", and every new token draws on it [3]. Claude Code's guide says the same thing in practical terms: when the window gets full, the agent may start "forgetting" earlier instructions or making more mistakes [6]. Topic 04 covers the evidence.

The guiding principle is to find "the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome" [3]. In practice:

- Split project knowledge into **separate, focused pieces**.
- Keep anything that is **always loaded** as small as possible.
- Load the rest **just in time**. Anthropic describes agents that keep lightweight references (file paths, stored queries, links) and pull data in with tools when needed. Claude Code uses a hybrid: its `CLAUDE.md` files go into context at the start, while search tools such as glob and grep fetch files on demand [3].
- Include **relevant external documentation**, such as library docs, by reference, so the agent can read it when it reaches that step [5][6][7].

## A workflow: requirements, plan, focused files, short rules

The following pattern is common among practitioners and matches the vendors' guidance. Cole Medin's public template is one worked example: a feature request file lists the feature, examples, documentation links and other considerations; one command turns it into a detailed implementation plan with validation steps, and a second command executes that plan [7].

### 1. Start with a requirements document

A **PRD** (product requirements document) or spec lists what you want built. Add the technical constraints you've already decided, especially the tech stack. Claude Code's guide suggests having the agent interview you about implementation, UI, edge cases and trade-offs, then write the spec to a file. It says the most useful specs name the files and interfaces involved, state what is out of scope, and end with a verification step [6]. Topic 08 covers spec-driven development in depth.

### 2. Generate a plan and focused reference files

Have the agent turn the requirements into documents it will consult later. One useful split:

| File | What the agent uses it for |
|---|---|
| **Implementation plan** | Staged tasks with subtasks and a check for each stage |
| **Project structure** | The intended folder and file layout, checked before creating files |
| **UI/UX conventions** | Design rules and patterns for interface work |
| **Known issues** | Errors already seen and how they were fixed, checked before debugging |

Generating these files can fill a session's context. That's fine, because generation is a separate step. Once the files exist, the knowledge lives on disk and later sessions don't have to hold it all.

### 3. Keep the always-on rules short

The files only help if the agent knows when to read them. A short, always-on rule can map situations to files:

```text
- Implementing a feature    → read docs/implementation-plan.md
- Working on UI             → read docs/ui-conventions.md
- Creating files or folders → check docs/project-structure.md
- Hitting an error          → check docs/known-issues.md first; log new fixes there
```

Both major coding tools support this split (as of 2026-10):

- **Cursor** project rules live in `.cursor/rules` and can be applied always, when the agent judges them relevant from their description, when matching files are in context, or only when @-mentioned. Cursor's docs say to keep rules under 500 lines, split large rules into composable ones, and reference files instead of copying their contents [5].
- **Claude Code** loads `CLAUDE.md` at the start of every session. Its docs recommend under 200 lines per file, because "longer files consume more context and reduce adherence". Instructions that matter only for part of the codebase can go in path-scoped rules under `.claude/rules/`, which load only when Claude works with matching files. Note that `@imports` help organise a file but don't reduce its cost, because imported files load at launch too [6].

Claude Code's guide gives a test for each always-on line: "Would removing this cause Claude to make mistakes?" If not, cut it [6].

### 4. Let the plan drive the work, and survive resets

With a written plan, you can start a **fresh session** and ask the agent to continue from the next stage. Anthropic's prompting guide notes that current models are "extremely effective at discovering state from the local filesystem", and recommends progress notes, structured test files and git history as the state a new session reads first [8]. Claude Code's guide suggests writing the spec in one session and starting a new one to implement it, so the implementation session has clean context [6].

This is the main benefit of the approach: **project memory that survives a context reset.** Topic 05 covers compaction and note-taking in more depth.

### 5. Build foundations first, and verify each stage

Put project setup, structure and core configuration at the front of the plan, before features. Give each stage a check the agent can run, such as tests, a build or a screenshot comparison. Claude Code's guide calls verification the difference between a session you have to watch and one you can leave running [6]. Topic 07 covers verification in depth.

## Pitfalls

### Contradictory instructions

Agents follow written instructions literally. If your request says "MVP" but your planning rule describes stages for a complete product, the plan may follow the more detailed text and come out far larger than you wanted. Claude Code's docs are explicit: "if two instructions contradict each other, Claude may pick one arbitrarily" [6]. Read what you feed the agent, review generated plans before building from them, and prune rule files regularly [6].

### Letting the agent choose key decisions

A workflow that proposes a tech stack may pick components that fit the requirements but not your situation, such as an API you don't have access to. Decide the stack yourself and write it into the requirements.

### Bloated or copied rule files

Long rule files get ignored: "If Claude keeps doing something you don't want despite having a rule against it, the file is probably too long and the rule is getting lost" [6]. Cursor's advice is to start small and add a rule only when the agent repeats the same mistake [5]. Someone else's templates are a starting point. Build files that match how you work.

## Parallel subagents: only for independent work

Claude Code can run **subagents**, each in "its own context window with a custom system prompt, specific tool access, and independent permissions" [6]. They keep exploration out of the main conversation and can run in parallel. The docs say parallel research "works best when the research paths don't depend on each other", and for multi-step workflows recommend running subagents in sequence, passing results from one to the next [6]. Generating several UI variations or researching separate modules suits parallel subagents. A staged build where each step depends on the last doesn't. Topics 23 and 27 cover this further.

## What lasts vs. what will date

- **Durable principles:**
  - Agent output depends on the information in its context, and you can design that information.
  - More context isn't automatically better. Keep always-on instructions small and load the rest when relevant.
  - Keep durable project knowledge in **files**, not chat history, so any fresh session can pick it up.
  - Split knowledge into focused documents with a short map of when to use each.
  - Contradictory instructions give unpredictable results. Review plans and prune rules.
  - Make key decisions, like the tech stack, yourself.
  - Parallelism helps only with independent tasks.
- **Time-sensitive details:**
  - Rule mechanisms and limits (`.cursor/rules` and its four modes, `CLAUDE.md`, `.claude/rules/`, the 500- and 200-line guidance), as of 2026-10.
  - Subagent features and concurrency limits.
  - Context window sizes of particular models.
  - The "context engineering" label itself. The practice is older than the term.

## Try it yourself

1. **Write a one-page requirements doc** for a small app: features, users, what's out of scope, and your chosen stack with a one-line reason for each choice.
2. **Split your project knowledge.** For an existing project, write four short files: plan, structure, UI conventions, known issues. Notice what you hadn't written down.
3. **Write an always-on rule under 10 lines** that maps situations to those files, and set it in your coding tool.
4. **Test a context reset.** Start a fresh session and ask the agent to continue from the plan with no other explanation. Note what's missing from your files.
5. **Hunt for contradictions.** Reread your rules and requirements for instructions that conflict ("MVP" vs. "full feature set"). Fix them and regenerate the plan.

## Key terms

- **Context engineering**: curating everything the model sees at inference time (instructions, tools, data, history, memory) so the task is plausibly solvable.
- **Prompt engineering**: writing and organising the instructions themselves. One part of context engineering.
- **Context rot**: the drop in recall and accuracy as the context grows.
- **Just-in-time context**: keeping references (paths, links, queries) and loading the content with tools only when it's needed.
- **PRD (product requirements document)**: a document stating what the product should do, and optionally technical constraints.
- **Always-on rule**: an instruction file loaded into every agent session. Keep it small.
- **Path-scoped rule**: a rule that loads only when the agent works with matching files.
- **Subagent**: a separate agent instance with its own context window, used for a focused task and returning a summary.

## Sources

1. [Tobi Lütke on "context engineering"](https://x.com/tobi/status/1935533422589399127) (Tobi Lütke, X, 2025-06-19). The preference for "context engineering" and the definition "providing all the context for the task to be plausibly solvable by the LLM".
2. [Andrej Karpathy: "+1 for 'context engineering' over 'prompt engineering'"](https://x.com/karpathy/status/1937902205765607626) (Andrej Karpathy, X, 2025-06-25). The "filling the context window with just the right information for the next step" framing, what goes into context, and the point about the application layer around the model.
3. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Anthropic Engineering, Rajasekaran et al., 2025-09-29). Definition vs. prompt engineering, context rot and the attention budget, smallest high-signal token set, just-in-time retrieval, and Claude Code's hybrid approach.
4. [The New Skill in AI is Not Prompting, It's Context Engineering](https://www.philschmid.de/context-engineering) (Philipp Schmid, 2025-06-30). A component checklist of what makes up an agent's context.
5. [Rules](https://cursor.com/docs/context/rules) (Cursor docs, accessed 2026-10-14). Project rule types and application modes, the 500-line guidance, composable rules, and referencing files instead of copying them.
6. Claude Code documentation: [How Claude remembers your project](https://code.claude.com/docs/en/memory), [Best practices](https://code.claude.com/docs/en/best-practices) and [Subagents](https://code.claude.com/docs/en/sub-agents) (Anthropic, accessed 2026-10-14). `CLAUDE.md` size and consistency guidance, path-scoped rules, the spec-then-fresh-session workflow, verification, and when subagents should run in parallel or in sequence.
7. [context-engineering-intro](https://github.com/coleam00/context-engineering-intro) (Cole Medin, GitHub, accessed 2026-10-14). A public template for the requirements-to-plan-to-execution workflow, including documentation links in the feature request.
8. [Prompting best practices: Agentic systems](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) (Anthropic, Claude docs, accessed 2026-10-14). Working across multiple context windows, starting fresh vs. compacting, and keeping state in progress notes, test files and git.
