# Getting Better Answers from LLMs: Clear Prompts, Managed Context, and the Right Tools

> **Core idea in one sentence:** Answer quality depends less on which model you use than on how you use it: clear and specific instructions, real material to work from, a context window kept focused, standing instructions saved where they belong, and the right tool for the job.

## TL;DR

- All three major providers give the same first piece of advice: **be clear, direct and specific** about the task, the constraints and the output format [1][2][3]. Explaining *why* a constraint exists helps further [1].
- **Structure** longer prompts with Markdown headings or XML tags, and **show examples** of the output you want [1][2][3].
- The **context window** is the model's working memory. In a chat, every turn resends the whole conversation, and accuracy degrades as the window fills ("context rot") [4]. **New task, new chat.**
- Give the model **material, not just questions**: documents, links, images. It works better from your facts than from its own recall [1][3].
- Save repeated instructions in **persistent places**: projects with their own instructions and files, and memory you review from time to time [5][6].
- Much of what modern assistants can do comes from **tools around the model**: search, multi-step research, code execution, and agents that act on files [7][8].

## Part 1: Writing the request

### Be clear, direct and specific

Anthropic's guide suggests thinking of the model as "a brilliant but new employee who lacks context on your norms and workflows". Its test: show your prompt to a colleague with no context on the task. If they would be confused, the model will be too [1]. Google's guide makes the same point, telling you to give "clear and specific instructions" and to state constraints such as a length limit [3]. OpenAI's guide likens a non-reasoning model to "a junior coworker" who needs explicit instructions [2].

In practice, replace fuzzy words with targets:

| Vague | Specific |
|---|---|
| "Not too long" | "Under 150 words" |
| "Make it better" | "Rewrite for a non-technical manager; keep the three key numbers" |
| "Summarise this" | "Five bullets, each under 15 words, then one sentence on risks" |

State the **output format** too: a table, a checklist, a formal email, a JSON object [1][3]. If you want more than the minimum, ask for it explicitly rather than hoping the model infers it [1].

### Say why, not just what

Anthropic's guide shows that a bare rule ("NEVER use ellipses") works less well than a rule with its reason ("your response will be read aloud by a text-to-speech engine, so never use ellipses"). The model can then generalise from the explanation to cases the rule didn't name [1]. The same applies to everyday requests: "under 200 words, because it goes in a newsletter sidebar" tells the model what to optimise for.

### Structure and examples

For anything longer than a few lines, separate the parts of the prompt. All three guides recommend Markdown headings or XML tags so the model can tell instructions, context, examples and input apart [1][2][3]. OpenAI suggests a developer message usually covers identity, instructions, examples and context, in roughly that order [2].

**Examples** are one of the most reliable ways to steer format and tone. Anthropic recommends three to five that are relevant, varied, and wrapped in tags so they aren't mistaken for instructions [1]. Google goes further and recommends always including few-shot examples, while warning that too many can make the model overfit to them [3]. (Reasoning models are a partial exception. Topic 02 covers why some vendors say to try zero-shot first.)

### Iterate

Google's guide says plainly that "prompt engineering is iterative": rephrase, reorder the content, or reframe the task, and test again [3]. A first answer that misses is normal. Give feedback in the next turn, and once you find wording that works, save it (Part 3).

### When a task needs reasoning

Some problems need several steps of work before an answer. Two research ideas underpin today's tools. **Chain-of-thought** prompting showed that models do better on multi-step problems when they write out intermediate steps. **ReAct** (reason + act) interleaves reasoning with actions such as searches or tool calls, so each observation informs the next step [8]. Current "thinking" or "reasoning" modes build this in. Turn it on for multi-step problems and leave it off for simple lookups, because thinking costs time and tokens. Topic 02 covers how to prompt these models.

## Part 2: The context window

The **context window** is "all the text a language model can reference when generating a response, including the response itself". It works as the model's working memory and is separate from what it learned in training [4]. Two consequences matter for everyday use:

