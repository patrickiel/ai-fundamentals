<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 30: The forward deployed engineer

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and for one last time, I'm the one learning here.

BEN: And I'm Ben. Today is chapter thirty, the final chapter: the forward deployed engineer. Fitting AI into how work really gets done.

MAYA: Last time, in chapter twenty-nine, we saw a model that learned to cheat on coding tests, and how that spread into worse behaviour elsewhere. A lesson about how training shapes models.

BEN: Right. And today we end the course far from training. Inside a real business, next to the people doing the work.

MAYA: [curious] Okay, the title. Forward deployed engineer. That sounds military.

BEN: [laughs] It does a bit. We'll get to where it comes from. First, the one sentence. Getting value from AI inside a business depends less on the model than on understanding the real process. Watch the work, decide which steps belong to software, a model or a person, design for failure, test on known answers, and measure the value. The forward deployed engineer is the role built around doing that on site.

MAYA: Less on the model. After thirty chapters about models and agents, that's quite a thing to say.

BEN: [laughs] It is. But it's what this chapter's sources keep saying. Let's start with the role. A forward deployed engineer, or {FDE|F D E}, embeds with a customer and builds working software inside its systems.

MAYA: Where does the role come from?

BEN: Palantir popularised it. Palantir has long split its engineers into "Devs", who build its platforms, and "Deltas", officially Forward Deployed Software Engineers, who deploy those platforms for specific customers. Palantir describes the difference as "one customer, many capabilities", versus a product engineer's "one capability, many customers".

MAYA: So a Delta goes deep with one customer, and a Dev goes wide with one product.

BEN: Exactly. A Delta's mandate is "to achieve technical outcomes for our customers", measured "in terms of impact on the customer's goal". Palantir's example is a manufacturer trying to reduce defective products coming off an assembly line.

MAYA: [thoughtful] Hmm. Embedding with a customer, solving their problem. Isn't that just consulting with extra steps?

BEN: That's the obvious objection, and Palantir is explicit that it's not consulting. One engineer puts it this way: consultants "generally create a one-time analysis, recommendation, or solution", while Deltas build "a long-term solution" with the customer.

MAYA: Okay, that's a real difference.

BEN: And Deltas also feed fixes and features back into the core product, in coordination with the product teams. A recurring judgement call is whether to build a capability into the platform, or "create something custom that we can deliver faster".

MAYA: And AI companies do this now too?

BEN: OpenAI, Anthropic and Cursor all run {FDE|F D E} teams for AI deployments. And they describe the job as engineering, not consulting. Anthropic places its {FDE|F D E}s in its Applied AI team. They embed with strategic customers, and "work within customer systems to build production applications with Claude models".

MAYA: What do they actually deliver?

BEN: Artefacts such as MCP servers, sub-agents and agent skills. And they feed repeatable deployment patterns back to product and engineering. The job posting expects twenty-five to fifty percent travel to customer sites.

MAYA: [laughs] So pack a bag. And OpenAI?

BEN: OpenAI's head of forward deployed engineering, Colin Jarvis, told the Altimeter Capital podcast, as reported by Business Insider, that the only approach that worked consistently was to "embed directly with clients, learn their workflows, and work alongside their staff".

MAYA: And Cursor?

BEN: Cursor's {VP|V P} of forward deployed engineering, Pauline Brunet, has a nuanced view of when it fits. She says {FDE|F D E} fits best where a customer is less digitally mature, and the product is highly customisable. Both at once.

MAYA: And otherwise?

BEN: Mature customers with simple products need documentation and self-service instead. She also warns against {FDE|F D E} teams drifting into solution architecture, bug logging or product training. And she frames engagements as roughly six weeks, with a defined outcome that the customer owns, while the {FDE|F D E} stays hands-on in the build.

MAYA: Okay. But why does this role exist at all? If the models are so good, why do you need someone on site?

BEN: Because of what the chapter calls the deployment gap. {MIT NANDA|M I T Nanda}'s July 2025 report, The GenAI Divide, found that "{95%|ninety-five percent} of organizations are getting zero return" from generative AI in its research. While "just {5%|five percent} of integrated AI pilots are extracting millions in value".

