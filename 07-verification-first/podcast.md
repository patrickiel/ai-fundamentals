<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 07: Verification first

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm still the one learning here.

BEN: And I'm Ben. Today is chapter seven: verification first. Giving a coding agent a way to check its own work.

MAYA: Last time, in chapter six, we talked about instruction files. And rule four of the community rules was goal-driven execution: define success criteria, and loop until verified.

BEN: Right. Today that one rule gets a whole chapter. Here's the one sentence. The most effective single improvement to an agentic coding workflow is to give the agent a check it can run, like tests, a build, a browser or a linter. Then it works inside a feedback loop, and it stops when the check passes, not when the work merely looks done.

MAYA: [curious] The single most effective improvement? That's a big claim.

BEN: It comes from Boris Cherny, who created Claude Code. He calls verification, quote, "probably the most important thing to get great results." And he estimates that with a feedback loop, the agent will double or triple the quality of the final result.

MAYA: Is that number from a study?

BEN: No, and the chapter is careful about that. It's his estimate from his own work, not a measured benchmark. But the direction is consistent across sources. A checkable target changes what the agent can do on its own.

MAYA: Okay. So why is verification such a big lever?

BEN: Think about what usually happens. Someone hands an agent a task, sees small errors or an unfinished result, and decides the tool doesn't work. But human developers rarely write correct code in one pass either. They write it, run it, see what breaks, and fix it. What makes them productive is the loop.

MAYA: So I'm judging the agent on its first draft, when I'd never judge a person that way.

BEN: Exactly. An agent without a check can only return its best guess. An agent that can run the tests, open the page, or read the compiler output can see its own mistakes and fix them before it hands the work back.

MAYA: And how does Anthropic explain it?

BEN: Their best-practices guide puts it very plainly. Quote: "Claude stops when the work looks done. Without a check it can run, 'looks done' is the only signal available, and you become the verification loop."

MAYA: [laughs] I become the verification loop. That's exactly how my afternoons feel.

BEN: [laughs] Every mistake waits for you to notice it. The guide says a check is "the difference between a session you watch and one you walk away from."

MAYA: Didn't Karpathy say something similar last chapter?

BEN: He did, and here's the longer version of that quote. "LLMs are exceptionally good at looping until they meet specific goals." Then: "Don't tell it what to do, give it success criteria and watch it go. Get it to write tests first and then pass them."

MAYA: Okay, I'm convinced. But how do I actually turn a normal task into a checkable one?

BEN: The guide shows three before-and-after prompts. The first is about verification criteria. Before: "implement a function that validates email addresses." After: write a {validateEmail|validate email} function, with example test cases. {user@example.com|user at example dot com} is true. The word "invalid" is false. {user@.com|user at dot com} is false. And then: run the tests after implementing.

MAYA: So I give it examples it can test against.

BEN: Right. The second is for UI. Before: "make the dashboard look better." After: you paste a screenshot of the design and say, implement this design, take a screenshot of the result, compare it to the original, list the differences and fix them.

MAYA: And the third?

BEN: Root causes. Before: "the build is failing." After: you paste the actual error and say, fix it and verify the build succeeds, address the root cause, don't suppress the error.

MAYA: [thoughtful] I like that last part. Otherwise it'll just make the error go away.

BEN: That's why it's in there. And three habits follow from these examples. First, test first where you can. For bugs, the guide says, "write a failing test that reproduces the issue, then fix it." For features, tests written before the code act as both the specification and the check. Chapter eight builds a whole workflow around that.

MAYA: Second habit?

BEN: Review the check, not just the code. A common pattern is to ask the agent which tests it would write, and review that list before implementation starts. You're checking intent, which is faster than reading every line.

MAYA: And third?

BEN: Ask for evidence, not assertions. Have the agent show you "the test output, the command it ran and what it returned, or a screenshot of the result." Reviewing evidence is faster than running the verification again yourself.

MAYA: So not "trust me, it works", but "here's the output".

BEN: Exactly. And you can make that the default with a standing rule in your instruction file, the kind from chapter six. The chapter suggests a small verification section with two lines. Before starting a task, state how you will verify it. And don't report a task as done until that check has passed, and show the output.

MAYA: Okay, what counts as a check? Is it always tests?

BEN: No. Cherny again: "Verification looks different for each domain. It might be as simple as running a bash command, or running a test suite, or testing the app in a browser or phone simulator. Make sure to invest in making this rock-solid."

MAYA: Can you break that down by type of work?

BEN: The chapter has a little table. For backend code, scripts and libraries, the evidence is test results, a command's exit code and output, or a diff against a fixture. For build health, it's the compiler and type-checker output. For a web UI, it's a browser the agent drives, clicking through the flow and taking screenshots. For mobile, a phone simulator the agent can operate. And for style and getting ready for CI, linters and formatters.

