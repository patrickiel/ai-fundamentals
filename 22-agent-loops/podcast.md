<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 22: Agent loops

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Last time, in chapter twenty-one, we said the harness around the model matters as much as the model itself. Today is chapter twenty-two: agent loops. Goal, action, check, stop.

MAYA: That sounds almost too simple to be a whole chapter.

BEN: [laughs] The skeleton is simple. The interesting part is how loops differ. So here's the one sentence for today. Every agent loop has the same skeleton, a goal, an action, a check and a stop condition, and loops differ mainly in what does the checking and what survives between rounds. So pick the loop whose check you can actually trust.

MAYA: The check you can trust. Okay. But first, what actually makes something a loop? When I chat with Claude, isn't that already a loop?

BEN: It is, but you're the one running it. Anthropic's guide on loop engineering puts it like this: "Every prompt you send starts a manual loop with you directing each turn." Loop engineering moves that job into the system. The agent acts, observes the result, and keeps going until a condition says it's finished.

MAYA: So I stop being the control system.

BEN: Exactly. And the idea is older than any product. The ReAct paper, from 2022, had the model produce, quote, "reasoning traces and task-specific actions in an interleaved manner". Reasoning can then "induce, track, and update action plans", while the actions fetch new information.

MAYA: Think, act, look at what happened, think again.

BEN: That's it. And Anthropic's "Building effective agents" describes an agent as "typically just LLMs using tools based on environmental feedback in a loop". It adds two requirements. The agent must gain "ground truth" from the environment at each step, like tool call results or code execution, to assess its progress. And "it's also common to include stopping conditions (such as a maximum number of iterations) to maintain control".

MAYA: Ground truth and a stopping condition.

BEN: Right. And most practical loops mix a fixed outer workflow, say build, check, repeat, with an agent making decisions inside each step.

MAYA: Okay, so walk me through the skeleton.

BEN: The chapter has a little table with four parts. The goal answers: what does "done" mean? For example, "all tests in the auth test folder pass and lint is clean". The action answers: what does the agent do each round? Edit code, revise a skill, apply review fixes.

MAYA: And the other two?

BEN: The check answers: how is the result judged? A test suite, a build, a separate model, or a rubric. And the stop condition answers: when does the loop end? When the check passes, when a turn or time cap is hit, or when the goal is judged impossible.

MAYA: Is that everything that distinguishes loops?

BEN: Two more properties. State: does anything persist between rounds, like a progress file or a journal? And who checks: the same agent, a separate model, several critics, or a deterministic tool?

MAYA: [curious] I've heard Claude Code has different kinds of loops. Is that the same classification?

BEN: That's a different axis. Anthropic's guide sorts loops by what starts each cycle. As of October 2026, there are four types. Turn-based loops start with your prompt, and stop when Claude judges it's done or needs input. That's an ordinary session.

MAYA: Next?

BEN: Goal-based. A prompt with a condition, that stops when the goal is met or a turn cap is hit. In Claude Code that's {/goal|slash goal}. Then time-based, on an interval, until you cancel or the work completes. That's {/loop|slash loop} and {/schedule|slash schedule}. And proactive: triggered by an event or schedule with no human present, using routines together with {/goal|slash goal} and skills.

MAYA: So that tells me when a loop runs.

BEN: Right. The chapter's five patterns sort loops by how they judge their own work instead. And that's what decides whether a loop converges or just burns tokens. The guide points the same way: "The more quantitative the checks are, the easier it is for Claude to self-verify."

MAYA: Okay, five patterns. Number one?

BEN: The single-task loop. It does one job, checks it and stops. It carries little or nothing forward in the model's context. The best-known form is called Ralph, from Geoffrey Huntley.

MAYA: [laughs] Ralph?

BEN: Ralph. Huntley says that in its purest form, it's a Bash loop. An endless loop that pipes a prompt file into the coding agent, over and over. Each pass starts with a fresh context window, and does "only one thing".

MAYA: Wait, if every pass starts fresh, how does it remember anything?

BEN: Good catch. Ralph is stateless only in its context. The state lives in files. Specs, a fix plan the agent updates, and an {AGENT.md|agent dot M D} on how to build and run the project.

MAYA: So fresh context doesn't mean no memory. The memory just moved out of the window and into files.

BEN: That's a good way to put it. And that's a theme across all five patterns: what survives between rounds, and where it lives.

MAYA: And the check?

BEN: Huntley calls it "back pressure". Tests, type checkers and static analysis that "reject invalid code generation". He positions Ralph for greenfield work, and says it still needs senior engineers to guide it.

MAYA: And the other form?

BEN: {/goal|slash goal} in Claude Code. You state a completion condition. After each turn, a separate "small fast model" judges it not yet met, met, or impossible. So, quote, "completion is decided by a fresh model rather than the one doing the work".

MAYA: Oh, that's nice. The worker doesn't get to grade itself.