1. **The model has no memory between turns.** Each turn's input is "all previous conversation history plus the current user message", and the reply becomes input for the next turn [4]. Everything counts toward the budget: system prompt, messages, uploaded files and images, tool definitions and results [4].
2. **More context is not automatically better.** As the token count grows, "accuracy and recall degrade, a phenomenon known as *context rot*" [4]. Chat apps may also drop the oldest turns once the window is full [4]. Topic 04 covers this in depth.

The practical rule follows: **start a new chat for each new task**, and when you continue a long piece of work, open the new chat with a short recap of what matters. Claude Code's guide gives the same advice for agents: a clean session with a better prompt usually beats a long one cluttered with failed attempts [7].

## Part 3: Give it material, and save what you repeat

### Material, not just questions

Models answer better from sources you supply than from what they remember. Anthropic's guide has specific advice for long inputs: put long documents near the **top** of the prompt and your question at the **end**, which in its tests improved quality by up to 30% on complex multi-document inputs [1]. Google's guide warns against assuming the model already has all the information it needs, and says to supply it [3]. In chat apps that means attaching PDFs, pasting links, and adding screenshots or diagrams. Current Claude models accept up to 600 images or PDF pages in a single API request (as of 2026-10) [4].

### Projects and standing instructions

If you keep pasting the same instructions into new chats, they belong in a **project**. In Claude, a project is a self-contained workspace with its own chat history, a **knowledge base** of uploaded files that serves as background for every chat in it, and **project instructions** that shape responses [5]. ChatGPT and Gemini offer similar workspaces under their own names (as of 2026-10).

The provider guides suggest what to put in those instructions:

- **A role.** Anthropic notes that "even a single sentence makes a difference" in focusing behaviour and tone [1].
- **The task, constraints and output format**, written as specifically as in Part 1 [1][3].
- **A sample output**, because examples steer format more reliably than descriptions [1][2].
- **Clear sections** in Markdown or XML [1][2].

```markdown
# Role
You are a careful editor who fact-checks and grammar-checks essays.

# Task
For each essay, list the factual claims that need a source, then the grammar fixes.
Readers are first-year students, so explain each fix in one short sentence.

# Output format
## Claims to verify
- ...
## Grammar fixes
- original → corrected (reason)
```

Projects work well for **recurring tasks** (weekly reports, essay checking) and for **one-off but document-heavy work** where many questions draw on the same files.

### Memory

Assistants also keep longer-term memory across chats. Claude's memory, for example, is built from your conversations "as a set of individual topics"; you can view, edit or delete them under Settings > Memory, turn memory off for a single chat, or use incognito chats that are neither saved nor searched. Each project has its own separate memory space (as of 2026-10) [6]. Claude can also search past chats when asked [6]. Other assistants have equivalent features with different names and controls.

Memory and custom instructions do the same job as a project: they place relevant information in the context automatically. That makes them worth auditing. A wrong or outdated memory skews answers without you noticing.

## Part 4: Tools around the model

### Search vs. research

**Web search** runs a query and answers from the results: fast, but only as deep as the first pages it reads. **Research** modes run a sequence of searches, each informed by the last, explore different angles on their own, and return a cited report in minutes rather than seconds, as Anthropic described when it launched its Research feature [7]. OpenAI and Google offer comparable deep research modes (as of 2026-10). Use quick search when speed matters and research when coverage does, and check the citations either way.

### Edit in place instead of regenerating

A common frustration is asking for one small change and getting a full rewrite. Most assistants now open longer outputs in an editable panel beside the chat (Claude calls these artifacts, ChatGPT calls it Canvas, as of 2026-10). A good habit is to ask for changes to a **named section only**, and to keep earlier versions you might want to return to.

### Code execution and coding agents

Google's guide recommends turning on code execution for arithmetic, counting and calculation, and search grounding for recent or obscure facts [3]. For software work, terminal coding agents such as Claude Code, OpenAI's Codex and Google's Gemini CLI read and change files across a whole project (names as of 2026-10). Claude Code's guide describes the difference from chat: the agent reads files, runs commands and works through problems instead of answering and waiting [7]. Topics 03, 06 and 21 cover working with agents.

