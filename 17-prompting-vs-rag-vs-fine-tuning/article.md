# Prompting, RAG or Fine-Tuning: Choosing the Right Lever

> **Core idea in one sentence:** Prompting changes what you tell the model, retrieval changes what it can look up, and fine-tuning changes the model itself, so start with the cheapest lever, measure with evals, and move to a heavier one only for a problem the lighter one demonstrably can't fix.

## TL;DR

- **Evals come first.** Without a way to measure success you can't tell whether any lever helped. OpenAI's and Anthropic's docs and experienced practitioners put this step before prompting, never mind fine-tuning [1][3][5].
- **Prompting** (instructions, examples, context) is the fastest, cheapest and most reversible lever. Push it hard before anything else [1][5].
- **Retrieval (RAG)** is for **knowledge**: material that is large, changes often, or is private. Research comparing the two found retrieval beat fine-tuning at adding facts, including new ones [6].
- **Fine-tuning** is for **behaviour**: a consistent format or style, a narrow specialised skill, or a smaller, cheaper model once prompting has topped out [1][4][5].
- Fine-tuning has real costs: training data, eval infrastructure, retraining when the base model is retired, and possible loss of general ability (**catastrophic forgetting**) [2][7].
- The levers **combine**. A common pattern is a prompt, plus retrieved facts, plus a model fine-tuned (or distilled) for the task's format.

## Before any lever: define "better"

The guides agree on the starting point. Anthropic's prompt engineering docs assume you already have clear success criteria, a way to test against them empirically, and a first-draft prompt; if not, build those first [3]. OpenAI describes optimisation as a loop of evals, prompting and fine-tuning, and suggests writing evals before prompts, much like behaviour-driven development [1]. Hamel Husain is blunter: "It's impossible to fine-tune effectively without an eval system" [5].

The reason is practical. Model output is non-deterministic, and behaviour changes between model versions [1]. Without a fixed test set you can't tell an improvement from noise, can't compare a prompt change with a fine-tune, and can't tell whether a new model release broke anything. **Topic 18 (Evals)** is the prerequisite for everything below.

Anthropic adds a reminder that not every failing eval is a prompting problem. Latency and cost, for example, are sometimes easier to fix by choosing a different model [3]. "Switch model" is a lever too, and often the cheapest one.

## What each lever actually changes

| | Prompting | Retrieval (RAG) | Fine-tuning |
|---|---|---|---|
| **Changes** | Instructions, examples, context in the request | What knowledge the model can see at run time | The model's weights, so its default behaviour |
| **Good for** | Most tasks; clarifying what you want | Large, changing or private knowledge; citations | Consistent format/style, narrow skills, smaller models |
| **Time to try** | Minutes | Days (pipeline, index) | Days to weeks (data, training, evaluation) |
| **Reversible?** | Edit the text | Re-index | Retrain or roll back to the base model |
| **Main cost** | Tokens per request | Pipeline and index upkeep | Data, training runs, re-training on new base models |

A useful shorthand from this comparison: **retrieval for facts, fine-tuning for form.** Husain describes fine-tuning as best for teaching a model syntax, style and rules, and RAG as the way to supply context and up-to-date facts [5]. A 2023 study by Ovadia et al. tested this directly. Unsupervised fine-tuning on new material gave modest gains, while RAG consistently did better, both for knowledge the model had partly seen and for entirely new knowledge. The authors found models struggle to learn new facts through fine-tuning, though exposing them to many rephrasings of the same fact helped somewhat [6].

That is the most common mistake this topic guards against: fine-tuning on your company wiki in the hope that the model will "know" it. Retrieval does that job more reliably, keeps the knowledge current, and lets the model cite its source (topic 16).

## A decision guide

### Step 1: Prompt, with examples

Write clear instructions, show a few input/output examples, and put the needed context in the prompt. Google's tuning docs say foundation models work well when the task "can be clearly and concisely defined in a prompt" and the prompt consistently produces the expected output [4]. If that is true for you, stop here. Husain recommends doing as much prompt engineering as possible before fine-tuning, partly because it stress-tests your evals, and stopping if prompting works: "I'm a big believer in using the simplest approach to solving a problem" [5].

### Step 2: Add retrieval when the problem is knowledge

Look at the failures your eval surfaces. If the model is wrong because it **doesn't have the information**, such as internal policies, recent events or customer records, the fix is access, not training. Add retrieval if the material is too large to include, changes often, or must be filtered per user. If it is small enough, just include it in the prompt (topic 16 covers when that works).

### Step 3: Consider fine-tuning when the problem is behaviour

