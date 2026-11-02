# Open-Weight and Local Models: What They Are and When to Run Your Own

> **Core idea in one sentence:** Running a model yourself trades frontier capability and convenience for control over data, cost and versions; whether that trade pays off depends on memory arithmetic, license terms and an eval on your own task, not on a leaderboard.

## TL;DR

- **"Open-weight" is not "open source."** Open-weight means downloadable parameters under some license. The OSI definition also requires training code, data information and freedom to use the model for any purpose [1].
- **Parameters set the memory bill:** roughly 2 GB per billion parameters at 16-bit [3], plus a KV cache that grows with context.
- **Quantization** stores weights in fewer bits, cutting memory several-fold. Quality loss is often small at 8- and 4-bit, grows below that, and must be checked on your task [3][4].
- **Runtimes** (llama.cpp, Ollama, LM Studio, vLLM, as examples, not endorsements) load weights and expose an API, often OpenAI-compatible [4][5][6][7].
- **Local fits** privacy, offline use, version control, fine-tuning and some high-volume work. **It doesn't fit** frontier-capability needs or teams that can't absorb hardware and ops.
- **Read the license** and **evaluate on your own data**: benchmark scores shift with how the benchmark is run [2][8].

## Three kinds of model access

| | Closed / API | Open-weight | Open source (OSI definition) |
|---|---|---|---|
| **You get** | An endpoint; weights stay with the provider | Downloadable weights under the provider's license | Weights **plus** training code **plus** detailed data information |
| **Run where** | Provider's servers | Anywhere you have the hardware | Anywhere |
| **Use restrictions** | Terms of service | Whatever the license says; may include limits | Must allow use for any purpose |
| **Can you retrain from scratch?** | No | No (no data or training code) | In principle, yes |

The Open Source Initiative's **Open Source AI Definition (OSAID) 1.0** carries the four classic freedoms (use, study, modify, share for any purpose) over to AI systems. It says the "preferred form for making modifications" includes three things [1]:

- **Data information:** enough detail about the training data that a skilled person could build a substantially equivalent system.
- **Code:** "The complete source code used to train and run the system."
- **Parameters:** the weights, available under OSI-approved terms.

Most "open" models ship only the third item, often under a custom license. The Llama 3.1 Community License, for example, requires companies above 700 million monthly active users to request a separate license and binds all users to an Acceptable Use Policy [2]. A license that limits who may use a model, or for what, fails the OSAID's "use for any purpose" freedom, so by that definition the model is **open-weight, not open source**. The label matters because it decides what you may legally build.

## What "parameters" means in practice

A model's **parameters** (weights) are the numbers learned in training. A "7B" or "70B" model has roughly 7 or 70 billion of them. To run the model, all of them must sit in memory (GPU VRAM, unified memory, or system RAM).

Hugging Face's inference guide gives the rule of thumb [3]:

> Loading the weights of a model having X billion parameters requires roughly 4 × X GB of VRAM in float32 precision … roughly 2 × X GB in bfloat16/float16.

Most models are trained in bfloat16, and the guide notes that running them in float32 "won't give better inference results" [3]. So 16-bit is the uncompressed baseline. As examples, the guide lists a 70B model at about 140 GB and a 40B model at about 80 GB in bfloat16 [3]. That is why the largest open-weight models need multiple data-centre GPUs.

### Weights aren't the whole bill: the KV cache

During generation, the model caches intermediate values (keys and values) for every token so far. This **KV cache** grows **linearly with context length** and with the number of parallel requests. In Hugging Face's worked example, a ~15B-parameter model at 16,000 tokens needed about 15 GB just for the cache, roughly half the size of its weights. Architectures that share key/value heads (multi-query or grouped-query attention) shrink that dramatically, in the example to under 400 MB [3]. The takeaway: **long contexts and concurrency cost memory**, and how much depends on the model's architecture. Ollama's FAQ makes the same point operationally: serving parallel requests increases the context memory by the number of parallel requests [6].

## Quantization: fewer bits per weight

**Quantization** stores weights at lower precision, as 8-bit or 4-bit integers instead of 16-bit floats, "while trying to preserve as much accuracy as possible" [3]. The intuition from Hugging Face: for text generation you mostly need the *ranking* of likely next tokens to stay the same, not the exact values, so some precision can be dropped [3].

Rough weight memory using the rule above (weights only; add KV cache and runtime overhead):

| Parameters | 16-bit (2 bytes) | 8-bit (~1 byte) | ~4.5 bits/weight |
|---|---|---|---|
| 3B | ~6 GB | ~3 GB | ~1.7 GB |
| 8B | ~16 GB | ~8 GB | ~4.5 GB |
| 30B | ~60 GB | ~30 GB | ~17 GB |
| 70B | ~140 GB | ~70 GB | ~39 GB |

(Arithmetic from [3] and [4]; treat as back-of-envelope figures, as of 2026-10.)

Quantization affects output quality and speed:

- In Hugging Face's example, 8-bit gave essentially the same output, and 4-bit produced "almost" the same output with one small difference. The guide adds that 4-bit "can in practice often lead to different results … It is up to the user to try it out" [3].
- Quantization "trades improved memory efficiency against accuracy and in some cases inference time". In the Transformers implementation it was slightly *slower*, because weights are dequantized on the fly [3]. Other runtimes behave differently, so measure speed on your own setup.

### GGUF and the quant names you'll see

Most local tooling uses **GGUF**, a file format from the llama.cpp project that bundles the tensors with standardized metadata [4]. Files are named by quantization type. Hugging Face documents the effective **bits per weight** [4]:

| Type | Bits per weight |
|---|---|
| Q6_K | ~6.56 |
| Q5_K | 5.5 |
| Q4_K | 4.5 |
| IQ4_XS | 4.25 |
| Q3_K | ~3.44 |
| Q2_K | ~2.63 |
| IQ1_S | ~1.56 |

Pick the highest-precision file that fits your memory with room left for the KV cache. Then test whether a smaller one is good enough **for your task**. The memory savings are certain. The quality cost has to be measured.

## Running locally: what a runtime does

A runtime loads the weights, runs inference on your hardware and exposes an interface. Four widely used examples, in rough order from low-level to server-scale:

- **llama.cpp**: a C/C++ inference engine aiming for "minimal setup and state-of-the-art performance" on many backends, including NVIDIA (CUDA), AMD (HIP), Apple Silicon (Metal), Vulkan and plain CPUs. It supports 1.5- to 8-bit integer quantization, **CPU+GPU hybrid inference** for models larger than your VRAM, and an **OpenAI-compatible server** [5].
- **Ollama**: a tool for downloading and running open models with a local API and OpenAI-compatible clients [6]. Its FAQ shows practical details that matter: `ollama ps` shows whether a model is on GPU, CPU or split; models stay loaded for 5 minutes by default; the default context length is 4,096 tokens unless you raise it (as of 2026-10) [6].
- **LM Studio**: a desktop application listed by Hugging Face among the tools that run GGUF files [4].
- **vLLM**: a serving engine built for **throughput**: many concurrent users on server GPUs. It uses PagedAttention for KV-cache memory and continuous batching, supports many quantization formats (including GGUF, GPTQ/AWQ, FP8) and exposes an OpenAI-compatible API [7].

