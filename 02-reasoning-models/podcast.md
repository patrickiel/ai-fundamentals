<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 02: Reasoning models

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter two: reasoning models. How to prompt models that think before they answer, and when they're worth the cost.

MAYA: Okay, so last time, in chapter one, we talked about getting better answers from a model in general. Clear instructions, real material to work from, and when to use thinking.

BEN: Right. And today we pick up that last thread. What changes when the model does its own thinking?

MAYA: [curious] My first guess would be: not much. It's a smarter model, so I write the same prompts and get better answers. Right?

BEN: Partly. But some old prompting habits stop helping, and a few can even get in the way. So here's the one sentence to remember from this episode: reasoning models already work through problems step by step before they answer, so you get better results from short, clearly structured prompts that state the constraints and the goal, and from using these models for planning and judgment while faster models handle routine execution.

MAYA: That's a long sentence. Let me shrink it. Short prompts, constraints and goal, and use them for planning, not for everything.

BEN: [laughs] That's the compressed version, yes.

MAYA: Okay. So what actually makes a reasoning model different from a normal one?

BEN: A standard model starts writing its answer straight away. A reasoning model first generates intermediate reasoning. It breaks the problem down, tries approaches, checks itself, and only then writes the final answer. OpenAI describes these models as suited to, quote, "complex problem solving, coding, scientific reasoning, and multi-step agentic workflows."

MAYA: Where did that idea come from?

BEN: From something called chain-of-thought prompting. In 2022, Wei and colleagues showed that giving a model a few worked examples, with the reasoning steps written out, improved its results on arithmetic, commonsense and symbolic reasoning tasks.

MAYA: So people were teaching the model to show its working. Like in school maths.

BEN: [laughs] That's a fair analogy. And reasoning models internalise that behaviour through training. The DeepSeek-R1 paper showed that reasoning abilities, quote, "can be incentivized through pure reinforcement learning". Behaviours such as self-verification and reflection emerged during training.

MAYA: So it picked up checking its own work while it was being trained.

BEN: That's how the paper describes it. Now, three practical consequences. First, reasoning costs tokens. OpenAI bills reasoning tokens as output tokens, and they take up space in the context window, even though you don't see them.

MAYA: [curious] Wait. I pay for tokens I never see?

BEN: You do. OpenAI's guide suggests reserving, quote, "at least {25,000|twenty-five thousand} tokens for reasoning and outputs" when you start experimenting. And Gemini and Claude also bill thinking tokens, and count them against output limits.

MAYA: Okay. Second?

BEN: Reasoning takes time. So use it where accuracy matters more than latency. And third, the line is blurring. As of October 2026, many current models think adaptively. They decide per request how much to reason.

MAYA: So "reasoning model" versus "normal model" isn't a clean split anymore.

BEN: Not a clean one, no. Now, the prompting. OpenAI's prompting guide sums up the difference. Reasoning models, quote, "will provide better results on tasks with only high-level guidance", whereas GPT models need more precise instructions.

MAYA: High-level guidance. That sounds almost too relaxed.

BEN: It's less relaxed than it sounds. The chapter turns it into six practices. Number one: keep the prompt simple and direct. The models, quote, "excel at understanding and responding to brief, clear instructions." Leave out filler and padding. Every unnecessary sentence is one more thing the model has to interpret.

MAYA: Number two?

BEN: Don't tell it how to think.

MAYA: [laughs] But "think step by step" is the one magic phrase everyone knows!

BEN: It was a useful trick for older models. But with a reasoning model, the reasoning already happens internally. So asking it to "think step by step" or "explain your reasoning" is unnecessary. OpenAI says it, quote, "may not enhance performance, and can sometimes hinder it."

MAYA: Hinder. So it can actually make things worse.

BEN: It can. And Anthropic makes a related point. A general instruction such as "think thoroughly" often produces better reasoning than a hand-written step-by-step plan, because, quote, "Claude's reasoning frequently exceeds what a human would prescribe."

