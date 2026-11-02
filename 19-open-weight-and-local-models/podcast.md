<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 19: Open-weight and local models

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter nineteen: open-weight and local models. What they are, and when to run your own.

MAYA: Last time, in chapter eighteen, we did evals. Read real outputs, build a small golden set, check your graders against a human. And I have a feeling that's coming back today.

BEN: [laughs] It is, right at the end. Here's the one sentence for today. Running a model yourself trades frontier capability and convenience for control over data, cost and versions. And whether that trade pays off depends on memory arithmetic, license terms, and an eval on your own task. Not on a leaderboard.

MAYA: Memory, license, eval. Okay. But let me start with a confession. I use "open-weight" and "open source" as if they mean the same thing.

BEN: Most people do. And they don't. The chapter describes three kinds of model access. First, closed, or {API|A P I}. You get an endpoint, and the weights stay with the provider. It runs on their servers, under their terms of service, and you can't retrain it from scratch.

MAYA: Second?

BEN: Open-weight. You get downloadable weights under the provider's license. You can run them anywhere you have the hardware. But the use restrictions are whatever the license says, and you still can't retrain from scratch, because you don't have the data or the training code.

MAYA: And open source is more than that?

BEN: By the Open Source Initiative's definition, yes. Their Open Source AI Definition, version one point oh, carries the four classic freedoms over to AI systems. Use, study, modify and share, for any purpose.

MAYA: And what does that require in practice?

BEN: It says the "preferred form for making modifications" includes three things. Data information: enough detail about the training data that a skilled person could build a substantially equivalent system. Code: quote, "The complete source code used to train and run the system." And parameters: the weights, available under {OSI|O S I}-approved terms.

MAYA: [thoughtful] So weights, plus code, plus data information. And most "open" models ship...

BEN: Only the third item. Often under a custom license. The Llama three point one Community License, for example, requires companies above seven hundred million monthly active users to request a separate license. And it binds all users to an Acceptable Use Policy.

MAYA: But who cares about the label, if I can download it?

BEN: Here's why it matters. A license that limits who may use a model, or for what, fails the "use for any purpose" freedom. So by that definition, the model is open-weight, not open source. And the label matters because it decides what you may legally build.

MAYA: Okay. Next: parameters. I see "seven B" and "seventy B" everywhere.

BEN: Parameters, or weights, are the numbers learned in training. A seven B or seventy B model has roughly seven or seventy billion of them. And to run the model, all of them must sit in memory. {GPU|G P U} memory, unified memory, or system {RAM|ram}.

MAYA: How much memory?

BEN: Hugging Face's inference guide gives a rule of thumb. Roughly four gigabytes per billion parameters in thirty-two-bit floats, and roughly two gigabytes per billion in sixteen-bit. And most models are trained in sixteen-bit, and the guide notes that running them in thirty-two-bit "won't give better inference results".

MAYA: So sixteen-bit is the starting point. Two gigabytes per billion.

BEN: That's the uncompressed baseline. As examples, the guide lists a seventy B model at about a hundred and forty gigabytes, and a forty B model at about eighty. That's why the largest open-weight models need multiple data-centre {GPUs|G P Us}.

MAYA: [laughs] Not my laptop, then.

BEN: Not at sixteen-bit. And the weights aren't the whole bill. During generation, the model caches intermediate values, keys and values, for every token so far. That's the {KV cache|K V cache}. It grows linearly with context length, and with the number of parallel requests.

MAYA: How big does it get?

BEN: In Hugging Face's worked example, a model of about fifteen billion parameters, at sixteen thousand tokens, needed about fifteen gigabytes just for the cache. Roughly half the size of its weights.

MAYA: Wow. Just for remembering the conversation.

BEN: But architectures that share key and value heads, called multi-query or grouped-query attention, shrink that dramatically. In the example, to under four hundred megabytes. So the takeaway: long contexts and concurrency cost memory, and how much depends on the model's architecture. Ollama's {FAQ|F A Q} makes the same point: serving parallel requests multiplies the context memory by the number of parallel requests.

MAYA: Okay. So how does anyone run these things on normal hardware?

