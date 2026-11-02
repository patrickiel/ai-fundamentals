<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 24: The advisor strategy

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Last time, in chapter twenty-three, we compared subagents with agent teams, and saw how fast tokens add up when you add workers. Today, chapter twenty-four, is a pattern that goes the other way on cost: the advisor strategy.

MAYA: Advisor. So, a model that gives advice?

BEN: A strong model that gives advice to a cheaper one. Here's the one sentence for today. Let a faster, cheaper model run the whole task, and consult a stronger model only at the decisions that shape the outcome. You get much of the strong model's judgement, while most tokens are generated at the cheaper rate, provided the executor recognises when to ask.

MAYA: "Provided it recognises when to ask." That sounds like a big if.

BEN: [laughs] It is. We'll come back to it. Let's start with the problem. A strong model reasons better, but costs more per token and is slower. A cheaper model handles routine steps well, but can go wrong on hard decisions.

MAYA: So I pick one and live with the compromise.

BEN: That's the usual approach. Claude Code's cost guide says, quote, "Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning."

MAYA: That's easy when the whole task is either hard or easy. But a real task is a mix.

BEN: Exactly the question. How do you apply that advice inside a single long task, where most turns are routine and a few decide the outcome?

MAYA: Okay. So how does the advisor strategy answer that?

BEN: It's easiest to understand next to two older ideas. The chapter compares three approaches. The first is routing. A classifier or router decides before the task starts. If the request is routed to the strong model, that model runs for the whole thing, and writes the full answer.

MAYA: Second?

BEN: A cascade. The cheap model answers first, and a check judges that answer. Only if it's not good enough does the strong model run, and again, it produces the full answer.

MAYA: And the advisor?

BEN: In the advisor approach, the executor itself decides, in the middle of the task. The strong model runs only at chosen decision points. And it produces guidance only. No tools, no final output.

MAYA: Is there research behind routing and cascades?

BEN: There is. "Building effective agents" describes routing as sending "{easy/common|easy or common} questions to smaller, cost-efficient models", and harder ones to more capable models. RouteLLM trains routers that "dynamically select between a stronger and a weaker LLM during inference", using human preference data. It reports cost reductions of more than two times in some cases, without losing response quality.

MAYA: And cascades?

BEN: FrugalGPT queries models in order of cost, and stops when an answer is judged reliable. The authors report matching the best single model with "up to {98%|ninety-eight percent} cost reduction" on their benchmarks.

MAYA: [curious] Ninety-eight percent? Then why do we need a third approach?

BEN: Because routing and cascades pick which model answers. The advisor strategy keeps the cheap model in charge throughout, and buys the strong model's judgement in small amounts, at the points where it changes the plan.

MAYA: So routing and cascades choose a model for the answer. The advisor gives the cheap model a smart friend to call.

BEN: [laughs] As an analogy, that works. And notice that it inverts the usual orchestrator pattern. In Anthropic's multi-agent research system, an Opus lead delegates to Sonnet subagents.

MAYA: The strong model is the boss there.

BEN: Right. The advisor strategy puts the smaller model in the driver's seat, and uses frontier reasoning only when needed, "without decomposition, a worker pool, or orchestration logic".

MAYA: Okay, walk me through one task.

BEN: The chapter has a little diagram. A task goes to the executor, the cheaper model. It works: reads, edits, runs tools. At some point it reaches a hard decision, or a recurring error, or the question "am I done?". So it calls the advisor. The advisor, the stronger model, reads the full transcript, and returns a plan, a correction, or a stop signal. No tools. Then the executor applies the guidance and continues.

MAYA: So the advisor never touches the code.

BEN: Never. Anthropic says it "never calls tools or produces user-facing output". It changes decisions. The executor still does all the work.

MAYA: How do I actually use this? Is it a product, or a pattern I build myself?

BEN: Both exist, as of October 2026. On the Claude Platform API it's a beta server tool. You add a beta header, and a tool with the type {advisor_20260301|advisor, twenty twenty-six oh three oh one}, the name advisor, and the advisor's model ID.

MAYA: And then the executor has to send the advisor some context?