MAYA: [sighs] So my carefully written plan can hold it back.

BEN: Often, yes. And one more detail, as of October 2026. On some current Claude models, a prompt that asks the model to write its reasoning into the answer may even be declined. The docs say to rely on built-in thinking instead.

MAYA: Okay, mini-summary. Don't describe how to think. Spend those words on something else.

BEN: Exactly. On clear inputs, constraints and the goal. Number three: use delimiters.

MAYA: Delimiters meaning what, exactly?

BEN: Markdown headings, XML tags, or section titles. OpenAI says they help, quote, "clearly indicate distinct parts of the input". They remove ambiguity about what role each piece of text plays. Without them, the model has to guess whether a pasted paragraph is background, an example to imitate, or an instruction to follow.

MAYA: [thoughtful] I do that a lot. I paste something in, and write "do it like this". And the model has to guess what "this" refers to.

BEN: That's exactly the ambiguity. Number four is where the vendors disagree, so pay attention. Zero-shot first, examples if needed.

MAYA: Zero-shot meaning no examples at all.

BEN: Right. And few-shot means a small number of example outputs. OpenAI advises trying the prompt with no examples first, and adding a few if the output misses. Any examples should, quote, "align very closely" with your instructions, because discrepancies produce poor results.

MAYA: And DeepSeek?

BEN: DeepSeek went further for {R1|R one}. They found the model sensitive to prompts, with few-shot prompting consistently degrading its performance. So they recommended describing the problem directly, and specifying the output format, zero-shot.

MAYA: [curious] Hold on. Examples making things worse? That goes against my instinct. Isn't showing always better than telling?

BEN: Not on every model. And to make it more interesting, Anthropic's advice for Claude is different. Multishot examples work with thinking, and worked examples shape how the model approaches similar problems in its own reasoning. It suggests presenting each one as the problem, the method, and the expected answer.

MAYA: So who's right?

BEN: The chapter doesn't pick a winner. The safe conclusion is: start without examples, add them when the output misses, keep them consistent with the instructions, and measure whether they help on the model you actually use.

MAYA: Test on my own model. Got it.

BEN: Number five: be specific about constraints and the goal. OpenAI's guide gives two related tips. First, provide specific guidelines. If there's a limit, state it. Their example is, quote, "propose a solution with a budget under {$500|five hundred dollars}".

MAYA: And the second tip?

BEN: Be very specific about your end goal. Give clear criteria for a successful response, and encourage the model to keep reasoning until it meets them. OpenAI's reasoning guide repeats this for agentic work: give the model, quote, "the task, constraints, and desired output format", and define what "done" means, and how the model should check its result.

MAYA: So the model knows what "done" looks like.

BEN: Yes. And one extra step. Explain the purpose behind a constraint. Not just "under five hundred dollars", but "under five hundred dollars, because it's a starter setup for a beginner." That gives the model something to judge its intermediate ideas against.

MAYA: Oh, that's the point from chapter one, isn't it? Give the reason behind an instruction.

BEN: It is. Anthropic's general guidance makes the same point, and that's where we covered it.

MAYA: And number six?

BEN: Ask for a self-check where it helps. Anthropic suggests ending with something like, quote, "Before you finish, verify your answer against {[test criteria]|test criteria}." It says that catches errors reliably for coding and maths.

MAYA: Sounds like something to add to every prompt.

BEN: Careful. Anthropic also warns that some newer models verify on their own, as of October 2026. And then old verification instructions can cause over-verification.

MAYA: So an old prompt can make a new model check too much.

BEN: Right. So when you change models, re-test the prompts you've inherited.

MAYA: Okay. The chapter also has a section just for the API. Do I need that if I only use the chat apps?

BEN: Only if you call models through an API. And all of it is as of October 2026. There's a table with four points, so I'll walk through them. First, developer messages. For OpenAI reasoning models, from the version called {o1-2024-12-17|oh one, twenty twenty-four, twelve, seventeen} onward, developer messages replace system messages for setting conversation-level rules.

