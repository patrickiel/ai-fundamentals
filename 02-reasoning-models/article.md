# Prompting Reasoning Models: Planners, Workhorses, and Simpler Prompts

> **Core idea in one sentence:** Reasoning models already work through problems step by step before they answer, so you get better results from short, clearly structured prompts that state the constraints and the goal, and from using these models for planning and judgment while faster models handle routine execution.

## TL;DR

- A **reasoning model** spends internal "reasoning tokens" before it answers. Those tokens cost money and take up context, but they help with planning, multi-step problems and tool use [2].
- OpenAI's guide says these models work best with **simple, direct prompts**. Don't add "think step by step"; it may not help and "can sometimes hinder" performance [1].
- Use **delimiters** (Markdown, XML tags, section titles) to separate the parts of the prompt [1].
- **Zero-shot first** is OpenAI's and DeepSeek's advice: DeepSeek found few-shot examples consistently degraded R1's results [1][6]. Anthropic says worked examples still help Claude's thinking [4]. Test on your own model.
- State **explicit constraints** and a **specific end goal** so the model knows what "done" looks like [1][2].
- OpenAI calls reasoning models **planners** and faster GPT models **workhorses**. Many systems combine them: cheap models do high-volume steps, the reasoning model plans or makes the hard call [1].
- Every major vendor now exposes a **thinking-effort control** (as of 2026-10). Treat it as a tuning knob, not your main lever for quality [2][4][5].

## What makes a reasoning model different

A standard model starts writing its answer straight away. A reasoning model first generates intermediate reasoning: it breaks the problem down, tries approaches, and checks itself, and only then writes the final answer. OpenAI describes these models as suited to "complex problem solving, coding, scientific reasoning, and multi-step agentic workflows" [2].

The idea grew out of **chain-of-thought prompting**. In 2022, Wei et al. showed that giving a model a few worked examples with written-out reasoning steps improved its results on arithmetic, commonsense and symbolic reasoning tasks [7]. Reasoning models internalise that behaviour through training. The DeepSeek-R1 paper showed that reasoning abilities "can be incentivized through pure reinforcement learning", with behaviours such as self-verification and reflection emerging during training [6].

Three practical consequences:

- **Reasoning costs tokens.** OpenAI bills reasoning tokens as output tokens, and they occupy context window space even though you don't see them. Its guide suggests reserving "at least 25,000 tokens for reasoning and outputs" when you start experimenting [2]. Gemini and Claude also bill thinking tokens and count them against output limits [4][5].
- **Reasoning takes time.** Use it where accuracy matters more than latency.
- **The line is blurring.** Many current models think adaptively, deciding per request how much to reason (as of 2026-10) [4][5].

## Prompting best practices

OpenAI's prompting guide sums up the difference: reasoning models "will provide better results on tasks with only high-level guidance", whereas GPT models need more precise instructions [3].

### 1. Keep the prompt simple and direct

The models "excel at understanding and responding to brief, clear instructions" [1]. Leave out filler and padding. Every unnecessary sentence is one more thing the model has to interpret.

### 2. Don't tell it how to think

Asking a reasoning model to "think step by step" or "explain your reasoning" is unnecessary because the reasoning already happens internally. OpenAI notes it "may not enhance performance (and can sometimes hinder it)" [1]. Anthropic makes a related point: a general instruction such as "think thoroughly" often produces better reasoning than a hand-written step-by-step plan, because "Claude's reasoning frequently exceeds what a human would prescribe" [4]. On some current Claude models, a prompt that asks the model to write its reasoning into the answer may even be declined; the docs say to rely on built-in thinking instead (as of 2026-10) [4].

### 3. Use delimiters

Markdown headings, XML tags or section titles help "clearly indicate distinct parts of the input" [1]. They remove ambiguity about what role each piece of text plays. Without them, the model has to guess whether a pasted paragraph is background, an example to imitate, or an instruction to follow.

### 4. Zero-shot first, examples if needed

OpenAI advises trying a prompt with no examples first, then adding a few if the output misses. Any examples should "align very closely" with your instructions, because discrepancies produce poor results [1]. DeepSeek went further for R1: it found the model sensitive to prompts, with few-shot prompting consistently degrading performance, and recommended describing the problem directly and specifying the output format zero-shot [6].

Anthropic's advice for Claude differs: multishot examples work with thinking, and worked examples shape how the model approaches similar problems in its own reasoning. It suggests presenting each as the problem, the method and the expected answer [4]. The safe conclusion is to start without examples, add them when the output misses, keep them consistent with the instructions, and measure whether they help on the model you use.