BEN: Right. But there's a catch. The evaluator "doesn't run commands or read files independently". So the condition must be something Claude's own output can show, like a test run in the transcript.

MAYA: So if my condition is "the tests pass", but the tests never actually run in the session, the evaluator has nothing to go on.

BEN: Right. It only sees what's in the conversation. So make the check part of the work, and make its result visible.

MAYA: How do I write a good condition?

BEN: The docs say: "one measurable end state", "a stated check", the constraints that must hold, and a cap, like "or stop after 20 turns". And under the hood, {/goal|slash goal} wraps a session-scoped, prompt-based stop hook. We met stop hooks last chapter.

MAYA: So far it sounds like the model saying "I'm done" is the check. That makes me a bit nervous.

BEN: It should. A model's opinion that the work is done is the weakest check. The fix is to write the tests before the feature. Then the goal becomes "make these tests pass; don't modify the tests".

MAYA: So the stop condition is evidence, not an opinion.

BEN: Exactly. And tests for other features act as guardrails. If the agent breaks something it shouldn't touch, a test fails. Anthropic's harness research found the failure you get without such checks. Claude tended to declare victory on the whole project too early, and to "mark a feature as complete without proper testing".

MAYA: [sighs] Declaring victory too early. Very human.

BEN: [laughs] Very. One more thing for unattended runs: you need a way back. In Anthropic's long-running harness, each session commits its work with a descriptive message. So the agent can "use git to revert bad code changes and recover working states of the code base". A one-line instruction in your agent's project file, like "commit after each passing test run", gives the single-task loop the same safety net.

MAYA: So single-task fits well-specified work with an objective check.

BEN: Tests, a build, a schema validator, an empty queue. Yes.

MAYA: Pattern two?

BEN: The learning loop. Its job is to improve something you'll reuse, like a skill, a prompt or a workflow, and to write down what it learned, so later rounds and later users benefit.

MAYA: Is there research behind that one?

BEN: The root is Reflexion, by Shinn and colleagues, from 2023. Agents "verbally reflect on task feedback signals, then maintain their own reflective text in an episodic memory buffer". That guides later attempts, and improves behaviour without changing the model's weights. Ralph's {AGENT.md|agent dot M D} is a practical version. The loop updates it with what it learns about building and running the project.

MAYA: How would I design one?

BEN: Two parts. First, a baseline comparison. Run the same task with and without the asset, and compare. Without a baseline, you can't tell whether a skill helps at all. Chapter eighteen covers evals in depth.

MAYA: [thoughtful] I'll admit, I've added skills and just assumed they helped.

BEN: Most people do. The second part is a journal that lives with the asset. For example a {learning.md|learning dot M D} next to the skill, recording what was tried, the measured result, and the lesson. It stops the loop repeating failed experiments, and tells future runs what caused problems before.

MAYA: So it fits assets I'll run many times, where small improvements add up.

BEN: Exactly.

MAYA: Pattern three?

BEN: The multi-critic review loop. One reviewer asked to judge everything tends to miss things. So a common pattern splits review across specialised critics. For example factual accuracy, relevance, safety and style. Each with a narrow brief and the tools that brief needs, like web search for the fact-checker.

MAYA: Where does that come from?

BEN: The root is what "Building effective agents" calls parallelization by voting: "running the same task multiple times to get diverse outputs". For example, several prompts that each review code for vulnerabilities.

MAYA: And there's a Karpathy thing in the chapter, right?

BEN: Andrej Karpathy's llm-council. Several models answer, rank each other's answers anonymously, and a "chairman" model writes the final reply. But he calls it a "fun Saturday hack" he won't support. So treat it as inspiration, not a tool to depend on.

MAYA: Do the critics talk to each other?

BEN: Two options. A central orchestrator that runs the rounds keeps the history of earlier rounds in one place. Critics that message each other give more independent views, at more cost. Chapter twenty-three compares the two.

MAYA: So multi-critic fits output judged on several separate qualities, where no single test can say "correct".

BEN: That's it.

MAYA: Number four.

BEN: The build-and-score loop. One agent builds, another scores, and the builder's job is to raise the score against a fixed rubric. This is the evaluator-optimizer workflow: "one LLM call generates a response while another provides evaluation and feedback in a loop". Anthropic says it's "particularly effective when we have clear evaluation criteria", and when feedback demonstrably improves the result.

MAYA: What makes the score trustworthy?

BEN: Three design choices. First, separate the judge from the builder. Anthropic's guide notes that "a reviewer with fresh context is less biased and not influenced by the main agent's reasoning". Same reason {/goal|slash goal} uses a separate evaluator.

MAYA: Second?

BEN: Give the scorer no write access. A reviewer that can't edit can't quietly fix things, or grade its own changes. The chapter says that's general practice, and easy to enforce with a read-only tool list.

MAYA: And third?

BEN: Persist scores between rounds, for example in a JSON file. Then you can see whether the loop is converging or stuck.

MAYA: [curious] Should I always use lots of reviewers, then? More eyes, better score?