MAYA: So it's basically the system message under a new name?

BEN: Close enough. It's the conversation-level instruction message that took over from the system message. Second, from that same release, OpenAI reasoning models avoid Markdown in API output. To turn it back on, you put the words "Formatting re-enabled" on the first line of the developer message.

MAYA: [laughs] That's a very specific magic phrase.

BEN: It is, and exactly the kind of detail that will date. Third, effort controls. OpenAI has {reasoning.effort|reasoning dot effort}, from none or minimal up to {xhigh|x high} or max, depending on the model. Anthropic has adaptive thinking plus an effort setting. And Google has {thinking_level|thinking level}.

MAYA: And the fourth point?

BEN: Output caps include thinking. OpenAI's and Gemini's {max_output_tokens|max output tokens} settings cap the thinking plus the visible output together. So with a low cap, a response can end before any answer appears.

MAYA: [thoughtful] So the model spends the whole budget thinking, and I get nothing back. And on effort: should I just turn it up to the max for anything important?

BEN: OpenAI's advice is clear. Quote, "Treat {reasoning.effort|reasoning dot effort} as a tuning knob, not the primary way to recover quality." And Google suggests matching it to the task. Low thinking for simple retrieval or classification, the default for moderate tasks, and the highest level for hard maths, advanced coding and multi-step planning.

MAYA: So the knob fine-tunes, but the prompt is still the main lever.

BEN: That's the takeaway. And as of October 2026, every major vendor exposes a thinking-effort control like this.

MAYA: Is there a pattern I can just reuse?

BEN: The chapter gives a three-part template, written in Markdown. I'll describe it instead of reading it out. First, a heading called Context, with the background the model needs: the audience, what the output is for, and relevant facts.

MAYA: And the second part?

BEN: A heading called Examples, which is optional. You add them only if a zero-shot attempt missed the style or format, and you keep them consistent with the instructions below. And last, Instructions. The task, the explicit constraints, and the goal behind them. The sample asks for an original post for a given audience, under two hundred words, matching the style of the examples. And then it says: success means a reader knows what to do next.

MAYA: [curious] What's interesting is what's not in there. No "think carefully".

BEN: That's exactly the chapter's point. No "think carefully", no step-by-step plan. The structure and the stated goal do the work.

MAYA: Okay, now the part in the title. Planners and workhorses.

BEN: That's OpenAI's framing for its two model families. Reasoning models are "the planners". They think longer, and suit strategy, planning, and decisions based on large amounts of ambiguous information. GPT models are "the workhorses". They have lower latency and cost, and suit straightforward execution.

MAYA: Do people actually combine them?

BEN: Many systems do. OpenAI's example is a customer-service flow. {GPT-4o|G P T four oh} and {GPT-4o mini|G P T four oh mini} triage the order details and customer information, and identify the issues. Then they pass those data points to {o3-mini|oh three mini}, which makes the final decision on whether a return is viable.

MAYA: So the cheap models do the sorting, and the expensive one makes the call.

BEN: Exactly. And a related OpenAI cookbook goes the other way round. {o1|oh one} turns help-center policy articles into step-by-step routines, which a cheaper agent can then follow.

MAYA: So the reasoning model can sit at the start, writing the plan, or in the middle, making the decision.

BEN: Right. The chapter sketches it as a pipeline with three stages. Cheap, fast models gather and triage data, at high volume. The reasoning model plans, decides or checks, where errors cost most. Then cheap, fast models carry out the routine steps, again at high volume.

MAYA: Let me try the lesson. Put the careful, expensive model where mistakes are costly, and let the fast ones handle volume.

BEN: That's it. Turning messy material into a plan, making the judgment call, checking the result. Chapters twenty-four and twenty-six develop this into full harness designs.

MAYA: So when exactly is a reasoning model worth it?

