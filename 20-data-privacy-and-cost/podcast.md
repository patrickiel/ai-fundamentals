<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 20: Data privacy and token cost

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter twenty: data privacy and token cost. What you send, where it goes, and what it costs. It's the last chapter of Track five, building AI features.

MAYA: Last time, in chapter nineteen, we did open-weight and local models. And one big reason to run your own was that the data never leaves your hardware.

BEN: Right. Today is about the other case. When you do send data to a provider, which is most of the time. Here's the one sentence for today. Every prompt is data you hand to someone else, and tokens you pay for. So send only what the task needs, know the terms it's processed under, and measure what each task costs.

MAYA: Privacy and cost in one chapter. That feels like two different topics.

BEN: It does, but they meet in the middle. You'll see. Let's start with privacy. Your prompt, your attached files and the model's response all go to the provider's servers. And four policies decide what happens next. They can differ between products from the same company.

MAYA: Okay. Policy one?

BEN: Training use. As of October 2026, Anthropic lets consumer users, on Free, Pro and Max, choose whether their data trains future models. Under commercial terms, meaning Team, Enterprise and the {API|A P I}, it doesn't train on prompts or code unless the customer opts in.

MAYA: And OpenAI?

BEN: OpenAI says {API|A P I} data hasn't been used for training since the first of March 2023, unless you opt in. But consumer and business terms often differ. So check the setting on the account you actually use.

MAYA: [thoughtful] Okay. So if they don't train on it, it's gone, right?

BEN: That's the misconception the chapter warns about. "Not used for training" doesn't mean "not stored". That's policy two, retention. Anthropic documents thirty-day retention for commercial Claude Code use. Consumer data is kept five years if training is allowed, and thirty days if not.

MAYA: Five years. Wow.

BEN: And OpenAI keeps {API|A P I} abuse-monitoring logs for up to thirty days. There are exceptions, too. Anthropic may keep flagged content for up to two years, even under zero retention. And data may be kept where the law requires.

MAYA: Zero retention. That's the third policy?

BEN: Yes. Zero data retention, or {ZDR|Z D R}. Both providers offer arrangements where prompts and responses aren't stored after the response is returned. They require approval. And the key detail is feature scope.

MAYA: Meaning?

BEN: Stateful features fall outside {ZDR|Z D R} at both providers. Batch jobs, uploaded files, persistent containers, stored conversations. Using them, quote, "is a choice to step outside your ZDR arrangement for that specific data". And third-party tools you connect, such as {MCP|M C P} servers, follow their own terms.

MAYA: So zero retention has a big asterisk on it.

BEN: [laughs] A big one. And policy four is feedback and telemetry. A thumbs-down or a bug report can send a full transcript under different rules. In Claude Code, {/feedback|slash feedback} transcripts are kept for five years. Usage metrics exclude code and prompts, and can be turned off.

MAYA: Let me summarise. Four questions. Is it used for training? How long is it kept? Can I get zero retention, and which features fall outside it? And what happens to feedback?

BEN: Good. The chapter's own list adds one more question: where is it processed? That depends on how you access the model. Through a cloud platform, like Amazon Bedrock or Google Cloud, that cloud provider is the data processor, under its own terms.

MAYA: And if I need my data to stay in one region?

BEN: Data residency is usually opt-in, and priced. OpenAI lets you pin storage, and in some regions processing, per project, with a ten percent uplift on newer models. Account and billing metadata aren't covered. And Anthropic's {US|U S}-only inference costs one point one times the normal price, on Claude four point six and later models.

MAYA: Okay. Now the practical part. How do I keep secrets and personal data out?

BEN: {OWASP|oh wasp} lists sensitive information disclosure as a top {LLM|L L M} risk. That covers personal data, credentials, health, financial and confidential business data. And it warns that system-prompt instructions not to reveal data "may not always" hold.

MAYA: So "please don't leak this" in the prompt isn't a security control.

BEN: Not one you can rely on. So the chapter gives five habits. First: never paste keys, passwords or tokens. Once sent, they sit in someone's logs for the retention period. If one slips, rotate it.

MAYA: Second?

BEN: Redact before sending. Replace names, emails and account numbers with placeholders, like {CUSTOMER_1|customer one}, when the task doesn't need real values. {OWASP|Oh wasp} recommends pattern-based redaction or tokenisation.

MAYA: Third?

BEN: Scope what agents can read. A coding agent will read your {.env|dot env} file if nothing stops it. So keep credential files out of reach with deny rules or a sandbox, and give agents scoped, short-lived tokens. That's least privilege, applied to agents. Chapter thirteen explains why secrets plus a manipulated agent is the worst case.

MAYA: [sighs] I've definitely had an agent cheerfully open my {.env|dot env} file.

BEN: [laughs] You're not alone. Fourth: remember local transcripts. Claude Code keeps session transcripts on disk, in plain text, for thirty days by default. Anything the agent read is in them.