MAYA: [sighs] Ninety-five percent getting nothing? That's brutal.

BEN: Hold on, we'll come back to how much weight that number can bear. First, what they did. It drew on a review of over three hundred public AI initiatives, interviews with fifty-two organisations, and a survey of a hundred and fifty-three senior leaders.

MAYA: And why did they think most failed? Bad models?

BEN: No. The authors concluded that the divide "does not seem to be driven by model quality or regulation, but seems to be determined by approach". And that most enterprise tools fail because of "brittle workflows, lack of contextual learning, and misalignment with day-to-day operations".

MAYA: Anything about who builds it?

BEN: Yes. Partnerships with external builders reached deployment about twice as often as internal builds. Roughly sixty-seven percent versus thirty-three. But the report notes these outcomes were self-reported.

MAYA: Okay, so how careful should I be with the ninety-five percent?

BEN: Quite careful. The report calls itself "preliminary findings". And ninety-five percent describes organisations in its sample, not every AI project everywhere.

MAYA: Got it. So it's a signal, not a law of nature.

BEN: Exactly. And practitioners describe the same failure from the inside. Vasuman Moza, the CEO of Varick Agents, says AI is often "slapped on top of broken processes", because nobody has mapped how the work is actually done.

MAYA: But companies have documentation of their processes, don't they?

BEN: Usually it covers the main path, and maybe an edge case or two. Not what people do when things go wrong. And those undocumented recovery paths are exactly what an agent needs to know. That's why the role puts engineers next to the people doing the work.

MAYA: Let me summarise. An {FDE|F D E} embeds with one customer and builds lasting software, not one-time recommendations. The role exists because most companies aren't getting value from AI, and the reason seems to be approach, not models. And the real process lives in people's heads, not in the documentation.

BEN: That's it exactly. Now, the practices that recur across teams. Four of them. First: build on the systems the business already runs.

MAYA: As opposed to?

BEN: Replacing them. Moza describes a client who had spent five million dollars and five years migrating to NetSuite. A proposal that required leaving it would have been rejected. His team builds, quote, "the agents on top of your systems of record". And Varick's guide puts it generally: "Build over the current data and systems rather than beginning with a large replacement project."

MAYA: Makes sense. Nobody throws away five years. Second?

BEN: Don't change the workflow more than you must. Moza's example: if operators are used to an eleven-step workflow, and it suddenly becomes one step, "adoption rates might suffer".

MAYA: [curious] Wait, really? Eleven steps down to one sounds like a dream.

BEN: It sounds like one. But the new system should be close enough to the old one that people still understand how to operate it, and how to check it.

MAYA: Which leads to trust, I guess.

BEN: It does. Third practice: budget more time for trust than for the build. At Morgan Stanley, according to Jarvis, the technical work took six to eight weeks. The team then spent another four months, quote, "running pilots, collecting evaluations, and iterating with wealth advisors".

MAYA: Four months, after a six-to-eight-week build. Was it worth it?

BEN: Afterwards, "about {98%|ninety-eight percent}" of advisors adopted the tool.

MAYA: Wow. Okay, that's the payoff. And the fourth practice?

BEN: Pick work where improvement adds up. A common heuristic is to start with tasks that happen often and cost real time, since automating a rare task saves little. And Brunet's advice to define a concrete outcome before the engagement starts serves the same purpose.

MAYA: Now the method. Five steps.

BEN: Step one: audit the real process. Varick describes starting each engagement with an audit. Its guide describes collecting context from interviews, emails, spreadsheets, {SOPs|S O Ps} and approvals. Then mapping tools, people and exceptions. And producing what it calls an "operating map": the current and future workflow, the selected use case, its boundaries, and the expected value.

MAYA: What does that look like in practice?

BEN: Watch the job being done. Write every step in the order it actually happens. And ask why each step exists.

MAYA: And if nobody knows why a step exists, I can cut it?