MAYA: Does anyone really let the agent drive a browser?

BEN: Cherny does. He says Claude tests every change he lands to {claude.ai/code|claude dot A I slash code} with the Claude Chrome extension. Quote: "It opens a browser, tests the UI, and iterates until the code works and the UX feels good."

MAYA: Okay. So I have a check. How do I make sure the agent actually uses it?

BEN: That's the next question: how hard should the check gate the stop? Anthropic's guide lists four ways, each trading setup for attention. One: in a single prompt. You ask the agent to run the check and iterate, like the examples we just went through. That works on any task today.

MAYA: Two?

BEN: Across a session. As of October 2026, you can set the check as a {/goal|slash goal} condition. A separate evaluator re-checks it after every turn.

MAYA: Three?

BEN: A deterministic gate. A Stop hook runs your check as a script, and it blocks the turn from ending until the check passes.

MAYA: And four?

BEN: A second opinion. A verification subagent reviews the result in a fresh context, quote, "so the agent doing the work isn't the one grading it."

MAYA: [laughs] Right, nobody should mark their own homework.

BEN: That's the idea. Let's look at hooks first, since they're the deterministic option. Claude Code's hooks guide says they give "deterministic control: certain actions always happen rather than relying on the LLM to choose to run them." And an agent-based hook can, for example, "verify that tests pass before allowing Claude to stop."

MAYA: [curious] Wait. If the hook blocks the stop until the tests pass, and the tests never pass, doesn't it just run forever?

BEN: Good catch, that's a real risk. Stop hooks need a guard against endless loops. Claude Code overrides a Stop hook after it blocks eight times in a row with no tool call in between. And your script should check a field called {stop_hook_active|stop hook active}.

MAYA: Okay. And the reviewer subagent?

BEN: It matters more the longer the agent runs unattended. A reviewer subagent, quote, "sees only the diff and the criteria you give it, not the reasoning that produced the change." Subagents run in their own context window, with their own prompt and their own tool access.

MAYA: So it judges the result, not the story behind it.

BEN: That's the benefit. But Anthropic adds a caution. A reviewer that's asked to find gaps will usually report some. And chasing all of them leads to over-engineering. So tell it to flag only gaps that affect correctness or the stated requirements.

MAYA: Let me summarise the four. Ask for it in the prompt, set a goal for the session, add a Stop hook as a hard gate, or get an independent reviewer. More setup, less of my attention.

BEN: [warm] Exactly right.

MAYA: What about really long tasks? Like, hours long.

BEN: Cherny lists three options for those. Quote: "{(a)|A,} prompt Claude to verify its work with a background agent when it's done, {(b)|B,} use an agent Stop hook to do that more deterministically, or {(c)|C,} use the {ralph-wiggum|ralph wiggum} plugin."

MAYA: [laughs] The ralph wiggum plugin? What's that?

BEN: It's a loop pattern. Geoffrey Huntley, who named it, describes it as "a Bash loop" that feeds the same prompt to the agent again and again, with tests as backpressure.

MAYA: So the tests are what pushes back.

BEN: Right. Anthropic's plugin implements it with a Stop hook that blocks the exit and feeds the prompt back in. The README recommends it for "well-defined tasks with clear success criteria" and "tasks with automatic verification", like tests and linters. And not for tasks that need human judgement, or have unclear success criteria.

MAYA: Any safety net?

BEN: Yes. The README says to "always use {--max-iterations|max iterations} as a safety net." And Huntley himself frames the technique as suited to greenfield work under expert supervision.

MAYA: So it's not a "set it and go on holiday" thing.

BEN: No. And here's the key line from the chapter. All three patterns remove the human from the inner loop. None of them removes the check.

MAYA: Okay, the chapter also talks about planning. How does that fit with verification?

BEN: Cherny again. Quote: "Most sessions start in Plan mode." He goes back and forth with Claude until he likes its plan. Then, quote, "I switch into auto-accept edits mode and Claude can usually {1-shot|one-shot} it. A good plan is really important!"

MAYA: What does plan mode actually do?

BEN: In plan mode, Claude "reads files, runs shell commands to explore, and writes a plan, but does not edit your source." Anthropic's guide recommends explore, plan, implement, commit for uncertain or multi-file work. And it says skip the plan when "you could describe the diff in one sentence."

MAYA: So planning is like verification for the idea, before any code exists.

BEN: That's a nice way to see it. Planning moves the cost of misunderstanding to the start, where it's cheapest.

MAYA: And permissions? If the agent is running checks all the time, I'll be clicking "allow" all day.