Because many runtimes speak the OpenAI-style API, application code often needs only a different base URL:

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:<port>/v1", api_key="not-needed")
reply = client.chat.completions.create(
    model="<local-model-name>",
    messages=[{"role": "user", "content": "Summarize this ticket: ..."}],
)
```

That makes comparing a local model against an API model cheap. But **"local" depends on configuration.** Ollama states, "We don't see your prompts or data when you run locally". It also offers cloud-hosted models, which you can disable (`OLLAMA_NO_CLOUD=1`) when nothing may leave the machine [6].

## When local makes sense

- **Privacy and data control.** Prompts and documents never leave hardware you control. This is often the deciding reason in regulated settings (see topic 20, Data privacy & cost).
- **Offline or air-gapped use.** Once the weights are downloaded, no network is needed.
- **Version control.** The weights you tested are the weights you run. A provider can't silently update or retire them. (General background.)
- **Fine-tuning and customization.** Weights you hold can be fine-tuned for style or format, if the license allows it (see topic 17, Prompting vs RAG vs fine-tuning).
- **Cost at scale.** For steady, high-volume, fairly simple tasks (classification, extraction, embeddings), owned hardware can cost less than per-token pricing. This is general background: do the arithmetic, including hardware, power and staff time.

## When it doesn't

- **You need frontier capability.** The strongest models are often closed, and large open-weight ones need data-centre memory (~140 GB for 70B at 16-bit [3]). A laptop-sized model will usually be weaker on hard reasoning and long agent tasks (general background; check with your own eval).
- **You can't absorb the operations burden.** Serving concurrent users is its own engineering problem: batching and KV-cache management are what engines like vLLM are built for [7]. You also own updates, monitoring, patches and capacity.
- **Your hardware is the bottleneck.** If the model spills from GPU to CPU (`ollama ps` will show the split [6]), generation can slow sharply. Long contexts and parallel users add KV-cache memory on top of the weights [3][6].
- **Defaults hide limits.** A runtime's default context window may be much smaller than the model's maximum (4,096 tokens in Ollama's default, as of 2026-10) [6]. Check it before blaming the model.

## Licenses to check

Read the actual license file, not the model card's one-word summary. Using the Llama 3.1 Community License (released 2024-07-23) as an example of what can be in there [2]:

| Question | Example clause from [2] |
|---|---|
| Who may use it? | Over 700M monthly active users: must request a license from Meta |
| For what? | Must follow the incorporated Acceptable Use Policy |
| Attribution? | Must "prominently display 'Built with Llama'" when distributing |
| Derivative naming? | Models trained or fine-tuned with Llama materials **or outputs** must have names beginning with "Llama" |
| Redistribution? | Must include a copy of the agreement and a notice file |

Also check, for any model: whether **commercial use** is allowed, whether there are **geographic or field-of-use restrictions**, whether **outputs may be used to train other models**, and whether the license is OSI-approved (for example Apache-2.0 or MIT, general background). Licenses differ between versions of the same model family, so check each release. This is not legal advice. If the product matters, have counsel read it.

## Evaluate for your task, not the leaderboard

Leaderboards are a starting filter at best. In 2023, Hugging Face found that the **same MMLU benchmark** gave a 65B model scores of **0.637, 0.488 or 0.636** depending on which evaluation implementation and prompt format was used, and that these differences "even change the ranking order of the models" [8]. Scores also stop separating models as benchmarks saturate, and benchmark questions can leak into training data (general background).

Instead, reuse the approach from **topic 18, Evals**:

1. **Take your golden dataset**: 20–100 real inputs from your task, with pass/fail checks.
2. **Run each candidate as you'll actually deploy it**: same quantization file, same runtime, same context length and same chat template.
3. **Include an API baseline**, so you know what "good enough" costs elsewhere.
4. **Measure four things:** pass rate, latency (time to first token and tokens per second), peak memory, and failure modes (read the outputs).
5. **Try one quant level lower.** If the pass rate holds, you've saved memory. If it drops, you've found your floor.

"The 8B model at Q4_K passes 88% on our extraction set vs. 94% for the API model, with no data leaving the building" is a decision you can defend. "It's #3 on a leaderboard" isn't.

## What lasts vs. what will date

- **Durable principles:** open-weight ≠ open source, so check what you get and what the license permits; memory ≈ parameters × bytes per parameter, plus a KV cache that grows with context and concurrency; quantization trades memory for some task-dependent quality; local wins on control and privacy, hosted frontier models on capability and zero ops; your own eval beats any leaderboard.
- **Time-sensitive details (as of 2026-10):** runtime defaults and features (Ollama's 4,096-token default context, keep-alive, cloud models; llama.cpp backends; vLLM formats); GGUF quant type names; license terms, which change between versions; which open-weight models are strongest (deliberately not listed).

## Try it yourself

1. **Do the memory math** for three model sizes at 16-bit, 8-bit and ~4.5 bits, and compare with your RAM/VRAM.
2. **Run one model locally.** Check whether it sits on GPU, CPU or both, and what context length it actually uses.
3. **Compare two quant levels** of one model on 20 real inputs: pass rate, tokens per second, memory.
4. **Read one license end to end** and list thresholds, use restrictions, attribution, naming and output-training rules.
5. **Point an existing script at a local endpoint** by changing only the base URL. Note what breaks.

## Key terms

- **Open-weight model**: downloadable trained parameters, under the publisher's license.
- **Open Source AI (OSAID)**: OSI's definition requiring weights, full code and sufficient data information, usable for any purpose.
- **Parameters / weights**: the learned numbers in a model. Count × precision sets memory needs.
- **Quantization**: storing weights in fewer bits to cut memory, at some risk to quality.
- **KV cache**: per-token memory kept during generation. It grows with context length and parallel requests.
- **GGUF**: a model file format from the llama.cpp ecosystem bundling tensors and metadata.
- **Runtime / inference engine**: software that loads weights and serves generations.
- **Bits per weight**: average storage per parameter for a quantization type.

## Sources

1. [The Open Source AI Definition 1.0](https://opensource.org/ai/open-source-ai-definition) (Open Source Initiative, v1.0, read 2026-10). The four freedoms and the required data information, code and parameters.
2. [Llama 3.1 Community License Agreement](https://raw.githubusercontent.com/meta-llama/llama-models/main/models/llama3_1/LICENSE) (Meta, 2024-07-23). Concrete license terms: 700M MAU clause, Acceptable Use Policy, "Built with Llama", derivative naming.
3. [Optimizing LLMs for Speed and Memory](https://huggingface.co/docs/transformers/main/en/llm_tutorial_optimization) (Hugging Face Transformers docs, read 2026-10). Memory rule of thumb, 8/4-bit quantization results and caveats, KV-cache growth, MQA/GQA.
4. [GGUF](https://huggingface.co/docs/hub/gguf) (Hugging Face Hub docs, read 2026-10). What GGUF is, quantization types with bits per weight, and tools that use it (llama.cpp, LM Studio, Ollama).
5. [llama.cpp README](https://github.com/ggml-org/llama.cpp) (ggml-org, read 2026-10). Goals, hardware backends, 1.5–8-bit quantization, CPU+GPU hybrid inference, OpenAI-compatible server.
6. [Ollama documentation and FAQ](https://docs.ollama.com/faq) (Ollama, read 2026-10). Local vs. cloud privacy statement, default context length, keep-alive, CPU/GPU placement, parallel requests.
7. [vLLM documentation](https://docs.vllm.ai/en/latest/) (vLLM project, read 2026-10). Throughput-oriented serving, PagedAttention, continuous batching, quantization formats, OpenAI-compatible API.
8. [What's going on with the Open LLM Leaderboard?](https://huggingface.co/blog/open-llm-leaderboard-mmlu) (Hugging Face, 2023-06-23). The same benchmark gives different scores and rankings under different evaluation implementations.