BEN: [laughs] Careful, that's the trap. A step nobody can justify is often a workaround that matters more than it looks.

MAYA: Ah. So that's exactly where the hidden recovery paths are.

BEN: It can be. Step two: assign each step to software, a model or a person. Varick's guide uses three filters. Deterministic software, when the rules and inputs are predictable. An agent, when the objective is clear but the inputs, paths or actions vary. And a human in control, when there's material ambiguity, accountability, or irreversible consequences.

MAYA: So the answer is rarely "AI does everything".

BEN: Rarely. In Moza's eight-step example, four steps ran autonomously, three ran with a human in the loop, and one stayed fully human. Either because the risk was too high, or because automating it wouldn't produce measurable value.

MAYA: That sounds like the responsibility table from chapter twenty-eight.

BEN: The chapter says it mirrors it. Agent alone, agent with human approval, human only. Step three: design for failure. A model returns an answer whether or not it's sure. So design for the cases where it shouldn't act alone.

MAYA: What kind of failures?

BEN: Varick's guide lists missing data, malformed responses, dead APIs, timeouts, duplicates and partial completion. And it recommends checkpointing, retries with backoff, schema validation of outputs, confidence thresholds that trigger escalation, and routing high-risk actions to a person.

MAYA: How much should the agent be allowed to do at first?

BEN: Very little. The guide advises starting "with the smallest useful action", and granting "more authority only after the system proves reliable".

MAYA: Step four?

BEN: Test on cases with known answers. Collect real cases from the audited process, where the correct outcome is known. Include normal, edge, incomplete, ambiguous and high-risk inputs. Run the system on all of them, count the correct results, then read every miss and group the failures.

MAYA: Once each?

BEN: More than once, because outputs vary between runs. This is an evaluation set, from chapter eighteen. And it turns "seems to work" into a number you can track through pilots.

MAYA: And step five, put a number on the value.

BEN: Brunet sorts business value into three kinds. Increase revenue, decrease cost, or mitigate risk. An agent's running cost only means something next to the outcome it changes.

MAYA: Example?

BEN: A good one. A customer complained that an agent was, quote, "costing two thousand dollars per day". The agent helped send the right person to repair equipment. Framed as reduced dispatch costs, the customer agreed it could be worth it. But they had "never measured it this way".

MAYA: [laughs] So the cost was obvious, and the value was invisible.

BEN: Until someone measured it. That's why you record a baseline during the audit, so the comparison is possible later.

MAYA: Okay. What kind of person does this job?

BEN: The chapter names three things. First, engineering depth. Palantir lists "technical decomp", taking a business problem "down to the lines of code you need to solve it". Alongside autonomy, and the judgement to choose between platform work and faster custom builds.

MAYA: Any concrete requirements?

BEN: Cursor currently hires {FDE|F D E}s with at least five years of software engineering and extensive customer-facing experience. Anthropic asks for production experience with LLMs, agents and evaluation frameworks.

MAYA: And the second thing?

BEN: Business understanding and empathy. Palantir mentions "high user empathy", and a desire to understand business strategy. Anthropic asks for the ability "to conduct discovery with customers". And third, comfort with ambiguity. Both Anthropic and Palantir stress working autonomously in unclear situations.

MAYA: [thoughtful] Doesn't AI make the engineering part less important now? Coding agents write the code.

BEN: The chapter's view is more careful. Coding agents lower the cost of building a first version. That shifts more of the scarce work towards discovery, process mapping and evaluation. But the companies hiring for the role still expect strong engineers.

MAYA: So how would I start, if this interests me?

BEN: Pick the smallest real process you can observe, at your workplace or a friend's business. Sit next to the person who does the most repetitive task, for an hour, while they do it. Then run the five steps on what you saw.

MAYA: And what do I end up with?

BEN: An audit document. The process as it's actually performed. The proposed split between software, model and person. The evaluation cases you'd use. And the value, in money, time or risk. It's concrete evidence of the skill, whether you apply for an {FDE|F D E} role or propose the work to a business.