BEN: Quantization. You store the weights at lower precision, as eight-bit or four-bit integers instead of sixteen-bit floats, quote, "while trying to preserve as much accuracy as possible".

MAYA: [curious] But doesn't throwing away precision make the model dumber?

BEN: Hugging Face gives the intuition. For text generation, you mostly need the ranking of likely next tokens to stay the same, not the exact values. So some precision can be dropped. And the chapter has a back-of-envelope table, weights only. An eight B model is about sixteen gigabytes at sixteen-bit, eight at eight-bit, and about four and a half at roughly four and a half bits per weight.

MAYA: And a big one?

BEN: Seventy B goes from about a hundred and forty, to seventy, to about thirty-nine gigabytes. A three B model drops to under two. And remember, add the {KV cache|K V cache} and runtime overhead on top.

MAYA: So, does quality hold up?

BEN: In Hugging Face's example, eight-bit gave essentially the same output. Four-bit produced "almost" the same output, with one small difference. And the guide adds that four-bit "can in practice often lead to different results", and that it's up to the user to try it out.

MAYA: And speed? Smaller should be faster, right?

BEN: Not necessarily, and that's a good misconception to clear up. The guide says quantization "trades improved memory efficiency against accuracy and in some cases inference time". In the Transformers implementation it was slightly slower, because weights are dequantized on the fly. Other runtimes behave differently. So measure speed on your own setup.

MAYA: Okay. When I go to download a model, I see these cryptic file names. Q four K, and things like that.

BEN: Most local tooling uses {GGUF|G G U F}, a file format from the {llama.cpp|llama C P P} project, that bundles the tensors with standardised metadata. Files are named by quantization type, and Hugging Face documents the effective bits per weight.

MAYA: Give me a few.

BEN: {Q6_K|Q six K} is about six point six bits. {Q5_K|Q five K} is five point five. {Q4_K|Q four K} is four point five. {IQ4_XS|I Q four X S} is four point two five. {Q3_K|Q three K} is about three point four. {Q2_K|Q two K} is about two point six. And {IQ1_S|I Q one S} is about one point six.

MAYA: How do I pick?

BEN: Pick the highest-precision file that fits your memory, with room left for the {KV cache|K V cache}. Then test whether a smaller one is good enough for your task. The memory savings are certain. The quality cost has to be measured.

MAYA: [thoughtful] So, sixteen-bit is about two gigabytes per billion parameters. Quantization cuts that several-fold. Plus the cache, which grows with context and users. And quality loss is small at eight and four bits, but I have to test it.

BEN: Good summary, with one nuance. Often small, not always small. And the loss grows below four bits.

MAYA: Now, how do I actually run one?

BEN: With a runtime. It loads the weights, runs inference on your hardware, and exposes an interface. The chapter gives four widely used examples, and it's clear they're examples, not endorsements. First, {llama.cpp|llama C P P}. A C and C plus plus inference engine aiming for, quote, "minimal setup and state-of-the-art performance" on many backends.

MAYA: Which backends?

BEN: {NVIDIA|Nvidia}, {AMD|A M D}, Apple Silicon, Vulkan, and plain {CPUs|C P Us}. It supports one-and-a-half to eight-bit integer quantization. It does hybrid {CPU|C P U} and {GPU|G P U} inference, for models larger than your graphics memory. And it has an OpenAI-compatible server.

MAYA: Second?

BEN: Ollama. A tool for downloading and running open models, with a local {API|A P I} and OpenAI-compatible clients. And its {FAQ|F A Q} has practical details that matter. {ollama ps|ollama P S} shows whether a model is on {GPU|G P U}, {CPU|C P U}, or split. Models stay loaded for five minutes by default. And, as of October 2026, the default context length is four thousand and ninety-six tokens, unless you raise it.

MAYA: Wait, only four thousand by default? Even if the model supports way more?

BEN: Exactly. Hold that thought, it comes back. Third, LM Studio, a desktop application that Hugging Face lists among the tools that run {GGUF|G G U F} files. And fourth, {vLLM|V L L M}. A serving engine built for throughput: many concurrent users on server {GPUs|G P Us}.

