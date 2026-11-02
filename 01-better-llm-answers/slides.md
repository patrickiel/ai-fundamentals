---
theme: ../theme
chapter: '01'
header: 'AI Fundamentals · 01 · Better LLM answers'
layout: cover
---

# Getting Better Answers from LLMs

**Answer quality depends less on the model than on how you use it: clear instructions, real material, focused context, saved instructions, the right tool**

---

## The problem

- Weak answers usually come from **vague requests**, **overloaded chats**, or the **wrong tool**
- Anthropic, OpenAI and Google publish prompting guides, and their first advice is the same
- The fix is mostly **how** you use the model, not **which** model

<!-- This topic draws on the three providers' own prompting guides plus Anthropic's docs on context windows, projects and memory. -->

---

## Be clear, direct and specific

| Vague | Specific |
|---|---|
| "Not too long" | "Under 150 words" |
| "Make it better" | "Rewrite for a non-technical manager" |
| "Summarise this" | "5 bullets, each < 15 words" |

- Treat the model like a **brilliant new employee** with no context
- **Colleague test:** would someone new to the task understand your prompt?
- State the **output format**; ask explicitly for more than the minimum

<!-- Anthropic's guide gives the new-employee framing and the colleague test; OpenAI compares non-reasoning models to a junior coworker; Google says to give clear and specific instructions with constraints. -->

---

## Say why, not just what

> "NEVER use ellipses"

vs.

> "Your response will be read aloud by a text-to-speech engine, so never use ellipses"

- With the reason, the model can **generalise** to cases the rule didn't name
- Everyday version: "under 200 words, **because** it goes in a newsletter sidebar"

<!-- The ellipses example is from Anthropic's prompting best practices. -->

---

## Structure and examples

- Separate **instructions, context, examples, input** with Markdown headings or XML tags (all three guides)
- **Examples** are among the most reliable ways to steer format and tone
  - Anthropic: 3–5, relevant, varied, wrapped in tags
  - Google: always include some, but not so many the model overfits
- **Iterate:** rephrase, reorder, reframe, test again

<!-- Reasoning models are a partial exception on examples; topic 02 covers the zero-shot-first advice from OpenAI and DeepSeek. -->

---

## When the task needs reasoning

- **Chain-of-thought:** work through intermediate steps before answering
- **ReAct:** reason → act (search, tool call) → observe → reason again
- Today's **thinking / reasoning modes** build this in
- Use them for **multi-step** problems; skip them for simple lookups (they cost time and tokens)

<!-- ReAct is from Yao et al., 2022. Topic 02 covers how to prompt reasoning models. -->

---

## The context window = working memory

```
Turn 1:  [system][you][model]
Turn 5:  [system][you][model][you][model]...[you]   ← all resent every turn
Turn 40: [....early material....][recent]          ← accuracy degrading
```

- The model is **stateless**: each turn's input is the whole history + your new message
- System prompt, files, images, tools all count toward the budget
- As tokens grow, accuracy and recall degrade: **context rot**
- **New task, new chat** (with a short recap if you're continuing)

<!-- Definitions and the accumulation diagram are from Anthropic's context windows docs. Topic 04 goes deeper on context rot. -->

---

## Give it material, not just questions

- Attach **documents, links, screenshots, diagrams**
- Don't assume the model already has the facts; **supply them**
- Long inputs: put documents at the **top**, the question at the **end**
  - Anthropic reports up to **30%** better quality on complex multi-document inputs

<!-- Your facts beat the model's recall. The ordering advice and the 30% figure are from Anthropic's prompting guide. -->

---

## Projects: save what you repeat

- A project = its own chats + **knowledge base** (files) + **project instructions**
- Good instructions include:
  - A **role** ("even a single sentence makes a difference")
  - **Task, constraints, output format**
  - A **sample output**
  - Clear **Markdown/XML sections**
- Use for **recurring tasks** and **document-heavy one-offs**

<!-- Project structure is described in Claude's help center; ChatGPT and Gemini have equivalents under other names as of 2026-10. -->

---

## Memory: useful, but audit it

- Assistants build memory from your chats and use it in new ones
- In Claude (as of 2026-10): view/edit/delete in **Settings > Memory**, turn off per chat, **incognito** chats, separate memory **per project**
- Memory and custom instructions put context in **automatically**
- A wrong or stale memory **quietly skews** answers

<!-- Feature details are from Claude's help center and will change; the habit of auditing is what lasts. -->

---

## Tools around the model

| | Web search | Research mode |
|---|---|---|
| How | One query, answer from results | Many searches, each informed by the last |
| Time | Seconds | Minutes |
| Output | Short answer | Cited report |

- **Edit in place** (artifacts, Canvas): change a named section, don't regenerate everything
- **Code execution** for arithmetic and counting; **search grounding** for recent facts
- **Coding agents** (Claude Code, Codex, Gemini CLI) act on whole projects

<!-- Research description from Anthropic's Research launch post; code execution and grounding advice from Google's guide. Product names as of 2026-10. -->

---

## What lasts vs. what will date

**Lasts**
- Specific + reasons > vague; iterate
- Structure prompts; show examples
- Stateless model, degrading context: new task, new chat
- Supply material; save repeated instructions; audit memory

**Will date**
- Feature names, menus, plan limits (as of 2026-10)
- Context window sizes and file limits
- Vendor differences, e.g. how many examples to include

<!-- Learn the principles; expect product names and menus to change. -->

---

## Try it yourself

1. Same request, vague vs. specific: compare
2. Add a one-line **reason** to a rule; test unnamed cases
3. Notice when a long chat drifts; restart with a recap
4. Build one project: role + task + format + sample output
5. Audit your stored memory and delete wrong entries

<!-- Each exercise makes one principle visible in your own workflow. -->

---
layout: end
---

# Key takeaways

- **Be specific**, say why, structure the prompt, and show examples
- **Manage context**: fresh chats per task, real material, saved instructions
- **Pick the right tool**: thinking mode, research, edit-in-place, agents

platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices\
developers.openai.com/api/docs/guides/prompt-engineering\
ai.google.dev/gemini-api/docs/prompting-strategies
