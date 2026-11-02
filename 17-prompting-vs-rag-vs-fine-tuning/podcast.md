<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 17: Prompting vs. RAG vs. fine-tuning

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter seventeen: prompting, {RAG|rag} or fine-tuning. Choosing the right lever.

MAYA: Last time, in chapter sixteen, we built a {RAG|rag} pipeline in our heads. Chunk, embed, index, retrieve, and let the model answer from the passages. And we learned that retrieval quality caps everything.

BEN: Right. And today's question is the one people ask right after that. When my model isn't good enough, do I fix the prompt, add retrieval, or train the model itself?

MAYA: [laughs] And my honest instinct is: fine-tune it. That sounds like the serious, grown-up option.

BEN: That's the instinct the chapter is written to talk you out of. Here's the one sentence for today. Prompting changes what you tell the model, retrieval changes what it can look up, and fine-tuning changes the model itself. So start with the cheapest lever, measure with evals, and move to a heavier one only for a problem the lighter one demonstrably can't fix.

MAYA: Tell it, let it look things up, or change it. Cheapest first.

BEN: And measure. That part comes before any lever.

MAYA: Okay, why first? I'd have thought I start by trying things.

BEN: The guides all agree on this. Anthropic's prompt engineering docs assume you already have clear success criteria, a way to test against them empirically, and a first-draft prompt. If you don't, build those first. OpenAI describes optimisation as a loop of evals, prompting and fine-tuning, and suggests writing evals before prompts, much like behaviour-driven development.

MAYA: So, describe the behaviour you want first, then build. But for prompts.

BEN: That's the comparison they make. And Hamel Husain is blunter. Quote: "It's impossible to fine-tune effectively without an eval system."

MAYA: Why so strict, though?

BEN: Because model output is non-deterministic, and behaviour changes between model versions. Without a fixed test set, you can't tell an improvement from noise. You can't compare a prompt change with a fine-tune. And you can't tell whether a new model release broke anything. That's why chapter eighteen, on evals, is the prerequisite for everything we talk about today.

MAYA: Non-deterministic meaning, same prompt, possibly a different answer each time.

BEN: Right. So one good run on one example proves very little. You need a fixed set of cases you can run again and again, before and after every change.

MAYA: Okay. Measure first. Then the levers.

BEN: One more thing before the levers. Anthropic reminds you that not every failing eval is a prompting problem. Latency and cost, for example, are sometimes easier to fix by choosing a different model. So "switch model" is a lever too, and often the cheapest one.

MAYA: [thoughtful] Huh. I wouldn't have counted that as a lever. Okay, so what does each one actually change?

BEN: The chapter has a comparison table. I'll walk you through it. Prompting changes the instructions, examples and context in the request. It's good for most tasks, and for clarifying what you want. Time to try is minutes. You reverse it by editing the text. And the main cost is tokens per request.

MAYA: Retrieval?

BEN: Retrieval changes what knowledge the model can see at run time. It's good for large, changing or private knowledge, and for citations. Time to try is days, because you build a pipeline and an index. You reverse it by re-indexing. And the main cost is upkeep of that pipeline and index.

MAYA: And fine-tuning?

BEN: Fine-tuning changes the model's weights, so its default behaviour. It's good for a consistent format or style, narrow skills, and smaller models. Time to try is days to weeks, for data, training and evaluation. Reversing it means retraining, or rolling back to the base model. And the main cost is data, training runs, and retraining on new base models.

MAYA: So as you go down the list, it gets slower, more expensive, and harder to undo.

BEN: Exactly. And there's a shorthand worth memorising. [slowly] Retrieval for facts, fine-tuning for form.

MAYA: Facts versus form. Okay, say more.

BEN: Husain describes fine-tuning as best for teaching a model syntax, style and rules. And {RAG|rag} as the way to supply context and up-to-date facts. And a 2023 study by Ovadia and colleagues tested this directly. Unsupervised fine-tuning on new material gave modest gains. {RAG|rag} consistently did better, both for knowledge the model had partly seen and for entirely new knowledge.

MAYA: Wait, so fine-tuning is bad at teaching facts? I thought that was the whole point of training.

BEN: That's the misconception. The authors found models struggle to learn new facts through fine-tuning. Exposing them to many rephrasings of the same fact helped somewhat. But retrieval did the job better.

MAYA: So the classic mistake is...

BEN: Fine-tuning on your company wiki, hoping the model will "know" it. Retrieval does that job more reliably, keeps the knowledge current, and lets the model cite its source. That's chapter sixteen.

MAYA: [laughs] Okay, I'll admit that was exactly my plan. So how do I actually choose?

BEN: The chapter gives a decision guide in three steps. Step one: prompt, with examples. Write clear instructions, show a few input and output examples, and put the needed context in the prompt.

MAYA: And when is that enough?

BEN: Google's tuning docs say foundation models work well when the task, quote, "can be clearly and concisely defined in a prompt", and the prompt consistently produces the expected output. If that's true for you, stop there.