The providers' docs give converging reasons to fine-tune:

- **Prompts don't produce consistent enough results**, or the task is too complicated to describe in a prompt [4].
- **The behaviour is niche** and departs from general patterns, such as a private query language or an unusual output format [4][5].
- **You need more examples than fit in a prompt**, or want shorter prompts to save tokens and latency at scale [1].
- **You want a smaller, cheaper, faster model** to handle a narrow task that a large model handles well but expensively [1].
- **You don't want to send proprietary examples with every request** [1].

Husain's examples are typical. Honeycomb needed a model to write queries in its niche query language and found fine-tuning taught the syntax and rules better than prompting with a manual and examples. ReChat needed a very specific response format mixing structured and unstructured data, and fine-tuning was what made it work reliably [5]. He also names where fine-tuning rarely pays off: developer tools (frontier models already train heavily on code), general-purpose assistants, and early products without a domain-specific eval set [5].

### Diagnose before you choose

| Symptom in your evals | Likely lever |
|---|---|
| Misunderstands the task | Clearer prompt, examples |
| Wrong or missing facts | Retrieval (or put the facts in context) |
| Facts are right but go stale | Retrieval with a refresh plan |
| Right content, inconsistent format or tone | Better examples first, then fine-tuning |
| Good quality, too slow or too expensive | Smaller model; then fine-tune or distil into it |
| Weak at complex domain reasoning | Better model or reasoning settings; specialised fine-tuning as a last resort |

## The types of fine-tuning, briefly

OpenAI's docs distinguish several methods by the kind of training signal [1]:

- **Supervised fine-tuning (SFT)**: train on examples of correct responses. Suited to classification, specific output formats and fixing instruction-following failures.
- **Preference fine-tuning (e.g. DPO)**: train on pairs of a better and a worse response. Suited to tone, style and what a summary should focus on.
- **Reinforcement fine-tuning**: a grader scores the model's outputs and higher-scoring behaviour is reinforced. Suited to complex domain reasoning, and only workable when experts agree on what a good answer is.

There is also a choice of **how much of the model** to change. Full fine-tuning updates all weights. Parameter-efficient methods such as **LoRA** train small low-rank additions and leave the original weights frozen [7].

## Costs and risks of fine-tuning

**Data.** You need a labelled dataset that represents the task well, plus a separate validation set [4]. Getting good examples is usually the slowest part of the job.

**Evaluation infrastructure.** You must be able to show that the tuned model is better than the prompted baseline on your eval, and that it hasn't got worse elsewhere [1][5].

**Catastrophic forgetting.** Training hard on one domain can erode abilities outside it. Biderman et al. (2024) compared LoRA with full fine-tuning on code and maths. LoRA learned the target domain substantially less well, but also **forgot less** of the base model's other abilities [7]. There is a trade-off between how much you change and how much you lose, so your eval should include general tasks, not only the target task.

**Train/inference mismatch.** Google warns, for instance, that combining a tuned Gemini model with constrained structured output at inference time can lower quality, because inference no longer matches how the model was tuned [4]. The general point: a tuned model expects the conditions it was trained under.

**Maintenance and lock-in.** A fine-tune is tied to its base model. OpenAI's deprecation policy, for example, keeps fine-tuned models running only until their base model is retired [2]. When the provider moves on, you retrain on the new base and re-run your evals. Provider support can also change wholesale. As of October 2026, OpenAI is winding down self-serve fine-tuning: it closed to new organisations in May 2026 and plans to stop new training jobs for existing customers in January 2027 [1][2]. A prompt, by contrast, usually carries over to a new model with modest edits. Topic 19 (Open-weight & local models) covers the alternative of tuning a model you host yourself.

## Distillation: big model teaches small model

**Distillation** was introduced by Hinton, Vinyals and Dean (2015) as a way to compress the knowledge of a large model, or an ensemble, into a single smaller model that is cheaper to deploy [8]. The modern, practical version for applications is simple (general background):

1. Get a large model working well on your task with prompting (and retrieval if needed).
2. Run it on many realistic inputs and keep the outputs your evals rate as good.
3. Fine-tune a smaller model on those input/output pairs.
4. Check the small model against the same evals.

This is the "smaller, cheaper, faster model" case from OpenAI's list [1]. It only works if your evals can tell good outputs from bad ones, which is another reason they come first. Also check your provider's terms, which may restrict using outputs to train other models.

## Combining the levers

The levers are not exclusive, and they layer naturally:

- **Prompt + retrieval** is the default for knowledge-heavy assistants.
- **Fine-tuned model + retrieval** fits when you need both a specialised format and current facts: the weights handle form, retrieval supplies the facts.
- **Distilled small model + prompt** fits high-volume, narrow tasks where cost and latency dominate.

In every combination there is still a prompt, and the evals stay the referee. Change one lever at a time and re-run them, so you know which change did what.

## What lasts vs. what will date

- **Durable principles:**
  - Measure first: an eval set comes before any lever.
  - Prompting changes the request, retrieval changes what the model can see, fine-tuning changes the model.
  - Retrieval for facts, fine-tuning for form.
  - Escalate from cheap and reversible to expensive and sticky, and only for failures you can name.
  - Fine-tuning creates an asset tied to a base model, which needs maintenance and can forget general skills.
  - Distillation turns a working large-model pipeline into a cheaper small model, if your evals can filter its outputs.
- **Time-sensitive details:**
  - Which providers offer fine-tuning and on which models (OpenAI's self-serve wind-down, announced May 2026).
  - Specific methods each platform supports (SFT, DPO, reinforcement fine-tuning) and their dataset limits.
  - How far prompting alone gets you, which shifts with each model generation.

## Try it yourself

1. **Write the eval first.** For a task you care about, write 20 test inputs with a pass/fail check for each. Run your current prompt and record the score.
2. **Classify the failures.** For each failing case, label it as task misunderstanding, missing knowledge, format/style, or cost/latency. Use the diagnosis table to pick a lever for each group.
3. **Max out the prompt.** Add three good examples and clearer instructions. Re-run the eval and see how many "format" failures disappear without any training.
4. **Facts test.** Pick five facts the model gets wrong. Put the source text in the prompt and re-test. If they now pass, you needed access, not fine-tuning.
5. **Plan a distillation (on paper).** Estimate how many good outputs from a large model you'd need, how you'd filter them with your eval, and what you'd check to confirm the small model hasn't lost general ability.

## Key terms

- **Prompting**: steering a model through instructions, examples and context in the request, without changing the model.
- **Retrieval-augmented generation (RAG)**: fetching relevant documents at run time and adding them to the prompt.
- **Fine-tuning**: further training a pre-trained model on task-specific examples, changing its weights.
- **Supervised fine-tuning (SFT)**: fine-tuning on examples of correct input/output pairs.
- **LoRA**: a parameter-efficient method that trains small low-rank additions to selected weights instead of all weights.
- **Catastrophic forgetting**: loss of a model's general abilities after training on a narrow domain.
- **Distillation**: training a smaller model to reproduce a larger model's behaviour, typically from the larger model's outputs.
- **Eval**: a repeatable test set and scoring method used to decide whether a change helped.

## Sources

1. [Model optimization](https://developers.openai.com/api/docs/guides/model-optimization) (OpenAI docs, accessed 2026-10). The evals → prompt → fine-tune loop, fine-tuning methods and when to use each, stated benefits of fine-tuning, and the notice that the fine-tuning platform is winding down.
2. [Deprecations](https://developers.openai.com/api/docs/deprecations) (OpenAI docs, accessed 2026-10). Timeline of the self-serve fine-tuning wind-down (announced 2026-05-07) and the rule that fine-tuned models follow their base model's retirement.
3. [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) (Anthropic docs, accessed 2026-10). Success criteria and empirical tests before prompting; not every failing eval is a prompting problem.
4. [About supervised fine-tuning for Gemini models](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/gemini-tuning) (Google Cloud docs, accessed 2026-10). When prompting suffices vs. when to tune, typical tuning use cases, dataset limits, and the structured-output mismatch warning.
5. [Is Fine-Tuning Still Valuable?](https://hamel.dev/blog/posts/fine_tuning_valuable.html) (Hamel Husain, 2024-03-27). Evals as a precondition, prompting first, fine-tuning for syntax/style/rules, real examples, and where fine-tuning rarely helps.
6. [Fine-Tuning or Retrieval? Comparing Knowledge Injection in LLMs](https://arxiv.org/abs/2312.05934) (Ovadia et al., arXiv 2023-12, rev. 2024-01). RAG outperforms unsupervised fine-tuning for adding knowledge, including new facts.
7. [LoRA Learns Less and Forgets Less](https://arxiv.org/abs/2405.09673) (Biderman et al., TMLR 2024). The trade-off between learning a target domain and forgetting general ability; LoRA vs. full fine-tuning.
8. [Distilling the Knowledge in a Neural Network](https://arxiv.org/abs/1503.02531) (Hinton, Vinyals & Dean, arXiv 2015-03). The original distillation idea: compress a large model's knowledge into a smaller, cheaper model.
