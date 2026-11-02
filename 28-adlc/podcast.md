<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 28: A lifecycle for agentic systems

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter twenty-eight, and the start of Track seven, The bigger picture. The chapter is called A lifecycle for agentic systems: building software that doesn't behave the same way twice.

MAYA: Last time, in chapter twenty-seven, we ran agents in parallel. Worktrees, sandboxes, handoff specs, and one coordinator merging the results.

BEN: Right. That was about how you work with agents. Today we step back and look at the whole life of a system you build, where the decisions come from a model. From the first idea to running it in production.

MAYA: [curious] Okay. And the one sentence?

BEN: Here it is. An agent's behaviour comes from a model, prompts, tools and context together, and it varies from run to run. So the lifecycle for building one adds work that classic software rarely needed. Decide whether an agent is warranted at all. Assign responsibility between people and the agent. Prove value on real data before the full build. Evaluate continuously. And treat launch as the start of monitoring.

MAYA: Lots of extra work. First, the name. I've seen {ADLC|A D L C} around. Is that an official thing?

BEN: It stands for agent development lifecycle, or agentic development lifecycle. It describes the process of building and operating agentic systems. But vendors use different phase definitions, and there's no single standard. The chapter uses seven phases, synthesized from established guidance.

MAYA: And is this about using a coding agent to build software?

BEN: Good question, because there are two meanings to keep apart. This chapter is about the lifecycle of an agentic system you're building: software whose decisions come from a model. Using a coding agent to write that software is a separate matter. That was chapters six to eight.

MAYA: Got it. So why do we need a new lifecycle? We already have one for software.

BEN: The traditional one runs requirements, design, build, test, deploy and maintain. And it quietly assumes determinism. The same input gives the same output, so behaviour can be specified up front, and checked with pass or fail tests.

MAYA: And agents break that.

BEN: They weaken it in three ways. First, run-to-run variation. Anthropic's eval guide states plainly that "agent behavior varies between runs", so a single result is hard to interpret. And agents act over many turns, so early errors can compound.

MAYA: Second?

BEN: Behaviour is spread across components. OpenAI describes an agent as three parts: model, tools and instructions. Microsoft's build guidance also treats knowledge sources and memory as separate design decisions. Changing any one of them can change what the system does.

MAYA: [thoughtful] So I could change one sentence in a prompt, and the system behaves differently, without touching any code.

BEN: Exactly. And third, change after launch. {NIST|Nist}'s AI Risk Management Framework warns that risks measured "in a laboratory or a controlled environment" may differ from those in "operational, real-world settings". And that AI systems may need corrective maintenance due to data, model or concept drift.

MAYA: So we throw out the classic lifecycle?

BEN: No. The phases still map loosely onto the classic ones. What changes is the work inside each. Let me give you the map first. Seven phases. Frame the problem. Scope and responsibility. Design. Prove value. Build with evals. Test as a system. And deploy and operate.

MAYA: Which ones are new?

BEN: Most of them have a classic counterpart, like planning, requirements, design, development and testing. The odd one out is phase four, prove value. In classic projects that's rarely an explicit phase.

MAYA: Okay, phase one. Frame the problem.

BEN: The first question is whether an agent is the right tool at all. Anthropic recommends "finding the simplest solution possible", and notes this "might mean not building agentic systems at all". Because agents "trade latency and cost for better task performance".

MAYA: [laughs] An AI course telling me not to build agents.

BEN: [laughs] Anthropic, OpenAI and Microsoft all say it. OpenAI lists where agents earn their keep. Complex judgement calls with exceptions. Rule sets that have become hard to maintain. And heavy reliance on unstructured data. Otherwise, quote, "a deterministic solution may suffice."

MAYA: And Microsoft?

BEN: Microsoft adds two clear "no" cases. Structured, predictable tasks: use regular code. And static question answering over fixed documents: use classic RAG.

MAYA: So say an agent does look justified. Then what?

BEN: Find out how the work is done today and where it breaks. Then write testable hypotheses. The chapter's example: "an agent can draft first replies for billing tickets that support agents accept without edits at least seventy percent of the time."

MAYA: Why "testable"? Isn't the goal just "make support faster"?

BEN: Because each hypothesis needs a metric that could prove it wrong. "Make support faster" can't fail. Seventy percent accepted without edits can.

MAYA: Can a coding agent help with this phase?

BEN: Its planning mode can, provided you ask it about the behaviour of the system. Users, decisions, failure modes, assumptions. Not yet about code.

MAYA: Phase two. Scope and responsibility.

BEN: Microsoft advises setting success metrics "before any development begins". Identify the {KPIs|K P Is} the agent must improve, and record today's performance as a baseline.

MAYA: Why does the baseline matter so much?

BEN: Without a baseline, you can't later tell whether the agent helped. You'd only know where you ended up, not where you started.

MAYA: And the responsibility part?