MAYA: Just stop?

BEN: Just stop. Husain recommends doing as much prompt engineering as possible before fine-tuning. Partly because it stress-tests your evals. And he says, quote, "I'm a big believer in using the simplest approach to solving a problem."

MAYA: [curious] But if I end up fine-tuning anyway, wasn't all that prompt work wasted?

BEN: Not at all. Besides stress-testing your evals, the well-prompted model becomes your baseline. A fine-tune has to beat that prompted baseline on your eval, or it isn't worth having.

MAYA: Step two?

BEN: Add retrieval when the problem is knowledge. Look at the failures your eval surfaces. If the model is wrong because it doesn't have the information, like internal policies, recent events or customer records, the fix is access, not training.

MAYA: And I always build a retrieval pipeline for that?

BEN: Only if the material is too large to include, changes often, or must be filtered per user. If it's small enough, just put it in the prompt. Chapter sixteen covers when that works.

MAYA: And step three is fine-tuning.

BEN: Consider fine-tuning when the problem is behaviour. The providers' docs give converging reasons. One: prompts don't produce consistent enough results, or the task is too complicated to describe in a prompt. Two: the behaviour is niche, and departs from general patterns. Like a private query language, or an unusual output format.

MAYA: Okay, and the others?

BEN: Three: you need more examples than fit in a prompt, or you want shorter prompts to save tokens and latency at scale. Four: you want a smaller, cheaper, faster model to handle a narrow task that a large model handles well, but expensively. And five: you don't want to send proprietary examples with every request.

MAYA: Are there real examples of it working?

BEN: Husain gives two. Honeycomb needed a model to write queries in its niche query language. They found fine-tuning taught the syntax and rules better than prompting with a manual and examples. And {ReChat|Re Chat} needed a very specific response format, mixing structured and unstructured data. Fine-tuning was what made it work reliably.

MAYA: Both of those are form, not facts.

BEN: Good catch. That's exactly the pattern. And he also names where fine-tuning rarely pays off. Developer tools, because frontier models already train heavily on code. General-purpose assistants. And early products without a domain-specific eval set.

MAYA: Okay. So in practice, I look at my eval failures and diagnose them?

BEN: Yes. The chapter has a symptom table. If the model misunderstands the task, you want a clearer prompt and examples. If it has wrong or missing facts, retrieval, or just put the facts in context. If the facts are right but they go stale, retrieval with a refresh plan.

MAYA: What if the content is right, but the format or tone keeps wobbling?

BEN: Better examples first, then fine-tuning. If quality is good but it's too slow or too expensive, a smaller model, and then fine-tune or distil into it. And if it's weak at complex domain reasoning, try a better model or reasoning settings first. Specialised fine-tuning is the last resort there.

MAYA: [thoughtful] Let me summarise. Misunderstanding is a prompt problem. Missing or stale facts are a retrieval problem. Format and tone, try examples, then maybe fine-tune. Cost and speed, a smaller model.

BEN: That's it. Now, "fine-tuning" isn't one thing. OpenAI's docs distinguish several methods by the kind of training signal. First, supervised fine-tuning, {SFT|S F T}. You train on examples of correct responses. That suits classification, specific output formats, and fixing instruction-following failures.

MAYA: Second?

BEN: Preference fine-tuning, for example {DPO|D P O}. You train on pairs of a better and a worse response. That suits tone, style, and what a summary should focus on.

MAYA: And third?

BEN: Reinforcement fine-tuning. A grader scores the model's outputs, and higher-scoring behaviour is reinforced. It suits complex domain reasoning, and it's only workable when experts agree on what a good answer is.

MAYA: Makes sense. If the experts can't agree, the grader can't either.

BEN: Right. And there's also a choice of how much of the model to change. Full fine-tuning updates all the weights. Parameter-efficient methods, such as {LoRA|Laura}, train small low-rank additions, and leave the original weights frozen.

MAYA: Okay. Now the costs. You've been hinting fine-tuning isn't free.

BEN: [laughs] It really isn't. First, data. You need a labelled dataset that represents the task well, plus a separate validation set. Getting good examples is usually the slowest part of the job.

MAYA: Second?

BEN: Evaluation infrastructure. You have to show that the tuned model is better than the prompted baseline on your eval, and that it hasn't got worse elsewhere.

MAYA: Got worse elsewhere?

BEN: That's the third cost, catastrophic forgetting. Training hard on one domain can erode abilities outside it. Biderman and colleagues, in 2024, compared {LoRA|Laura} with full fine-tuning on code and maths. {LoRA|Laura} learned the target domain substantially less well. But it also forgot less of the base model's other abilities.

MAYA: So it's a dial. Change more, learn more, lose more.

BEN: That's a nice way to put it. A trade-off between how much you change and how much you lose. Which is why your eval should include general tasks, not only the target task.

MAYA: Okay. Fourth?