MAYA: How is that different?

BEN: It uses something called PagedAttention for {KV cache|K V cache} memory, and continuous batching. It supports many quantization formats, including {GGUF|G G U F}. And it also exposes an OpenAI-compatible {API|A P I}.

MAYA: Everyone speaks OpenAI's language, then.

BEN: Many do. And that's useful. Application code often needs only a different base {URL|U R L}. The chapter shows a short Python snippet. You create the usual OpenAI client, but point the base {URL|U R L} at localhost, put any placeholder as the {API|A P I} key, and call chat completions with your local model's name.

MAYA: So comparing a local model with an {API|A P I} model is cheap.

BEN: Very cheap. But there's a catch: "local" depends on configuration. Ollama states, quote, "We don't see your prompts or data when you run locally". But it also offers cloud-hosted models. You can disable those, with an environment variable, {OLLAMA_NO_CLOUD=1|Ollama no cloud, set to one}, when nothing may leave the machine.

MAYA: Ooh. So "I'm using Ollama" doesn't automatically mean "my data stays here".

BEN: Right. Check the setup. So, when does local make sense? First, privacy and data control. Prompts and documents never leave hardware you control. That's often the deciding reason in regulated settings, and chapter twenty, on data privacy and cost, goes further.

MAYA: Second?

BEN: Offline or air-gapped use. Once the weights are downloaded, no network is needed. Third, version control. The weights you tested are the weights you run. A provider can't silently update or retire them.

MAYA: And fine-tuning, from chapter seventeen.

BEN: Yes. Weights you hold can be fine-tuned for style or format, if the license allows it. And fifth, cost at scale. For steady, high-volume, fairly simple tasks, like classification, extraction or embeddings, owned hardware can cost less than per-token pricing. But the chapter flags that as general background. Do the arithmetic, including hardware, power and staff time.

MAYA: And when doesn't it make sense?

BEN: When you need frontier capability. The strongest models are often closed, and large open-weight ones need data-centre memory. A laptop-sized model will usually be weaker on hard reasoning and long agent tasks. Again, check that with your own eval.

MAYA: Okay. What else?

BEN: When you can't absorb the operations burden. Serving concurrent users is its own engineering problem. Batching and {KV cache|K V cache} management are what engines like {vLLM|V L L M} are built for. And you own updates, monitoring, patches and capacity.

MAYA: [sighs] Right. Someone has to be on call for it.

BEN: Third, when your hardware is the bottleneck. If the model spills from {GPU|G P U} to {CPU|C P U}, and {ollama ps|ollama P S} will show the split, generation can slow sharply. And long contexts and parallel users add cache memory on top of the weights.

MAYA: And fourth is my thought I was holding.

BEN: [laughs] Yes. Defaults hide limits. A runtime's default context window may be much smaller than the model's maximum. Like that four-thousand-token Ollama default. So check it before blaming the model.

MAYA: Okay. Licenses. What do I actually look for?

BEN: Read the actual license file, not the model card's one-word summary. The chapter uses the Llama three point one Community License, from July 2024, as an example of what can be in there. Who may use it? Above seven hundred million monthly active users, you must request a license from Meta. For what? You must follow the Acceptable Use Policy.

MAYA: Anything surprising?

BEN: A few things. Attribution: when distributing, you must "prominently display 'Built with Llama'". Derivative naming: models trained or fine-tuned with Llama materials, or outputs, must have names beginning with "Llama". And redistribution: you must include a copy of the agreement and a notice file.

MAYA: [laughs] So if I fine-tune it, my model is called Llama something.

BEN: That's what the clause says for derived models. And for any model, also check whether commercial use is allowed, whether there are geographic or field-of-use restrictions, whether outputs may be used to train other models, and whether the license is {OSI|O S I}-approved. Apache two point oh or {MIT|M I T}, for example.

MAYA: And I check this once per model family?

BEN: No, per release. Licenses differ between versions of the same model family. And the chapter is clear: this isn't legal advice. If the product matters, have counsel read it.

MAYA: Okay. Last part. Leaderboards. Can't I just pick whatever is top of the chart?