BEN: That's the new artefact in this phase: a human–agent responsibility model. {NIST|Nist} frames human–AI configurations as a spectrum, quote, "from fully autonomous to fully manual". And it asks organisations to define and differentiate human roles in "decision making and overseeing AI systems".

MAYA: That's quite abstract. How does it look in practice?

BEN: OpenAI names two triggers for handing control to a person. Exceeding a failure threshold, like too many retries. And high-risk actions, ones that are "sensitive, irreversible, or have high stakes", such as cancelling orders or authorising large refunds.

MAYA: And Microsoft?

BEN: Microsoft recommends an agent charter. It lists what the system is responsible for, each agent's role, and prohibited actions. Plus human confirmation for high-impact actions, like database writes or financial transactions.

MAYA: Is there a simple way to start?

BEN: A simple format. List every decision in the workflow, and mark each one: agent alone, agent proposes and a human approves, or human only.

MAYA: Okay. Phase three, design.

BEN: The key distinction comes from Anthropic. Workflows, where "LLMs and tools are orchestrated through predefined code paths". And agents, which "dynamically direct their own processes and tool usage".

MAYA: We've heard that one before.

BEN: Many times. And Anthropic's catalogue of workflow patterns, prompt chaining, routing, parallelisation, orchestrator–workers, evaluator–optimizer, covers many problems without full autonomy. Microsoft goes further, and recommends deterministic workflows for critical business logic.

MAYA: Other design decisions?

BEN: Four. Start with one agent. OpenAI's advice is to "maximize a single agent's capabilities first", and add agents only when needed. Chapter twenty-three went into that.

MAYA: Second?

BEN: Choose models against a baseline. Prototype with the most capable model to set a performance baseline, then try smaller models where the results stay acceptable. Microsoft frames this as matching the model tier to task complexity, to control cost.

MAYA: [thoughtful] Oh, that's the opposite of what I'd do. I'd start cheap and upgrade if it's not good enough.

BEN: The advice is the other way round. First learn what's possible with the best model, then see how far down you can go. Third, design the tools carefully. Anthropic suggests investing as much in the agent–computer interface as in a human interface. Clear tool descriptions, examples, and formats that are hard to misuse.

MAYA: And fourth?

BEN: Define success before code, so the evals in phases four to six test something you agreed on.

MAYA: Phase four, the one that's new. Prove value.

BEN: Before the full build, test the riskiest assumptions on real data. Microsoft's advice is to "pilot the hardest steps", and to use business metrics as decision gates. Continue, change direction, or stop, based on, quote, "the data, not assumptions or optimism."

MAYA: [laughs] "Not optimism." That's a great line for a project kickoff.

BEN: [laughs] It is. And {NIST|Nist} describes the same gate, after its {MAP|map} function. Enough context "to inform an initial {go/no-go|go, no-go} decision about whether to design, develop, or deploy an AI system".

MAYA: Does anything come out of this phase besides the decision?

BEN: The first evaluation set. Anthropic suggests that "twenty to fifty simple tasks drawn from real failures is a great start". Keep it, because it becomes the regression suite later. Chapter eighteen covers how to build one.

MAYA: Let me summarise the first four phases. Ask whether I need an agent at all, and write hypotheses that could fail. Record a baseline and decide who decides what. Prefer workflows where they work, start with one agent, and pick models against the best one. Then pilot the hardest part on real data, and make a real go or no-go decision.

BEN: That's it exactly. And that's before the full build has even started.

MAYA: Phase five, build with evals in the loop.

BEN: Because behaviour lives in prompts, tool definitions and model choice, as well as code, treat all of them as versioned artefacts. Microsoft recommends storing agent instructions in source control, so changes get review and rollback. And integrating evaluations into {CI/CD|C I C D} pipelines, "to catch issues before production deployment".

MAYA: So a prompt change goes through review like a code change.

BEN: Yes. And two practices follow. Validate as you go: small prompt or tool changes can shift behaviour, so re-run the eval set after each meaningful change, instead of saving testing for the end. And test tools in a sandbox. Anthropic recommends "extensive testing in sandboxed environments", and Microsoft adds checking how the agent handles API failures and unexpected payloads.

MAYA: Anything else in this phase?

BEN: Context management belongs here too. What the agent sees at each step strongly affects quality and cost. That's chapters three to five.

MAYA: Phase six. Test as a system. How is that different from normal testing?

BEN: Several ways. First, repeat trials. Anthropic distinguishes {pass@k|pass at K}, where at least one of k attempts succeeds, from {pass^k|pass power K}, where all k succeed.

MAYA: Why does that difference matter?

BEN: Here's the chapter's example. Say an agent succeeds seventy-five percent of the time per trial. The chance that it succeeds three times in a row, {pass^3|pass power three}, is about forty-two percent. And that's the number that matters for a customer-facing agent.

MAYA: [sighs] Oof. Seventy-five sounds fine, forty-two doesn't.