MAYA: Including that {.env|dot env} file.

BEN: Including that. And fifth: use the right account. Work data belongs under commercial terms, not a personal consumer account.

MAYA: What about the law? Is this all just good hygiene, or is there a legal side?

BEN: There's a legal side. {GDPR|G D P R}-style law, and the chapter uses the {UK|U K} {GDPR|G D P R} as explained by the {ICO|I C O}, requires personal data to be, quote, "adequate, relevant and limited to what is necessary" for the purpose. That's data minimisation.

MAYA: And for AI, that means?

BEN: The {ICO|I C O}'s guidance means four things. Identify the smallest amount of personal data the task needs, and send no more. Don't include data "just in case it's useful". Send data only about the people the task concerns. And review periodically, and delete what's no longer needed. That includes stored files, vector stores, and batch results on the provider's side.

MAYA: And here's where privacy and cost meet, I'm guessing.

BEN: [warm] You guessed it. Data you don't send can't leak, and you don't pay for its tokens. Which brings us to part two: token cost.

MAYA: Quick check. A token is roughly a word?

BEN: A token is a chunk of text, often a word fragment. Context limits, rate limits and bills are all counted in tokens. And counts depend on the tokenizer. Anthropic's newer models produce roughly thirty percent more tokens for the same text than earlier ones.

MAYA: Wait, so the same prompt can cost more, even at the same price per token?

BEN: Exactly. So use the provider's token counter, rather than rules of thumb.

MAYA: And input and output cost the same?

BEN: No. Output costs more. Every model in Anthropic's pricing table charges five times more for output than input. For example, {Claude Sonnet 5.5|Claude Sonnet five point five} is two dollars per million input tokens, and ten dollars per million output tokens.

MAYA: So output is the expensive part. I should keep answers short.

BEN: That helps a little, but for agents it misses where the money really goes. Hold on to that for a minute. First, a hidden cost. Tool use adds input you don't see. Tool definitions, a tool-use system prompt, and every tool call and result are billed as input. So every connected tool costs tokens on every request.

MAYA: Okay, now tell me where the money really goes.

BEN: Model calls are stateless. Each turn re-sends the whole conversation so far. An agent loop does this dozens of times, and each tool result makes the next request longer. Anthropic reported that in its research system, agents used about four times the tokens of a chat interaction, and multi-agent systems about fifteen times.

MAYA: Fifteen times. Is that ever worth it?

BEN: Anthropic concluded that multi-agent designs only pay off when, quote, "the value of the task is high enough". And the chapter does some illustrative arithmetic. It's not a benchmark. An agent runs twenty turns, and its context grows evenly from ten thousand to a hundred thousand tokens.

MAYA: Okay.

BEN: So the average request is about fifty-five thousand tokens. Twenty times that is about one point one million input tokens. If each turn writes a thousand output tokens, total output is only twenty thousand.

MAYA: [slowly] So even at five times the price, twenty thousand output tokens are nothing next to a million input tokens.

BEN: That's it. In agent loops, input dominates. And that's why context discipline, from chapters three to five, is also cost discipline.

MAYA: Okay, so how do I bring it down?

BEN: Four levers. Lever one, prompt caching. Most of each agent turn repeats the previous one. Caching lets the provider reuse the processed prefix, instead of re-computing it. With Anthropic, the cache matches an exact prefix, in the order tools, then system prompt, then messages. So put stable content first, and changing content last.

MAYA: How much does it save?

BEN: Cache reads cost a tenth of the base input price, and on some newer models, a twentieth. Writing to the cache costs one and a quarter times the base price for a five-minute lifetime, or twice the base price for one hour.

MAYA: And what makes it expire?

BEN: The five-minute lifetime refreshes each time the cache is used. Long pauses let it expire. And changing tool definitions, the system prompt, or some settings invalidates the cache from that point on.

MAYA: How do I know it's working?

BEN: Check the usage fields in the response. There are two: {cache_creation_input_tokens|cache creation input tokens}, and {cache_read_input_tokens|cache read input tokens}. If both are zero, nothing was cached. Applied to our example, most of those one point one million input tokens become cache reads, at a fraction of the price. The exact saving depends on how often the prefix changes.

MAYA: Lever two?

BEN: Batching. If you don't need the answer right away, like evaluations, bulk classification, or overnight document processing, batch {APIs|A P Is} process requests asynchronously. Anthropic's batch {API|A P I} charges fifty percent less on input and output, and the discount stacks with caching.

MAYA: Half price. What's the catch?

BEN: Latency and privacy. Batch jobs are stored, for twenty-nine days at Anthropic, and they aren't {ZDR|Z D R}-eligible.

MAYA: Oh, that's the stateful-features asterisk again.

