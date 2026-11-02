---
theme: ../theme
chapter: '19'
header: 'AI Fundamentals · 19 · Open-weight & local models'
layout: cover
---

# Open-Weight & Local Models

**Trade frontier capability and convenience for control over data, cost and versions**

---

## The problem

- "Open" models are everywhere, but **"open" means different things**
- Running one yourself raises practical questions:
  - Will it **fit** on my hardware?
  - How much **quality** do I lose by shrinking it?
  - What does the **license** let me do?
  - Is it **good enough for my task**?
- Leaderboards don't answer the last one

<!-- This topic gives the arithmetic and checklists to answer these yourself. -->

---

## Three kinds of model access

| | Closed / API | Open-weight | Open source (OSAID) |
|---|---|---|---|
| You get | An endpoint | Weights | Weights **+ code + data info** |
| Use limits | Terms of service | Whatever the license says | Any purpose |
| Retrain from scratch | No | No | In principle |

- OSI's definition: freedom to **use, study, modify, share**
- A license limiting who or what → **open-weight, not open source**

<!-- Example: the Llama 3.1 license has a 700M-MAU threshold and an Acceptable Use Policy, so it fails "use for any purpose". -->

---

## Parameters = the memory bill

> X billion parameters ≈ **2 × X GB** at 16-bit (4 × X at fp32)

- Most models are trained in bfloat16; fp32 adds nothing at inference
- 70B at 16-bit ≈ **140 GB**: multi-GPU territory
- Plus the **KV cache**: grows with **context length** and **parallel requests**
- Example: ~15B model at 16k tokens → ~15 GB of cache (architecture-dependent)

<!-- Grouped- or multi-query attention can shrink the KV cache by an order of magnitude or more. -->

---

## Quantization: fewer bits per weight

| Params | 16-bit | 8-bit | ~4.5 bits |
|---|---|---|---|
| 8B | ~16 GB | ~8 GB | ~4.5 GB |
| 30B | ~60 GB | ~30 GB | ~17 GB |
| 70B | ~140 GB | ~70 GB | ~39 GB |

- Weights only; back-of-envelope, as of 2026-10
- 8-bit: usually near-identical; 4-bit: "can often lead to different results"
- Lower still: more degradation; **measure on your task**

<!-- Generation mostly needs the ranking of likely next tokens preserved, which is why moderate quantization works. -->

---
zoom: 0.9
---

## GGUF and quant names

| Type | Bits/weight |
|---|---|
| Q6_K | ~6.56 |
| Q5_K | 5.5 |
| Q4_K | 4.5 |
| Q3_K | ~3.44 |
| Q2_K | ~2.63 |

- GGUF = tensors + metadata in one file (llama.cpp ecosystem)
- Pick the **highest precision that fits** with KV-cache headroom
- Then test whether one level lower is good enough

<!-- New quant types appear regularly; the bits-per-weight idea is what lasts. -->

---

## Runtimes (examples, not endorsements)

| Runtime | Shape |
|---|---|
| **llama.cpp** | C/C++ engine; CPU, CUDA, Metal, Vulkan…; CPU+GPU hybrid |
| **Ollama** | Download-and-run tool with local API |
| **LM Studio** | Desktop app for GGUF models |
| **vLLM** | Server engine for **throughput**: PagedAttention, batching |

- Many expose an **OpenAI-compatible API**: swap the base URL
- "Local" depends on config: Ollama also offers **cloud models**

<!-- Ollama: "We don't see your prompts or data when you run locally"; cloud features can be disabled. -->

---

## When local makes sense

- **Privacy**: data never leaves your hardware (topic 20)
- **Offline / air-gapped** use
- **Version control**: the weights you tested are the ones you run
- **Fine-tuning**, if the license allows (topic 17)
- **Cost at scale** for steady, simple, high-volume tasks: do the full arithmetic

<!-- Include hardware, power and staff time when comparing with per-token pricing. -->

---

## When it doesn't

- You need **frontier capability**: strongest models are often closed or huge
- You can't absorb **ops**: serving, batching, updates, monitoring, patches
- **Hardware** bottleneck: spilling from GPU to CPU slows sharply
- **Defaults hide limits**: e.g. Ollama's default context 4,096 tokens (as of 2026-10)

<!-- Check `ollama ps` (or your runtime's equivalent) to see where a model is actually loaded. -->

---

## Licenses: what to check

| Question | Llama 3.1 example |
|---|---|
| Who may use it? | >700M MAU must ask Meta |
| For what? | Acceptable Use Policy applies |
| Attribution? | "Built with Llama" |
| Derivative names? | Must start with "Llama", even if trained on **outputs** |

- Also: commercial use, geography, training on outputs, OSI-approved?
- Terms change **between versions**; not legal advice

<!-- Read the license file itself, not the one-word summary on a model card. -->

---

## Evaluate for your task, not a leaderboard

- Same MMLU benchmark, one 65B model: **0.637 / 0.488 / 0.636** depending on the eval implementation
- Rankings changed too; benchmarks also saturate and leak
- Instead (topic 18, Evals):
  1. Your golden set, 20–100 real inputs
  2. Same quant, runtime, context, chat template as production
  3. Include an **API baseline**
  4. Measure pass rate, latency, memory; read failures

<!-- Try one quant level lower: if the pass rate holds, you have saved memory. -->

---

## What lasts vs. what will date

**Lasts**
- Open-weight ≠ open source; read the license
- Memory ≈ params × bytes/param + KV cache
- Quantization trades memory for task-dependent quality
- Your own eval beats any leaderboard

**Will date (as of 2026-10)**
- Runtime defaults and features; GGUF type names
- License terms per model version
- Which open-weight models are strongest (not listed)

<!-- The arithmetic and the checklists transfer to whatever models exist next year. -->

---

## Try it yourself

1. Memory math for three model sizes at 16, 8 and ~4.5 bits
2. Run one model locally; check GPU/CPU placement and context length
3. Compare two quant levels on 20 real inputs
4. Read one license end to end; list every restriction
5. Point an existing script at a local endpoint via base URL

<!-- Exercise 5 quickly shows which features (tool calls, JSON mode) differ between providers and runtimes. -->

---
layout: end
---

# Key takeaways

- **Open-weight ≠ open source**: check weights, code, data and license terms
- **Do the memory math, then measure** what quantization costs on your task
- **Go local for control and privacy**; stay hosted for frontier capability and zero ops

opensource.org/ai/open-source-ai-definition · huggingface.co/docs/transformers/main/en/llm_tutorial_optimization · huggingface.co/blog/open-llm-leaderboard-mmlu
