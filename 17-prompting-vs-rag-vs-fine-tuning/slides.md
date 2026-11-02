---
theme: ../theme
chapter: '17'
header: 'AI Fundamentals · 17 · Prompting vs RAG vs fine-tuning'
layout: cover
---

# Prompting, RAG or Fine-Tuning

**Start with the cheapest lever, measure, and escalate only for failures you can name**

---

## The problem

- Three ways to improve an LLM app, often confused
- Teams fine-tune to "teach facts", and it works poorly
- Others never move past prompting when they should
- Each lever changes **something different**
- Picking the wrong one wastes weeks

<!-- The goal of this topic is a diagnosis habit: look at what is failing, then pick the lever that changes that thing. -->

---

## Step 0: evals first

- Anthropic: success criteria + empirical tests **before** prompt work
- OpenAI: write evals before prompts, then loop
- Husain: "impossible to fine-tune effectively without an eval system"
- Outputs are non-deterministic; models change between versions
- Prerequisite: **topic 18, Evals**

> Not every failing eval is a prompting problem: sometimes just switch model.

<!-- Without a fixed test set you can't tell an improvement from noise, or compare a prompt change against a fine-tune. -->

---

## What each lever changes

| | Prompting | RAG | Fine-tuning |
|---|---|---|---|
| Changes | the request | what it can look up | **the weights** |
| Good for | most tasks | large / changing / private knowledge | format, style, narrow skills |
| Time | minutes | days | days–weeks |
| Undo | edit text | re-index | retrain |

<!-- Cost and reversibility rise from left to right. That ordering is the core of the decision guide. -->

---

## Retrieval for facts, fine-tuning for form

- Ovadia et al. (2023): RAG **consistently beat** unsupervised fine-tuning at adding knowledge
- Models struggle to learn **new facts** by fine-tuning
- Husain: fine-tune for **syntax, style, rules**; RAG for context and fresh facts
- Don't fine-tune on the wiki; retrieve from it (topic 16)

<!-- Many rephrasings of a fact helped fine-tuning somewhat, but retrieval was still more reliable and stays current. -->

---

## The decision ladder

1. **Prompt + examples**: stop if evals pass
2. **Add retrieval** when failures are missing knowledge
   - or just put small material in context
3. **Fine-tune** when failures are behaviour:
   - inconsistent results; task too complex to describe
   - niche format or language
   - need more examples than fit; shorter prompts
   - smaller, cheaper, faster model

<!-- Google's docs: foundation models work well when the task can be clearly defined in a prompt and the prompt is consistent. -->

---

## Diagnose, then choose

| Symptom | Lever |
|---|---|
| Misunderstands the task | clearer prompt, examples |
| Wrong / missing facts | retrieval or in-context facts |
| Inconsistent format or tone | examples, then fine-tune |
| Good but slow / expensive | smaller model; fine-tune or distil |
| Weak domain reasoning | better model first; specialised tuning last |

<!-- Label each failing eval case with one of these categories before deciding anything. -->

---

## Where fine-tuning earns its place

- **Honeycomb**: niche query language; tuning beat prompting with a manual
- **ReChat**: strict mixed structured/unstructured output format
- Rarely pays off for: developer tools, general assistants, early products without evals
- Methods: **SFT** (correct examples), **preference/DPO** (better vs. worse), **reinforcement** (graded outputs)

<!-- Examples from Hamel Husain's 2024 post. Method list from OpenAI's model optimization guide. -->

---

## Costs and risks of fine-tuning

- **Data**: a representative labelled set plus a validation set
- **Evals**: prove it beats the prompted baseline, and nothing regressed
- **Catastrophic forgetting**: narrow training erodes general ability
  - LoRA learns less, **forgets less** than full fine-tuning
- **Mismatch**: tuned models expect training-time conditions
- **Lock-in**: tied to a base model that will be retired

<!-- Include general tasks in your eval, not only the target task, so you notice forgetting. -->

---

## Maintenance is the hidden cost

- Fine-tuned models run only until their **base model** is retired
- Then: retrain on the new base, re-run evals
- Platform support changes too:
  - OpenAI self-serve fine-tuning closed to new orgs (**May 2026**)
  - new jobs for existing customers end (**Jan 2027**, planned)
- A prompt usually carries over with modest edits
- Self-hosting open weights is the alternative (topic 19)

<!-- As of October 2026. The specific dates will date; the lesson that weights are a sticky, maintained asset will not. -->

---

## Distillation

```
Large model + good prompt (+ RAG)
        │  run on many real inputs
        ▼
Keep outputs your evals rate as good
        │
        ▼
Fine-tune a small model ─► re-check on same evals
```

- Hinton et al. (2015): compress a big model into a cheaper one
- Only works if evals can **filter** good outputs
- Check provider terms on training with outputs

<!-- This is OpenAI's "smaller, cheaper, faster model" case. The pipeline itself is general practice. -->

---

## Combining the levers

- **Prompt + RAG**: default for knowledge-heavy assistants
- **Fine-tuned model + RAG**: weights for form, retrieval for facts
- **Distilled small model + prompt**: high-volume narrow tasks
- There is always a prompt; **evals stay the referee**
- Change **one lever at a time**

<!-- Changing several levers at once makes it impossible to tell which change helped. -->

---

## What lasts vs. what will date

**Lasts**
- Evals before levers
- Request vs. lookup vs. weights
- Retrieval for facts, fine-tuning for form
- Escalate cheap → sticky, for named failures

**Will date**
- Which providers offer fine-tuning, on which models
- Supported methods and dataset limits
- How far prompting alone gets you

<!-- Each model generation moves the point where prompting stops being enough. -->

---

## Try it yourself

1. Write 20 test inputs with pass/fail checks; score your prompt
2. Label failures: task, knowledge, format, or cost
3. Add 3 examples + clearer instructions; re-score
4. Put source text for 5 wrong facts in the prompt; re-test
5. Sketch a distillation plan: data, filter, regression checks

<!-- Exercise 4 usually shows that "the model doesn't know X" is an access problem, not a training problem. -->

---
layout: end
---

# Key takeaways

- **Measure first**: evals decide, not intuition
- **Prompt → retrieve → fine-tune**, escalating only for named failures
- **Retrieval for facts, fine-tuning for form**; weights need maintenance

developers.openai.com/api/docs/guides/model-optimization\
hamel.dev/blog/posts/fine_tuning_valuable.html\
arxiv.org/abs/2312.05934