### 5. Be specific about constraints and the goal

OpenAI's guide gives two related tips [1]:

- **Provide specific guidelines.** If there is a limit, state it: "propose a solution with a budget under $500".
- **Be very specific about your end goal.** Give clear criteria for a successful response and encourage the model to keep reasoning until it meets them.

OpenAI's reasoning guide repeats this for agentic work: give the model "the task, constraints, and desired output format", and define what "done" means and how the model should check its result [2]. Explaining the purpose behind a constraint ("under $500, because it's a starter setup for a beginner") gives the model something to judge its intermediate ideas against. Anthropic's general guidance makes the same point about adding the reason behind instructions (topic 01).

### 6. Ask for a self-check where it helps

Anthropic suggests ending with something like "Before you finish, verify your answer against [test criteria]", which it says catches errors reliably for coding and maths. It also warns that some newer models verify on their own, and old verification instructions can then cause over-verification (as of 2026-10) [4]. Re-test inherited prompts when you change models.

## API-specific notes

Some guidance applies only when you call models through an API (all as of 2026-10):

| Point | What it means |
|---|---|
| **Developer messages** | For OpenAI reasoning models from `o1-2024-12-17` on, developer messages replace system messages for setting conversation-level rules [1]. |
| **Markdown off by default** | From the same release, OpenAI reasoning models avoid Markdown in API output. Put `Formatting re-enabled` on the first line of the developer message to turn it back on [1]. |
| **Effort controls** | OpenAI: `reasoning.effort`, from `none`/`minimal` up to `xhigh`/`max` depending on the model [2]. Anthropic: adaptive thinking plus an `effort` setting [4]. Google: `thinking_level` [5]. |
| **Output caps include thinking** | OpenAI's `max_output_tokens` and Gemini's `max_output_tokens` cap thinking plus visible output, so a low cap can end a response before any answer appears [2][5]. |

OpenAI's advice on effort: "Treat `reasoning.effort` as a tuning knob, not the primary way to recover quality" [2]. Google suggests low thinking for simple retrieval or classification, the default for moderate tasks, and the highest level for hard maths, advanced coding and multi-step planning [5].

## A reusable prompt template

A three-part structure puts these practices together:

```markdown
## Context
<Background the model needs: the audience, what the output is for,
relevant facts.>

## Examples
<Optional. Add only if a zero-shot attempt missed the style or format.
Keep them consistent with the instructions below.>

## Instructions
<The task, explicit constraints, and the goal behind them, e.g.
"Write an original post for <audience>, under 200 words, matching the
style of the examples. Success means a reader knows what to do next.">
```

Note what's missing: no "think carefully", no step-by-step plan. The structure and the stated goal do the work.

## Planners and workhorses

OpenAI's guide frames its two model families this way [1]:

- **Reasoning models ("the planners")** think longer and suit strategy, planning, and decisions based on large amounts of ambiguous information.
- **GPT models ("the workhorses")** have lower latency and cost, and suit straightforward execution.

Its example combines them in a customer-service flow: GPT-4o and GPT-4o mini triage order details and customer information and identify the issues, then pass those data points to o3-mini, which makes the final decision on whether a return is viable [1]. A related OpenAI cookbook goes the other way round: o1 turns help-center policy articles into step-by-step routines that a cheaper agent can then follow [3].

```
Cheap, fast model(s)  → gather and triage data          (high volume)
Reasoning model       → plan, decide, or check          (where errors cost most)
Cheap, fast model(s)  → carry out the routine steps     (high volume)
```

The lesson is about where to spend effort: put the careful, expensive model where mistakes are costly (turning messy material into a plan, making the judgment call, checking the result), and let fast models handle volume. Topics 24 and 26 develop this into full harness designs.

## When to use a reasoning model

OpenAI lists seven patterns it has seen in practice, noting the list isn't exhaustive [1]:

1. **Ambiguous tasks.** They infer intent from limited or scattered information and often ask clarifying questions instead of guessing.
2. **Needle in a haystack.** They pull the relevant details out of large unstructured inputs, such as a single clause buried in a long contract.
3. **Relationships across large datasets.** They reason across hundreds of pages of dense documents such as contracts, financial statements and insurance claims.
4. **Multi-step agentic planning.** The reasoning model plans and assigns steps to GPT models according to whether intelligence or speed matters more.
5. **Visual reasoning.** Interpreting detailed images, such as reading a legend on an architectural drawing and applying it elsewhere in the drawing.
6. **Code review and debugging.** Often run in the background because of latency. GPT models may still be the better choice for low-latency code writing.
7. **Evaluation and benchmarking.** Judging other models' outputs (see topic 18).