BEN: That's the clever bit. The executor emits an advisor call with an empty input. Quote, "The executor signals timing, and the server supplies context." The advisor receives the executor's full transcript: system prompt, tool definitions, prior turns and tool results.

MAYA: Oh, so the executor just says "help now", and the server hands over everything.

BEN: Exactly. The advisor "runs without tools and without context management". Its thinking is dropped, and "only the advice text reaches the executor". And all of that happens inside one request to the messages endpoint.

MAYA: What can I configure?

BEN: {max_uses|Max uses} caps advisor calls per request, not per conversation. {max_tokens|Max tokens}, with a minimum of one thousand twenty-four, caps the advisor's output per call. Advisor tokens are billed at the advisor's rates and reported separately in the usage data, so you can track spend per tier.

MAYA: Can any model advise any other model?

BEN: The advisor must be at least as capable as the executor. Models of equal capability can advise each other.

MAYA: And where do the savings come from, if the expensive model reads the whole transcript?

BEN: Good question. Typical advice is short, about four hundred to seven hundred text tokens. The docs say, "The cost savings come from the advisor not generating your full final output."

MAYA: Okay. And in Claude Code?

BEN: Claude Code wraps the same tool. You set it with {/advisor opus|slash advisor opus}, or with the advisor model setting, or with {claude --advisor opus|claude, dash dash advisor opus} for one session. And {/advisor off|slash advisor off} turns it off.

MAYA: Can I use it everywhere?

BEN: No. It's experimental, and it requires the Anthropic API. It isn't available on Amazon Bedrock, Google Cloud, Microsoft Foundry or Claude Platform on AWS.

MAYA: And who decides when to call the advisor? Me?

BEN: Claude does. The docs say it "tends to consult before committing to an approach, when an error keeps recurring, and before declaring a task done". There's no setting to force or cap calls. You steer it with instructions, like "consult the advisor before you continue".

MAYA: [thoughtful] What if the advice is wrong?

BEN: If Claude's own evidence contradicts the advice, for example a recommended step fails, Claude surfaces the conflict, rather than following the advice blindly.

MAYA: Good. Anything about cost in Claude Code?

BEN: One thing. Each advisor call processes the full transcript anew. The advisor's read isn't cached between calls.

MAYA: How is this different from the other ways to mix models in Claude Code?

BEN: The docs compare it with three related features. {opusplan|Opus plan}, which uses Opus during plan mode and Sonnet for execution. Subagents with their own model. And switching models with {/model|slash model}.

MAYA: Okay. Does it actually work? What are the numbers?

BEN: Anthropic's announcement, with Opus 4.6 as the advisor in all runs. On {SWE-bench Multilingual|swee bench multilingual}, Sonnet with an Opus advisor scored two point seven percentage points above Sonnet alone, while cost per agentic task fell eleven point nine percent.

MAYA: Better and cheaper. Nice.

BEN: On {BrowseComp|browse comp}, Haiku with an Opus advisor scored forty-one point two percent, against nineteen point seven percent for Haiku alone. And on {BrowseComp|browse comp} and {Terminal-Bench 2.0|terminal bench two point oh}, Sonnet with an Opus advisor scored higher than Sonnet alone, at lower cost per task.

MAYA: Anything from the API docs?

BEN: Two practical findings. First, quote, "for coding tasks, pairing a Sonnet executor at medium effort with an Opus advisor achieves intelligence comparable to Sonnet at default effort, at lower cost."

MAYA: And second?

BEN: The right setup depends on where you start. If you run Sonnet on complex tasks, an Opus advisor "keeps total cost similar or lower". If you run Haiku and want more intelligence, expect "higher cost than Haiku alone, but lower than switching the executor to a larger model."

MAYA: So with Haiku, it's not a saving. It's a cheaper way to get smarter.

BEN: That's a fair reading. And keep in mind these are vendor benchmarks. The docs say, "Results are task-dependent. Evaluate on your own workload." Anthropic's own recommendation is to run your existing evals on three setups: the cheaper model alone, the cheaper model with the advisor, and the strong model alone. Chapter eighteen covers how to build those evals.

MAYA: So when does it fit?

BEN: Good fits are long, multi-step tasks where most turns are routine, but plan quality decides the outcome. Large refactors, implementing a spec. Debugging where an error keeps recurring, and a diagnosis unlocks mechanical fixes. And tasks you want independently checked before the agent declares them done.