BEN: OpenAI lists seven patterns it has seen in practice, and notes the list isn't exhaustive. One: ambiguous tasks. They infer intent from limited or scattered information, and often ask clarifying questions instead of guessing.

MAYA: Two?

BEN: Needle in a haystack. They pull the relevant details out of large unstructured inputs, such as a single clause buried in a long contract. Three: relationships across large datasets. Reasoning across hundreds of pages of dense documents, such as contracts, financial statements and insurance claims.

MAYA: Four?

BEN: Multi-step agentic planning. The reasoning model plans, and assigns steps to GPT models according to whether intelligence or speed matters more. Five: visual reasoning. Interpreting detailed images, such as reading a legend on an architectural drawing and applying it elsewhere in the drawing.

MAYA: Six and seven?

BEN: Six: code review and debugging, often run in the background because of latency. GPT models may still be the better choice for low-latency code writing. And seven: evaluation and benchmarking. Judging other models' outputs. That gets its own chapter later, chapter eighteen.

MAYA: And when is it the wrong choice?

BEN: For simple, well-defined, high-volume tasks, a non-reasoning model or a low effort setting is usually the better trade-off.

MAYA: [thoughtful] So a reasoning model isn't simply "the better model". It's the right tool for certain jobs.

BEN: That's the right way to see it.

MAYA: Okay. What part of this will still be true in a few years, and what will date?

BEN: The durable principles first. Don't tell a model how to think if it already reasons; spend your words on clear inputs, constraints and the goal. Structure prompts with explicit delimiters. Start without examples, add them when needed, keep them consistent with the instructions, and measure. Define what success looks like, including how to check it. And split work by capability: careful models for planning and judgment, fast models for volume.

MAYA: And what will date?

BEN: Model names, like the {o-series|oh series}, {GPT-4o|G P T four oh}, {R1|R one}, and the current Claude and Gemini lines, and which ones think by default. The effort parameter names and values, the output caps, and the developer message and "Formatting re-enabled" conventions, all as of October 2026. And vendor-specific differences, such as whether examples help or hurt.

MAYA: Let me try to sum up, and you correct me. A reasoning model thinks before it answers, and that thinking costs tokens and time. So I don't tell it how to think. I write short prompts, structured with delimiters, and spend my words on the constraints and the goal, ideally with the reason behind them.

BEN: Good so far.

MAYA: Examples: zero-shot first, add them if the output misses, keep them consistent, and measure, because the vendors disagree. Effort is a tuning knob, not the main lever. And reasoning models are the planners, fast models the workhorses, so the expensive thinking goes where mistakes cost most.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You have an old prompt that says "think step by step", and you're moving it to a reasoning model. What do you change, and how do you check it?

PAUSE: 5

MAYA: Drop the phrase. The model already reasons internally, and it may not help, or can even hurt. I'd spend those words on constraints and the goal instead. And to check, run it with and without the phrase, and compare quality, length and token use.

BEN: Perfect. Question two. You ask for a specific writing style with no examples, and the output misses. What's the next step, and what's the catch?

PAUSE: 5

MAYA: Add a couple of short examples. The catch is they have to match the instructions closely, because discrepancies give poor results. And I measure whether they help, because on some models, like DeepSeek's {R1|R one}, few-shot made things worse.

BEN: Exactly. And last one. You're building a customer-service flow with lots of requests. Where does the reasoning model go?

PAUSE: 5

MAYA: Where mistakes cost most. The fast models triage the order details at high volume, and the reasoning model makes the hard call, like whether a return is viable.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. My favourite is the fourth one: ask for a recommendation under five hundred dollars, then ask again with the purpose stated, and compare.

BEN: And try restructuring a messy prompt into Context, Examples and Instructions. Or turn the effort knob: run the same hard and easy task at low and high effort, and note where higher effort helps, and where it only adds latency. Next time, in chapter three, we zoom out from the single prompt to context engineering: designing the information an agent works from, instead of crafting one clever prompt.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter three.