BEN: That's where Cherny is quite specific. Quote: "I don't use {--dangerously-skip-permissions|dangerously skip permissions}. Instead, I use {/permissions|slash permissions} to pre-allow common bash commands that I know are safe in my environment." That allowlist is checked into the project's settings file and shared with the team.

MAYA: Never skips permissions at all?

BEN: One exception. For long unattended runs, he uses a don't-ask mode, or skips permissions, but in a sandbox, so the agent isn't blocked waiting for him. Claude Code's docs match that. The bypass permissions mode is for "isolated containers and {VMs|V Ms} only", and deny rules block in every mode. Claude Code also offers an auto mode, where a classifier model reviews actions instead of you.

MAYA: So: explicit allow rules normally, and full bypass only inside isolation.

BEN: Exactly. Next, cheap checks. Verification covers form as well as function. Cherny's team uses a {PostToolUse|post tool use} hook to format Claude's code. Quote: "Claude usually generates well-formatted code out of the box, and the hook handles the last {10%|ten percent} to avoid formatting errors in CI later."

MAYA: Is that hard to set up?

BEN: The hooks guide has a ready-made example that runs Prettier on every file Claude edits. And catching it locally is cheaper than a failed CI run.

MAYA: [thoughtful] This all sounds like a lot of setup. Does Cherny have some huge custom system?

BEN: [laughs] The opposite. He calls his setup "surprisingly vanilla! Claude Code works great out of the box, so I personally don't customize it much."

MAYA: So what does he use?

BEN: A few small pieces. A shared instruction file grown from mistakes, which we covered in chapter six. Lessons are also added during code review through the Claude Code GitHub Action. Slash commands for "inner loop" workflows he repeats many times a day, checked into the project. And a few subagents that automate common steps for most pull requests.

MAYA: Which subagents?

BEN: For example, a code simplifier, and a verify app agent "with detailed instructions for testing Claude Code end to end." Several of them exist to check work.

MAYA: Of course they do.

BEN: [laughs] He also runs parallel sessions. Five in his terminal, with numbered tabs and system notifications, plus more on the web. That's chapter twenty-seven. And for the model, at the time, he used the largest model with thinking, Opus four point five. Because, quote, "since you have to steer it less and it's better at tool use, it is almost always faster than using a smaller model in the end."

MAYA: So the model is a detail that will date.

BEN: The model will date. The principle of measuring time to a correct result won't.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. An agent with a feedback loop beats one without, so make "how will this be verified?" part of every task. Choose evidence that fits the domain. Ask for evidence, not claims of success. Escalate enforcement as autonomy grows: prompt, then deterministic gate, then independent reviewer. Resolve ambiguity in a planning step, before execution. And grant autonomy through explicit permissions, bypassing checks only inside isolation.

MAYA: And what will date?

BEN: As of October 2026: Claude Code's mode names, {/goal|slash goal}, auto mode, the hook event names and loop limits, the Chrome extension, the {ralph-wiggum|ralph wiggum} plugin, Opus four point five. And the specific numbers from one person's January 2026 setup, like double or triple the quality, and five sessions.

MAYA: Let me try to sum up, and you correct me. Without a check, the agent stops when the work looks done, and I become the verification loop. So give it a check it can run, and pick the evidence that fits the work: tests, build output, a browser, a simulator, a linter.

BEN: Good so far.

MAYA: Write the check into the prompt, ideally test-first, review the test list, and ask for evidence. Then decide how hard the check gates the stop: prompt, goal, Stop hook, or reviewer subagent. Plan first, and use explicit permission rules, with full bypass only in a sandbox.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your prompt says "the build is failing". How do you rewrite it so the agent can verify its own fix?

PAUSE: 5

MAYA: Paste the actual error, and say: fix it, verify the build succeeds, and address the root cause, don't suppress the error.

BEN: Perfect. Question two. You add a Stop hook that runs your tests before the agent may stop. What guard do you need, and why?

PAUSE: 5

MAYA: A guard against endless loops. The script should check the {stop_hook_active|stop hook active} field. And Claude Code overrides a Stop hook after eight blocks in a row with no tool call in between.

BEN: Exactly. And last one. You add a reviewer subagent. Why does it help, and what should you tell it?

PAUSE: 5

MAYA: It sees only the diff and the criteria, in a fresh context, so the agent doing the work isn't grading itself. And I tell it to flag only gaps that affect correctness or the requirements, so it doesn't push me into over-engineering.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the test list: for one feature, ask only for the tests the agent would write, edit the list, and then let it implement.

BEN: And try rewriting three old prompts with the before-and-after examples, adding a Stop hook with that guard, adding a reviewer, and committing a permission allowlist with deny rules. Next time, in chapter eight, we look at spec-driven development. Spec, plan, tasks, then implementation, so you and the agent agree on what to build before it builds it.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter eight.