BEN: Train and inference mismatch. Google warns, for instance, that combining a tuned Gemini model with constrained structured output at inference time can lower quality, because inference no longer matches how the model was tuned. The general point: a tuned model expects the conditions it was trained under.

MAYA: So if I train it one way and then call it a different way, I can lose what I paid for.

BEN: That's the risk. Keep the conditions at inference close to the conditions in training, and test that combination in your eval.

MAYA: And fifth?

BEN: Maintenance and lock-in. A fine-tune is tied to its base model. OpenAI's deprecation policy, for example, keeps fine-tuned models running only until their base model is retired. When the provider moves on, you retrain on the new base, and re-run your evals.

MAYA: [sighs] So it's a pet you have to keep feeding.

BEN: [laughs] Fair analogy. And provider support can change wholesale. As of October 2026, OpenAI is winding down self-serve fine-tuning. It closed to new organisations in May 2026, and plans to stop new training jobs for existing customers in January 2027.

MAYA: Wow. And a prompt?

BEN: A prompt usually carries over to a new model with modest edits. And chapter nineteen, on open-weight and local models, covers the alternative of tuning a model you host yourself.

MAYA: You mentioned distillation earlier. What is that exactly?

BEN: Distillation was introduced by Hinton, Vinyals and Dean in 2015. It's a way to compress the knowledge of a large model, or an ensemble of models, into a single smaller model that's cheaper to deploy.

MAYA: Big model teaches small model.

BEN: That's the chapter's own heading. And the modern, practical version has four steps. One, get a large model working well on your task with prompting, and retrieval if needed. Two, run it on many realistic inputs, and keep the outputs your evals rate as good. Three, fine-tune a smaller model on those input and output pairs. Four, check the small model against the same evals.

MAYA: And step two only works if my evals can actually tell good from bad.

BEN: Exactly. Another reason they come first. This is the "smaller, cheaper, faster model" case from OpenAI's list. And one practical warning: check your provider's terms, which may restrict using outputs to train other models.

MAYA: Good to know. Can I combine these levers, or do I have to pick one?

BEN: They layer naturally. Prompt plus retrieval is the default for knowledge-heavy assistants. A fine-tuned model plus retrieval fits when you need both a specialised format and current facts. The weights handle form, retrieval supplies the facts.

MAYA: Facts and form again.

BEN: Right. And a distilled small model plus a prompt fits high-volume, narrow tasks where cost and latency dominate. In every combination there's still a prompt, and the evals stay the referee. Change one lever at a time and re-run them, so you know which change did what.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Measure first: an eval set comes before any lever. Prompting changes the request, retrieval changes what the model can see, fine-tuning changes the model. Retrieval for facts, fine-tuning for form. Escalate from cheap and reversible to expensive and sticky, and only for failures you can name.

MAYA: And the fine-tuning ones?

BEN: Fine-tuning creates an asset tied to a base model. It needs maintenance, and it can forget general skills. And distillation turns a working large-model pipeline into a cheaper small model, if your evals can filter its outputs.

MAYA: And what will date?

BEN: Which providers offer fine-tuning, and on which models. OpenAI's self-serve wind-down, announced in May 2026, is the obvious example. The specific methods each platform supports, and their dataset limits. And how far prompting alone gets you, which shifts with each model generation.

MAYA: Let me try to sum up, and you correct me. Before any lever, I build an eval. Then I push the prompt as far as it goes. If failures are missing knowledge, I add retrieval, or just put the facts in the prompt. If failures are about format, style, or a niche skill, then fine-tuning comes into play.

BEN: Good so far.

MAYA: Fine-tuning costs data, eval work, maintenance when the base model is retired, and it can make the model forget general skills. Distillation is using a big model's good outputs to train a small, cheap one. And the levers combine, but I change one at a time.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your team wants to fine-tune a model on the company wiki so it "knows" internal policies. What do you suggest instead, and why?

PAUSE: 5

MAYA: Retrieval, or putting the policies in the prompt if they're small enough. Retrieval for facts, fine-tuning for form. Research found retrieval beat fine-tuning at adding facts, and it stays current and lets the model cite its source.

BEN: Perfect. Question two. Your eval shows the content is right, but the output format is inconsistent. What do you try, in what order?

PAUSE: 5

MAYA: Better examples in the prompt first, and clearer instructions. Re-run the eval. Only if that tops out, consider fine-tuning, because format is behaviour.

BEN: Exactly. And last one. Why must evals come before distillation?

PAUSE: 5

MAYA: Because distillation trains the small model on the large model's outputs that the evals rate as good. If the evals can't tell good outputs from bad, I'm training on junk. And I need the same evals to check the small model afterwards.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the first: write twenty test inputs with a pass or fail check for each, and record the score of my current prompt.

BEN: Then classify the failures, max out the prompt with three good examples, try the facts test, and plan a distillation on paper. Next time, in chapter eighteen, we go deep on the thing that kept coming up today: evals. How to measure whether an AI feature actually works.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter eighteen.