MAYA: And bad fits?

BEN: Short or single-turn tasks with nothing to plan. Work where every turn needs the strongest model. Then just switch the main model instead. And products where users already pick their own cost-quality trade-off.

MAYA: Now back to my big if. What happens if the executor doesn't ask?

BEN: [sighs] That's the first limit. The advice is only as good as the escalation. The executor decides when to call. Anthropic's suggested prompt tells it to call "BEFORE substantive work", when stuck, and when it believes the task is complete. And on longer tasks, "at least once before committing to an approach and once before declaring done".

MAYA: Does nudging it work?

BEN: For some executors more than others. In Anthropic's tests, a mid-conversation nudge raised Haiku's pass rate by about seven points. It had no measurable effect on Sonnet. And it slightly lowered Opus's.

MAYA: [laughs] Interesting. So the weakest model benefits most from being reminded.

BEN: In those tests, yes. The second limit: the advisor improves decisions, not execution. It has no tools and writes no code. Speed, thoroughness and how well the work is split up still depend on the executor.

MAYA: So if the task is hard because there's just a lot of tangled work?

BEN: Then, if the hard part is the sheer amount of interdependent work rather than a few key decisions, running the strong model as the main agent may finish sooner, with fewer wrong turns.

MAYA: Any practical habit to take away?

BEN: Write explicit escalation triggers into your instructions. For example, "consult the advisor before any change that touches the data model, before starting a new feature, and after two failed attempts at the same fix."

MAYA: What lasts, and what will date?

BEN: The durable part. Spend expensive reasoning where it changes the outcome, because most agent turns are routine. Separating deciding from doing lets a cheaper model do the doing. Escalation is only as good as its trigger: if the worker decides when to ask for help, it will sometimes not ask. Routing, cascades and advisors are three answers to the same cost-quality question, so pick by when the strong model is needed and what it should produce. And measure on your own workload before switching.

MAYA: And what will date?

BEN: As of October 2026: the tool name, the beta header, the parameters and pairing rules. Claude Code's {/advisor|slash advisor} command, its experimental status and provider limits. And the specific model pairings and benchmark numbers, which come from Opus 4.6 as advisor, in April 2026.

MAYA: Let me recap, and you correct me. A cheaper executor does all the work: tools, edits, the final answer. At key moments it calls a stronger advisor, which reads the whole transcript and sends back a short plan, a correction, or a stop signal. Because the advisor writes so little, most tokens stay cheap.

BEN: Good so far.

MAYA: Unlike routing and cascades, the cheap model stays in charge the whole time. It fits long tasks where a few decisions matter. Its weak spot is that the executor has to know when to ask, so I should write explicit triggers. And I should test it on my own evals against both models alone.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. What's the difference between a cascade and the advisor strategy, in terms of what the strong model produces?

PAUSE: 5

MAYA: In a cascade, the strong model runs after the cheap answer is judged not good enough, and writes the full answer. In the advisor strategy, it only gives guidance, mid-task, with no tools and no final output. The cheap model stays in charge.

BEN: Perfect. Question two. You've switched on the advisor, but the executor rarely calls it. What do you do?

PAUSE: 5

MAYA: Write explicit escalation triggers into the instructions. Like, consult the advisor before touching the data model, before starting a new feature, and after two failed attempts at the same fix.

BEN: Exactly. And last one. A task has many interacting parts, and the executor keeps going wrong despite good advice. What might that tell you?

PAUSE: 5

MAYA: That it's an execution problem, not a decision problem. The advisor improves decisions, but it doesn't write code. So running the strong model as the main agent might be better here.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I like the manual advisor: when a cheaper model fails twice on the same problem, paste the context into a stronger model, and ask only for a diagnosis and a plan, no code. Then hand the plan back.

BEN: And then audit a session for routine versus judgement-heavy turns, run the three-way comparison and check tokens with {/usage|slash usage}, add escalation rules, and find the ceiling on a task with many interacting parts. Next time, in chapter twenty-five, we look at graphs and verification. From loops to multi-agent graphs, and why every node needs its own check.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-five.