## What lasts vs. what will date

- **Durable principles:**
  - Specific instructions with stated reasons beat vague adjectives. Iterating is normal.
  - Structure long prompts and show examples of the output you want.
  - The model is stateless between turns. Everything it "knows" in a chat is context that is resent, and quality degrades as that context grows. New task, new chat.
  - Supplying source material beats relying on the model's memory. Long documents go first, the question last.
  - Save repeated instructions in projects or system prompts, and audit what memory has stored.
  - Match the tool to the job: search vs. research, regenerate vs. targeted edit, thinking vs. standard mode.
- **Time-sensitive details:**
  - Product names, menus and plan limits (Projects, Memory settings, Research, artifacts/Canvas), as of 2026-10.
  - Context window sizes and per-request file limits.
  - Which coding agents exist and what they're called.
  - Vendor differences in advice, such as how many examples to include.

## Try it yourself

1. **Vague vs. specific.** Ask for a summary "not too long", then ask for "five bullets, each under 15 words, for a busy executive". Compare.
2. **Add the why.** Take one rule from a prompt you use and add a one-line reason. See whether the output handles cases the rule didn't name.
3. **Watch context degrade.** Work on one task for a long time in a single chat. When answers start repeating or drifting, start a new chat with a one-paragraph recap and compare.
4. **Build one project.** Pick a task you repeat weekly. Write instructions with a role, task, output format and one sample output. Use it three times and refine it.
5. **Audit your memory.** Open your assistant's memory settings, read what it has stored, and delete anything wrong or outdated.

## Key terms

- **Context window**: all the text a model can reference when generating a response, including the conversation resent on each turn and the response itself.
- **Token**: a chunk of text, often part of a word, used to measure model input and output.
- **Context rot**: the drop in accuracy and recall as the amount of text in the context grows.
- **Few-shot prompting**: including a small number of example inputs and outputs in the prompt to show the pattern you want.
- **System prompt / project instructions**: standing instructions that set the model's role and behaviour for a project or application.
- **Chain-of-thought**: having a model work through intermediate steps before answering.
- **ReAct**: a pattern where a model alternates between reasoning and actions (e.g. tool calls), using each result to guide the next step.
- **Research mode**: an agentic search feature that runs many dependent searches and returns a cited report.

## Sources

1. [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) (Anthropic, Claude docs, accessed 2026-10-14). Be clear and direct, the "new employee" framing and colleague test, adding the reason behind instructions, examples, XML structure, roles, and long-document placement.
2. [Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering) (OpenAI, API docs, accessed 2026-10-14). Message roles, Markdown and XML formatting, few-shot examples, the order of a developer message, and the "junior coworker" analogy.
3. [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) (Google, Gemini API docs, accessed 2026-10-14). Clear and specific instructions, constraints, response formats, the recommendation to include few-shot examples, iteration, and when to enable search grounding and code execution.
4. [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows) (Anthropic, Claude docs, accessed 2026-10-14). Definition of the context window, how turns accumulate, what counts toward it, context rot, and per-request limits.
5. [What are projects?](https://support.claude.com/en/articles/9517075-what-are-projects) (Anthropic, Claude Help Center, accessed 2026-10-14). Projects as workspaces with a knowledge base and project instructions.
6. [Using Claude's chat search and memory to build on previous context](https://support.claude.com/en/articles/11817273) (Anthropic, Claude Help Center, accessed 2026-10-14). How memory is stored, viewed, edited and paused, incognito chats, and project-scoped memory.
7. Anthropic: [Claude takes research to new places](https://claude.com/blog/research) (2025-04-15) and [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (accessed 2026-10-14). How agentic research differs from single-query search; how an agent differs from chat, and why a fresh session beats a cluttered one.
8. [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) (Yao et al., arXiv 2022-10, rev. 2023-03). Interleaving reasoning traces with actions so each observation guides the next step.