BEN: Leaderboards are a starting filter at best. In 2023, Hugging Face found that the same {MMLU|M M L U} benchmark gave a sixty-five B model scores of zero point six three seven, zero point four eight eight, or zero point six three six. Depending on which evaluation implementation and prompt format was used.

MAYA: [curious] Same model, same benchmark?

BEN: Same model, same benchmark. And they found these differences, quote, "even change the ranking order of the models". On top of that, scores stop separating models as benchmarks saturate. And benchmark questions can leak into training data.

MAYA: So, chapter eighteen.

BEN: Exactly. Reuse the evals approach, in five steps. One, take your golden dataset, twenty to a hundred real inputs from your task, with pass or fail checks. Two, run each candidate the way you'll actually deploy it. Same quantization file, same runtime, same context length, and same chat template.

MAYA: Three?

BEN: Include an {API|A P I} baseline, so you know what "good enough" costs elsewhere. Four, measure four things. Pass rate. Latency, meaning time to first token and tokens per second. Peak memory. And failure modes, which means reading the outputs. And five, try one quant level lower. If the pass rate holds, you've saved memory. If it drops, you've found your floor.

MAYA: And then I end up with a sentence I can defend.

BEN: The chapter gives one. "The eight B model at {Q4_K|Q four K} passes eighty-eight percent on our extraction set, versus ninety-four percent for the {API|A P I} model, with no data leaving the building." That's a decision you can defend. "It's number three on a leaderboard" isn't.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Open-weight isn't open source, so check what you get and what the license permits. Memory is roughly parameters times bytes per parameter, plus a {KV cache|K V cache} that grows with context and concurrency. Quantization trades memory for some task-dependent quality. Local wins on control and privacy, hosted frontier models on capability and zero ops. And your own eval beats any leaderboard.

MAYA: And what will date?

BEN: Runtime defaults and features. Ollama's default context, keep-alive and cloud models. The {llama.cpp|llama C P P} backends and the {vLLM|V L L M} formats. The {GGUF|G G U F} quant type names. License terms, which change between versions. And which open-weight models are strongest, which the chapter deliberately doesn't list.

MAYA: Let me try to sum up, and you correct me. Open-weight means I can download the weights under some license. Open source, by the {OSI|O S I} definition, also needs the training code and data information, and use for any purpose. Memory is about two gigabytes per billion parameters at sixteen-bit, plus a cache that grows with context and users.

BEN: Good so far.

MAYA: Quantization shrinks that, and quality usually holds at eight and four bits, but I test it. Runtimes like Ollama or {llama.cpp|llama C P P} often speak the OpenAI {API|A P I}, so I just change the base {URL|U R L}. Local wins on privacy, offline use, versions and fine-tuning. It loses on frontier capability and ops. And I read the license, and pick with my own eval, not a leaderboard.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Roughly how much memory do the weights of an eight B model need at sixteen-bit, and at about four and a half bits per weight? And what comes on top?

PAUSE: 5

MAYA: About sixteen gigabytes at sixteen-bit, and about four and a half at four and a half bits. On top comes the {KV cache|K V cache}, which grows with context length and parallel requests, plus runtime overhead.

BEN: Perfect. Question two. A model's license requires companies above a certain user count to request a separate license, and binds everyone to a use policy. Is it open source?

PAUSE: 5

MAYA: Not by the {OSI|O S I} definition, because it limits who may use it and for what. That fails "use for any purpose". So it's open-weight, not open source.

BEN: Exactly. And last one. You're choosing between two quant levels of a local model. How do you decide?

PAUSE: 5

MAYA: Run both on my own golden set, deployed the way I'll actually use them, with an {API|A P I} baseline. Compare pass rate, latency, peak memory and failure modes. If the smaller one holds its pass rate, I save the memory.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the memory math first, for three model sizes at three precisions, and comparing it with my own machine.

BEN: Then run one model locally and check where it sits and what context it really uses, compare two quant levels on twenty real inputs, read one license end to end, and point an existing script at a local endpoint by changing only the base {URL|U R L}. Next time, in chapter twenty: data privacy and token cost. What you send, where it goes, and why agents multiply the bill.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty.