BEN: Breadth costs money. Save reviewers that fan out to many subagents for large, finished codebases. For everyday work, use one reviewer with a clear rubric.

MAYA: Okay. So build-and-score fits when I can state a quality metric, and want the work to converge on it.

BEN: Right. And that brings us to the fifth. None of the loops so far improves the loop itself.

MAYA: Ooh. Meta.

BEN: [laughs] A little. The process-improvement loop adds a step after each round. Review how the round went, and change the process, the prompts, the rubric, the step order, the checks. Not just the output.

MAYA: Who says that's a good idea?

BEN: Anthropic's guide states the principle. When a result misses the standard, encode the fix into the system, rather than only patching that one result. Huntley describes the manual version. Each time Ralph goes wrong, you add a "sign" to the prompt.

MAYA: Like a sign on the wall. "Don't do that again."

BEN: [laughs] That's the picture. A process-improvement loop automates part of that tuning, with a separate agent that reads the round's transcript and proposes changes. But keep a human approving them. A loop that rewrites its own rubric can drift toward whatever is easiest to score. That's chapter twenty-nine.

MAYA: So it fits processes I'll run many times, where a better process pays off across many future runs.

BEN: Exactly.

MAYA: Okay, how do I choose between the five?

BEN: The chapter has a table, and it comes down to the check. To finish a well-specified task, use single-task, because the check is objective: tests, a build, a validator. To improve a reusable skill or prompt, use the learning loop, with a with-and-without comparison logged over time.

MAYA: And the rest?

BEN: To review output on many dimensions, use multi-critic review, with several specialised critics. To push quality toward a target, build-and-score, with a separate, read-only scorer and a rubric. And to improve how you build, not just what, the process-improvement loop, with a score plus a reviewed critique of the process.

MAYA: What about cost? Some of these sound expensive.

BEN: Cost scales with agents times rounds. Anthropic's guidance is to match the primitive and the model to the job, use scripts for deterministic steps, and "pilot before a large run". They say model and effort level are "among the biggest levers on what a loop costs". And "Building effective agents" puts it more generally: aim for "the simplest solution possible, and only increasing complexity when needed".

MAYA: So the rule of thumb: start with the cheapest loop whose check I trust, and only add agents or rounds when that check can't tell good work from bad.

BEN: [warm] That's exactly the chapter's rule of thumb.

MAYA: What lasts, and what will date?

BEN: The durable part. A loop is only as good as its check, so invest in tests, rubrics and baselines before investing in autonomy. Prefer objective stop conditions to a model's own judgement, and always have a cap. Separate the judge from the builder, and give scorers no write access. Measure against a baseline before claiming an asset helps. Persist state in files, like plans, progress logs, journals and git history, not in the context window. And checkpoint working states, so autonomous runs can roll back.

MAYA: And what will date?

BEN: Claude Code's {/goal|slash goal}, {/loop|slash loop}, {/schedule|slash schedule} and routines, and the evaluator model and its behaviour, as of October 2026. Anthropic's four-way trigger taxonomy, which comes from a June 2026 guide. And the Ralph technique's specific files, and its 2025 framing.

MAYA: Let me recap, and you correct me. Every loop has a goal, an action, a check and a stop condition. Plus state, and who does the checking. Anthropic sorts Claude Code loops by trigger: turn-based, goal-based, time-based and proactive.

BEN: Good so far.

MAYA: And by how they judge their work, there are five patterns. Single-task, learning, multi-critic review, build-and-score, and process-improvement. The weakest check is the model's own opinion, so write tests first, separate the judge from the builder, and always have a cap.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You want an agent to implement a small function unattended. How do you set up the goal so the stop condition is evidence, not opinion?

PAUSE: 5

MAYA: Write the failing tests first. Then the goal is "make these tests pass, don't modify the tests", with a cap, like stop after ten turns. And have it commit after each passing run, so I can roll back.

BEN: Perfect. Question two. In a build-and-score loop, what three choices make the score trustworthy?

PAUSE: 5

MAYA: Separate the judge from the builder, so the reviewer has fresh context. Give the scorer no write access. And persist the scores between rounds, so I can see if it's converging or stuck.

BEN: Exactly. And last one. You've written a skill and you think it helps. How does a learning loop find out?

PAUSE: 5

MAYA: A baseline comparison. Run the same task with and without the skill. And keep a journal next to it, like {learning.md|learning dot M D}, with what was tried, the result, and the lesson.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the first: write three to five failing tests, then give {/goal|slash goal} a condition like "all tests in this file pass without modifying the test file, or stop after ten turns", and note the rounds and tokens.

BEN: And then annotate the anatomy of a workflow you already use, run a with-and-without baseline, split a reviewer into separate critics, and try a build-and-score loop with a hundred-point rubric. Next time, in chapter twenty-three, we compare subagents with agent teams. Workers that report back, versus sessions that coordinate with each other.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-three.