MAYA: What lasts, and what will date?

BEN: The durable principles. The documented process is not the real process, so observe the work, especially the exceptions. Assign each step by predictability, judgement and cost of error. Integrate with existing systems and keep workflows recognisable, because adoption is the constraint. Expand an agent's authority only as evidence of reliability accumulates. And value is revenue gained, cost removed or risk reduced, compared with the current way.

MAYA: And what will date?

BEN: As of October 2026: team sizes and hiring requirements at specific companies. Job-posting details, like the travel share. The {MIT NANDA|M I T Nanda} figures, which come from a preliminary 2025 study. And specific integration mechanisms, like MCP servers and agent skills.

MAYA: Let me try to sum up, and you correct me. An {FDE|F D E} embeds with one customer and builds lasting software inside their systems. It's engineering, not consulting. The role exists because of a deployment gap, which seems to be about approach, not model quality. The practices: build on existing systems, don't change workflows more than you must, and spend more time on trust than on the build.

BEN: Good so far.

MAYA: And the method: audit the real process, assign each step to software, a model or a person, design for failure, test on known answers, and put a number on the value.

BEN: [warm] That's it. You've got it.

MAYA: Okay, the last quiz of the series. You ask, I answer, and everyone listening answers too.

BEN: Question one. During an audit, you find a step that nobody can justify. What do you do with it?

PAUSE: 5

MAYA: Don't just cut it. A step nobody can justify is often a workaround that matters more than it looks. It might be exactly the undocumented recovery path the agent needs to know about.

BEN: Exactly. Question two. Using Varick's three filters, when does a step go to deterministic software, when to an agent, and when to a person?

PAUSE: 5

MAYA: Software when the rules and inputs are predictable. An agent when the objective is clear but the inputs, paths or actions vary. And a person when there's real ambiguity, accountability, or irreversible consequences.

BEN: Perfect. And last one. A customer says your agent costs too much per day. How do you answer that?

PAUSE: 5

MAYA: Put the cost next to the value it creates: revenue gained, cost removed, or risk reduced, compared with the current way. Like the dispatch example. And that's why I record a baseline during the audit.

BEN: Three out of three. For the last time.

MAYA: [cheerful] And for homework, the chapter has five exercises. Shadow a process for one hour, run the three filters, build a known-answer set, price the alternative, and write a one-page audit that pulls it all together.

BEN: That last one pulls the whole chapter together.

MAYA: [warm] Which is a good moment to look back. Ben, we've covered a lot of ground.

BEN: [warm] We have. We started with Track zero, how LLMs work: tokens, training, the context window. Track one was using models. Track two, context engineering, where we learned that context is a budget. Track three, working with coding agents, and giving the agent a check it can run.

MAYA: Then Track four, tools, protocols and capabilities, with MCP, skills and the security risks. Track five, building AI features, with RAG and evals. Track six, agent architecture, from loops to harnesses to parallel agents. And Track seven, the bigger picture, which we just finished.

BEN: And the five threads from the very first episode ran through all of it. Context is a budget. Checks beat hope. Agents act with your permissions. Measure before you optimize. And the simplest thing that works.

MAYA: [thoughtful] Today had one of them, too. Testing on known answers before you trust the system. Checks beat hope.

BEN: [laughs] It did. So, how to keep learning. Do the exercises, because each one makes a single principle visible in your own work. Try ideas on a real task, not a toy example. Follow the sources when you need depth or the latest details.

MAYA: And check the dates.

BEN: And check the dates. Anything marked "as of October 2026" may have changed. The principle behind it usually hasn't. And when a new tool appears, ask which of the threads it touches. That usually tells you how to use it well.

MAYA: And the quickest review of any chapter is its "what lasts versus what will date" section.

BEN: [warm] Exactly. Thank you for learning along with us, everyone. And thank you, Maya. You asked all the right questions.

MAYA: [warm] Thank you, Ben. You made thirty chapters feel short. Thanks for listening to AI Fundamentals. Now go and try it on something real.