BEN: Exactly, the two parts of the chapter meet again. Lever three, model routing. Not every step needs the largest model. Anthropic's own pricing guidance suggests small models for simple tasks, mid-size for most production work, and the largest for complex reasoning.

MAYA: How big is the price difference?

BEN: The input price gap between the cheapest and most expensive current models in that table is about a hundred times. So routing easy steps, like classification, extraction or simple edits, to a cheaper model, and escalating the hard ones, can save a lot. Provided quality holds, and you measure it. Chapter twenty-four, on the advisor strategy, covers one version: a cheap executor that consults a stronger model only when needed.

MAYA: And lever four?

BEN: Send less. Every lever works better when there's less to send. Shorter system prompts, fewer connected tools, subagents that return summaries instead of raw logs, and clearing context between unrelated tasks, from chapter four.

MAYA: Okay. So how do I know what a feature actually costs?

BEN: Per-token prices don't tell you. The chapter gives four steps. One, log usage per request: input, output, cache-write and cache-read tokens from each response. Two, group by task. Sum every call, including retries and subagents, that went into one user-visible outcome.

MAYA: Three?

BEN: Divide by successes. A cheap model that fails half the time and needs a retry or a human fix may cost more per completed task than an expensive one that succeeds. So pair cost with an eval, from chapter eighteen.

MAYA: [thoughtful] Oh, that's sneaky. The cheap model isn't cheap if I have to redo the work.

BEN: Right. And four, watch the trend. A prompt edit that breaks the cache prefix can quietly multiply costs.

MAYA: Is there a way to think about all these trade-offs together?

BEN: The chapter calls it the latency, cost and quality triangle. You can usually improve two of the three, at the expense of the third. A bigger model raises quality, but also cost and latency. A smaller model, or routing, lowers cost and latency, but may lower quality on hard steps. More agent turns, or multi-agent, can raise quality on complex tasks, but cost goes up a lot, and latency too.

MAYA: And the others?

BEN: Prompt caching keeps quality the same and lowers both cost and latency. Batching keeps quality, lowers cost, but adds a lot of latency, since it's asynchronous. And data residency keeps quality and latency, but adds cost, through the uplift.

MAYA: So caching is the free lunch.

BEN: The chapter calls it the rare near-free improvement. Everything else is a trade-off you should make on purpose, per task, based on measurements.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Data you send is governed by the terms of the specific product and account. So ask about training, retention, zero-retention scope, and processing location. Stateful features and third-party tools sit outside zero-retention promises. Minimise: send only what the task needs, which helps privacy and cost at the same time.

MAYA: And the rest?

BEN: Keep secrets out of prompts, and out of agents' reach. Transcripts are sensitive data too. Agent cost is driven by re-sent context, and caching, routing, batching and shorter contexts are the levers. And measure cost per successful task, not per token.

MAYA: And what will date?

BEN: Almost every number in this episode. All the prices, ratios and multipliers are as of October 2026. Five times for output, the cache read and write multipliers, the fifty percent batch discount, the one point one times {US|U S}-only inference, the ten percent residency uplift. The retention periods, and which features are {ZDR|Z D R}-eligible. The thirty percent tokenizer difference. The four times and fifteen times agent multipliers, from June 2025. And product names, plans and settings.

MAYA: Let me try to sum up, and you correct me. Every prompt goes to someone's servers, under the terms of that product and account. I ask: training, retention, zero retention and what falls outside it, and where it's processed. I keep secrets and personal data out, redact, scope what agents can read, and remember that transcripts are data too.

BEN: Good so far.

MAYA: On cost: output costs more per token, but in agent loops, input dominates, because the context is re-sent every turn. The levers are caching, batching, routing, and sending less. And I measure cost per completed task, not per call.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You have a zero data retention arrangement, and you start using batch jobs and uploaded files. Is that data covered?

PAUSE: 5

MAYA: No. Stateful features like batch jobs, uploaded files and stored conversations fall outside zero retention. Using them is a choice to step outside it for that data.

BEN: Perfect. Question two. Output tokens cost five times more than input. So why does input usually dominate the bill for an agent?

PAUSE: 5

MAYA: Because every turn re-sends the whole conversation, and it keeps growing with every tool result. Over twenty turns, that's around a million input tokens, against only twenty thousand output tokens.

BEN: Exactly. And last one. You send the same long system prompt twice, within five minutes, and both cache fields show zero. What does that tell you, and what might you check?

PAUSE: 5

MAYA: Nothing was cached. I'd check that the prefix is really identical, with stable content first, and that nothing at the start changed, like the tools or the system prompt. Changing one word at the start breaks the cache.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the third one first: add a deny rule so my coding agent can't read {.env|dot env}, then ask it to read the file and confirm it's blocked.

BEN: And audit your accounts, redact a real prompt, measure one agent task with two models, and test caching. Next time we start Track six, agent architecture, with chapter twenty-one: the harness matters as much as the model.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-one.