The opposite case follows: for simple, well-defined, high-volume tasks, a non-reasoning model or a low effort setting is usually the better trade-off.

## What lasts vs. what will date

- **Durable principles:**
  - Don't tell a model how to think if it already reasons. Spend your words on clear inputs, constraints and the goal.
  - Structure prompts with explicit delimiters.
  - Start without examples, add them when needed, keep them consistent with the instructions, and measure.
  - Define what success looks like, including how to check it.
  - Split work by capability: careful models for planning and judgment, fast models for volume.
- **Time-sensitive details:**
  - Model names (o-series, GPT-4o, R1, current Claude and Gemini lines) and which ones think by default.
  - Effort parameter names and values, output caps, and the "developer message" and `Formatting re-enabled` conventions (as of 2026-10).
  - Vendor-specific differences, such as whether examples help or hurt.

## Try it yourself

1. **Drop the chain-of-thought habit.** Run a prompt containing "think step by step" on a reasoning model with and without that phrase. Compare quality, length and token use.
2. **Restructure a messy prompt** into Context / Examples / Instructions and check whether it follows instructions more closely.
3. **Zero-shot, then few-shot.** Ask for a specific style with no examples. If it misses, add two short examples that match the instructions exactly, and compare.
4. **Add the goal.** Ask for a recommendation "under $500", then again with the purpose stated. Compare.
5. **Turn the effort knob.** Run the same hard and easy task at low and high effort. Note where higher effort helps and where it only adds latency.

## Key terms

- **Reasoning model**: an LLM that generates internal reasoning before its final answer, trading speed and cost for accuracy on complex tasks.
- **Reasoning (thinking) tokens**: the tokens a model spends reasoning. Usually hidden or summarised, but billed and counted against limits.
- **Chain-of-thought (CoT) prompting**: asking a model to write out intermediate steps. Useful for older models, redundant for reasoning models.
- **Delimiter**: a marker (heading, XML tag, section title) that separates the parts of a prompt.
- **Zero-shot / few-shot**: prompting with no examples vs. a small number of example outputs.
- **Reasoning effort / thinking level**: an API setting that controls how much a model reasons.
- **Developer message**: in OpenAI's API, the conversation-level instruction message that took over from the "system" message.
- **Planner vs. workhorse**: OpenAI's terms for reasoning models (planning, judgment) and faster GPT models (execution).

## Sources

1. [Reasoning best practices](https://developers.openai.com/api/docs/guides/reasoning-best-practices) (OpenAI, API docs, accessed 2026-10-14). Planners vs. workhorses, the customer-service example, the seven use cases, and the prompting advice (simple prompts, no CoT, delimiters, zero-shot first, guidelines and end goal, developer messages, `Formatting re-enabled`).
2. [Reasoning models](https://developers.openai.com/api/docs/guides/reasoning) (OpenAI, API docs, accessed 2026-10-14). Reasoning tokens, billing and context use, the 25,000-token reserve, effort values, and "tuning knob" advice.
3. OpenAI: [Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering) (API docs, accessed 2026-10-14) and [Using reasoning for routine generation](https://developers.openai.com/cookbook/examples/o1/using_reasoning_for_routine_generation) (Cookbook, archived). How prompting differs for reasoning vs. GPT models; o1 converting policy articles into routines for an agent.
4. [Prompting best practices: Thinking and reasoning](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) (Anthropic, Claude docs, accessed 2026-10-14). General instructions over prescriptive steps, multishot examples with thinking, self-checks, adaptive thinking and effort.
5. [Gemini thinking](https://ai.google.dev/gemini-api/docs/thinking) (Google, Gemini API docs, accessed 2026-10-14). `thinking_level`, thought summaries, token billing, and matching thinking level to task difficulty.
6. [DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning](https://arxiv.org/abs/2501.12948) (DeepSeek-AI, arXiv 2025-01, rev. 2026-01). Reasoning learned through reinforcement learning, emergent self-verification, and the finding that few-shot prompting degrades R1.
7. [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903) (Wei et al., Google, arXiv 2022-01). The original chain-of-thought result that reasoning models build on.