BEN: Right. For a customer-facing agent, consistency is what counts. Second, grade the outcome, not the exact path, so valid alternative solutions aren't penalised.

MAYA: Third?

BEN: Keep two suites. Capability evals, which start with low pass rates. And regression evals, which should stay near a hundred percent.

MAYA: So one measures what it can't do reliably yet, and the other protects what already works.

BEN: Nicely put. Fourth, red-team it. Microsoft calls for AI red teaming against prompt injection and system-prompt extraction. That's chapter thirteen. And fifth, test in deployment-like conditions. {NIST|Nist} asks that performance be "demonstrated for conditions similar to deployment setting(s)".

MAYA: And quality is the only thing I measure?

BEN: No. Track latency, token usage and cost per task alongside quality. A change that raises the pass rate but multiplies cost may still be a regression.

MAYA: Okay, phase seven. Deploy and operate.

BEN: Deploy in stages. Google's {SRE|S R E} workbook defines canarying as "a partial and time-limited deployment of a change in a service and its evaluation". A small share of traffic gets the change, its metrics are compared with the rest, and the rollout either proceeds or rolls back.

MAYA: Like the canary in the coal mine.

BEN: [laughs] A good image for it. And for agents, include behavioural metrics alongside error rates. Task success, escalation rate, user corrections. OpenAI notes that human intervention matters most "early in deployment", to find failures and edge cases.

MAYA: And after the rollout?

BEN: Monitor behaviour, not only uptime. Microsoft recommends tracing decision points, and setting baselines and alerts for latency, cost per interaction and success rate. {NIST|Nist}'s {MANAGE|manage} function calls for post-deployment monitoring plans that capture user input, and include override, incident response and change management.

MAYA: [curious] Is there one signal that tells me it's all working?

BEN: No single signal is enough. Anthropic lists automated evals, production monitoring, {{A/B test|A B test}s|A B tests}, user feedback and manual transcript review as complementary layers. It compares them to the Swiss cheese model, where "no single evaluation layer catches every issue".

MAYA: Every slice has holes, but stacked together they cover each other.

BEN: Exactly. And then you close the loop. Turn new production failures into eval cases. Add guardrails as real edge cases appear. Re-run the regression suite before any model upgrade. And Microsoft asks teams to compare live results with the original {KPIs|K P Is} on a schedule, and decide whether to refine, retire or reinvest.

MAYA: So the baseline from phase two comes back at the very end.

BEN: It does. That's why you recorded it.

MAYA: What lasts, and what will date?

BEN: The durable principles. Use the simplest design that meets the goal. An agent is a cost you should justify. Set baselines and decide human–agent responsibilities before building. Gate investment on evidence from real data. Version everything that shapes behaviour, and evaluate with repeated trials. And roll out gradually, and keep evaluating after launch, because models, data and users change.

MAYA: And what will date?

BEN: As of October 2026: the {ADLC|A D L C} label itself, and any particular phase count. Vendor platform names in Microsoft's guidance. The twenty to fifty tasks starting heuristic. And which models count as "most capable" for baselines.

MAYA: Let me try to sum up, and you correct me. Agents vary from run to run, their behaviour lives in the model, prompts and tools, and they change after launch. So the lifecycle adds work. First, check whether I need an agent at all, and write hypotheses that could fail. Record a baseline, and decide which decisions the agent makes alone.

BEN: Good so far.

MAYA: Then design with workflows where possible and one agent first. Pilot the hardest steps on real data, with a go or no-go gate. Version prompts and tools like code, with evals in CI. Test with repeated trials, because {pass^k|pass power K} is what users feel. And launch with a canary, then keep monitoring and turning failures into eval cases.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your team wants an agent to answer questions over a fixed set of policy documents that rarely change. What does Microsoft's guidance say?

PAUSE: 5

MAYA: That's one of the two "no" cases. Static question answering over fixed documents is a job for classic RAG, not an agent. The other one is structured, predictable tasks, which are for regular code.

BEN: Exactly. Question two. You run a twenty-case eval set five times. What do you compute, and what does the gap tell you?

PAUSE: 5

MAYA: {pass@1|pass at one} and {pass^5|pass power five}. {pass^5|Pass power five} means all five trials succeed. The gap between them is my consistency problem.

BEN: Perfect. And last one. You're writing the responsibility model. Name the three ways you can mark a decision, and one trigger for handing control to a person.

PAUSE: 5

MAYA: Agent alone, agent proposes and a human approves, or human only. And a trigger from OpenAI: high-risk actions, like cancelling orders or authorising large refunds. Or exceeding a failure threshold, like too many retries.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the "do we need an agent?" test on something I actually want to automate, and writing one paragraph on whether code, a single LLM call, a workflow or an agent fits.

BEN: And try writing three hypotheses with baselines, drafting a responsibility table, building a twenty-case eval set and running it five times, and planning a canary. Next time, in chapter twenty-nine: Reward hacking and emergent misalignment. What happens when a model learns to cheat.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-nine.
