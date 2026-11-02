<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 26: Agent harness design

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter twenty-six: agent harness design. Plan, generate, evaluate, and keep doing separate from judging.

MAYA: Last time, in chapter twenty-five, we went from loops to multi-agent graphs, and the big lesson was that every node needs its own check.

BEN: Right. Today we zoom in on one particular design that Anthropic tested for building whole applications. And on a way of thinking about every piece of scaffolding you put around a model.

MAYA: [curious] Start with the word. What's a harness, exactly?

BEN: An agent harness is the scaffolding around a model that lets it finish long tasks. Roles, plans, checks, files, how the context is handled. Everything that isn't the model itself.

MAYA: Okay. And the one sentence for today?

BEN: Here it is. Every component of an agent harness encodes an assumption about what the model can't do alone. So keep the parts that still earn their place, a product-level planner, a generator, and a separate, skeptical evaluator that grades against explicit criteria. And re-test the rest by removing it whenever the model changes.

MAYA: That's a long sentence. Let me start with the first half. Components are assumptions?

BEN: That's the central claim of a post by Prithvi Rajasekaran at Anthropic, on harness design for long-running application development. It opens with, quote, "Harness design is key to performance at the frontier of agentic coding." And the line that matters most is that "every component in a harness encodes an assumption about what the model can't do on its own".

MAYA: So if I add a planning step, I'm assuming the model can't plan well enough by itself.

BEN: Exactly. And those assumptions should be stress-tested, for two reasons. They may be wrong. And they expire, because models improve.

MAYA: How do you test an assumption like that?

BEN: You ablate. Remove one component, run a realistic task, compare. Keep the component only if removing it makes the results worse.

MAYA: [thoughtful] Why one at a time? Wouldn't it be faster to strip everything out and see what happens?

BEN: That's what Rajasekaran tried first. Radical simplification. And he couldn't match the original harness's performance. Removing components one by one is what worked. And you repeat the exercise whenever you change models.

MAYA: So eventually the harness just disappears, as models get better?

BEN: That's the natural guess, and the post says no. Its closing line is that the space of useful harnesses, quote, "doesn't shrink as models improve. Instead, it moves."

MAYA: Oh, I like that. The scaffolding doesn't vanish, it moves to wherever the model is weak now.

BEN: Yes. And it matches Anthropic's earlier general advice for agentic systems: find the simplest solution that works, and add complexity only when it demonstrably improves results.

MAYA: Okay, so the design that held up. What was it?

BEN: Planner, then generator, then evaluator. It was inspired by GANs, generative adversarial networks, where one network generates and another judges. And it's close to the evaluator-optimizer workflow from Anthropic's "Building effective agents" guide.

MAYA: Let's go through them. Planner first.

BEN: The planner takes a short prompt, one to four sentences, and expands it into a full product spec. Two choices matter. First, ambitious scope. It was prompted "to be ambitious about scope", and to look for chances to weave AI features into the product.

MAYA: And the second?

BEN: It stays at the product level, not the code level. In the post's words, it focuses on "product context and high level technical design rather than detailed technical implementation".

MAYA: [curious] Wait, that surprises me. I'd have thought a more detailed spec is always better. Less room for the agent to go wrong.

BEN: That's the misconception the chapter targets. The reason given is cascading errors. If the spec fixes technical details too early, any mistake in them is built into everything downstream. The post says early errors "cascade into the downstream implementation". And the agents lose the freedom to find a better path.

MAYA: So a detailed spec doesn't remove mistakes. It just bakes in the planner's mistakes.

BEN: Nicely put. Plan the product, not the implementation.

MAYA: Did the planner survive the ablations?

BEN: It did, even with the strongest model tested. When Rajasekaran removed it, the generator under-scoped its work. It built less than it should have.

MAYA: Interesting. So without a planner, the model aims too low.

BEN: Right. And there's a related lesson from Anthropic's earlier harness work, by Justin Young. When a single agent is handed a large goal, it tends, quote, "to try to do too much at once". It attempts to one-shot the whole app, and runs out of context halfway through.

MAYA: So one problem is aiming too low, the other is trying to do everything at once.

BEN: And the fix in that earlier work was a structured feature list. Over two hundred features, each one initially marked as failing. Later sessions worked through them one at a time. And notice what the list describes: what the user should be able to do. Not how to code it.

MAYA: Product level again. Okay, now the part in the title. Separating the doer from the judge.

BEN: This is my favourite finding. Asked to evaluate its own output, a model tends to praise it, quote, "even when, to a human observer, the quality is obviously mediocre." The post says agents "reliably skew positive when grading their own work".

MAYA: [laughs] That's very human, honestly.

BEN: It is. And the effect is strongest on subjective work, such as design. Where there's an objective check, like does this endpoint return the right data, the problem is manageable. Where there isn't, the builder happily signs off on its own average work.

MAYA: So the fix is to have a second model do the judging.

BEN: That's half of it. Rajasekaran's finding is that "tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work."

MAYA: Hmm. Why only half?

BEN: Because separation alone doesn't fix leniency. The evaluator is still a model, still inclined to be generous. What separation gives you is a component you can tune on its own. Claude Code's documentation gives the same advice for everyday work: "A fresh context improves code review since Claude won't be biased toward code it just wrote."

MAYA: That's code review, really. Someone other than the author looks at it.

BEN: Exactly. The same principle runs through human engineering. Code review by someone other than the author, QA separate from development, editors separate from writers.

MAYA: Let me summarise so far. Every component is an assumption, so test it by removing it. Plan the product, not the code. And don't let the builder grade its own work.

BEN: That's it. Now let's look at the three roles in more detail.

MAYA: The planner we've covered. It writes the spec.

BEN: And it writes it to a file that the other agents read. Then the generator builds the app feature by feature from that spec. In Anthropic's runs, the stack was React, Vite, FastAPI and a database. It commits to git as it goes, checks its own work at the end of each chunk, and then hands off to QA.

MAYA: Any habits that make that work across many sessions?

BEN: Anthropic's earlier harness adds a few. A progress file, called {claude-progress.txt|claude progress dot T X T}. A script called {init.sh|init dot S H} that starts the dev server. Descriptive commits, so bad changes can be reverted. And a start-of-session routine: read the log, and run a basic end-to-end test before touching anything new.

MAYA: Check that things still work before you change them. Sensible. And then the evaluator.

BEN: The evaluator is the adversary. It reads the spec to learn what "done" means. Then it exercises the running app through the Playwright MCP server. It clicks through flows, checks API endpoints, and inspects the database state.

MAYA: [curious] So it actually uses the app? It doesn't just read the code?

BEN: It uses the app, like a user would. Then it grades against criteria and returns specific feedback. In the earlier harness, giving the agent browser automation, through the Puppeteer MCP server, so it could test "as a human user would", markedly improved results.

MAYA: Any blind spots?

BEN: One the chapter mentions. Browser-native alert dialogs. The agent couldn't see them.

MAYA: And was the evaluator good straight away?

BEN: [laughs] No. The post is blunt: "Out of the box, Claude is a poor QA agent." It found real issues, and then talked itself into approving anyway. And it "tended to test superficially, rather than probing edge cases".

MAYA: So it had the same leniency problem as the generator.

BEN: The same inclination, yes. It took several rounds of reading its logs, comparing them with human judgement, and revising the prompt, before its grading was reasonable.

MAYA: Which is the point you made earlier. You can tune a separate evaluator. You just have to actually do it.

BEN: Exactly. And the most important tuning tool is telling the judge what "good" means. An evaluator with no criteria falls back on the model's defaults.

MAYA: What criteria did they use?

BEN: For the front-end design experiments, the generator and the evaluator got the same four criteria. Two weighted higher, two lower. The higher ones were design quality: do colours, typography, layout and imagery form "a coherent whole", with a distinct mood? And originality: custom decisions, or template layouts, library defaults and recognisable AI patterns?

MAYA: What counts as a recognisable AI pattern?

BEN: The post calls out purple gradients over white cards.

MAYA: [laughs] Oh, I've seen that one so many times.

BEN: [laughs] Everyone has. The two lower-weighted criteria were craft and functionality. Craft is typography hierarchy, spacing, colour harmony, contrast. The post calls it "a competence check rather than a creativity check". And functionality: can users understand the interface and find the primary actions?

MAYA: Why weight craft and functionality lower? They sound important.

BEN: Because "Claude already scored well on craft and functionality by default." So weighting design quality and originality more heavily put the pressure where the model was weak, and pushed it toward aesthetic risk-taking.

MAYA: So the weights aren't about what matters in general. They're about where this model needs a push.

BEN: That's a good way to read it. The evaluator was also calibrated with few-shot examples that showed detailed score breakdowns. And the wording of the criteria visibly shaped the character of the output. Design runs used five to fifteen iterations.

MAYA: And for full applications, not just design?

BEN: The QA agent graded each chunk on product depth, functionality, visual design and code quality. And each criterion had a hard threshold. If any single one fell below it, the chunk failed and went back with feedback.

MAYA: [thoughtful] Why bother with scores? Wouldn't pass or fail be simpler?

BEN: Simpler, but weaker, for three reasons. A score per criterion shows where the work is weak, not just that it failed. Weights tell both agents what matters most. And a pass or fail check against the plan can pass work that technically matches the spec, but is poor.

MAYA: Got it. And this is the evaluator-optimizer thing you mentioned.

BEN: Yes. From "Building effective agents": one model generates, another evaluates and feeds back, in a loop. It fits when, quote, "we have clear evaluation criteria", and when iteration adds measurable value. And as of October 2026, Anthropic also offers rubric grading in Claude Managed Agents, in beta. A separate grader checks outcomes against a rubric, and failures loop back for rework.

MAYA: Okay. The chapter also had a section on scaffolding that depends on the model.

BEN: Two components show how "necessary" depends on the model. The first is sprint contracts. Before each chunk of work, the generator proposes what it will build, and how success will be tested. The evaluator reviews the proposal, and they iterate until they agree. All before any code is written.

MAYA: How do two agents negotiate?

BEN: Through files. One writes, the other responds in the same file or a new one. One example contract had twenty-seven criteria. The contract turns a product-level spec into concrete, checkable steps, without fixing the implementation too early.

MAYA: That sounds really useful. Did it survive?

BEN: Not with the newer model. When Opus four point six arrived, Rajasekaran removed the sprint construct entirely, expecting the newer model to need less scaffolding. He kept the planner and the evaluator, but moved the evaluator from grading every sprint to a single pass at the end.

MAYA: So even the evaluator got scaled back.

BEN: Right. His conclusion is that the evaluator "is not a fixed yes-or-no decision". It "is worth the cost when the task sits beyond what the current model does reliably solo".

MAYA: And the second model-dependent component?

BEN: Context resets, which deal with something called context anxiety. Some models, as they approach what they perceive as their context limit, start wrapping up early.

MAYA: [laughs] Context anxiety. Like me packing up ten minutes before a meeting ends.

BEN: [laughs] Something like that. And Anthropic's earlier harness saw a related failure. A later session would, quote, "look around, see that progress had been made, and declare the job done."

MAYA: How do you fix that? Compaction?

BEN: There are two remedies, and they differ. Compaction summarises the earlier conversation in place. It keeps continuity, but the anxiety can persist. A context reset clears the window and starts a fresh agent from a structured handoff stored in files. A clean slate, at the cost of orchestration overhead, tokens and latency.

MAYA: Which one did they use?

BEN: It depended on the model. Claude Sonnet four point five showed the behaviour strongly enough that resets were essential. Opus four point five "largely removed that behavior", so the later harness dropped resets and relied on automatic compaction.

MAYA: So the lesson isn't "always reset" or "never reset".

BEN: The durable lesson is the diagnostic. If your agent declares success early as its context fills, external task files plus fresh contexts are a proven fix. If it doesn't, you may not need them.

MAYA: Okay, the practical question. What does all this cost?

BEN: It's expensive, and the post reports the numbers. Building a retro game maker with Opus four point five took twenty minutes and nine dollars solo. With the full harness, six hours and about two hundred dollars.

MAYA: [sighs] That's a big difference. Was it worth it?

BEN: The solo version looked plausible, but nothing responded to input. The harness version was playable.

MAYA: Ah. So the cheap one only looked finished.

BEN: Right. Another example: a browser-based audio workstation, built with the simplified Opus four point six harness, took about four hours and a hundred and twenty-five dollars. Most of that was the generator's build rounds. The QA rounds cost a few dollars each, and still caught real gaps, like clips that couldn't be moved, and recording that was only stubbed out.

MAYA: So the judge was cheap compared to the builder, and it still earned its keep.

BEN: In that case, yes. But weigh the cost against the value of the output. For tasks the model already handles reliably alone, the extra roles are overhead.

MAYA: Before we recap: what lasts, and what will date?

BEN: The durable principles. Every harness component is an assumption about model weakness, so test it by removing it, and test again when the model changes. Separate doing from judging. Plan outcomes, not keystrokes. Make "good" explicit with weighted criteria and hard thresholds, especially for subjective qualities. And let the evaluator use the product like a user, while keeping durable task state in files and git, outside the context window.

MAYA: And what will date?

BEN: As of October 2026: which components each model needs, Sonnet four point five, Opus four point five, Opus four point six. The reported costs and durations. The specific stack. Playwright and Puppeteer MCP as testing tools. And the Managed Agents rubrics, which are in beta.

MAYA: Let me try to sum up, and you correct me. A harness is the scaffolding around a model, and every piece of it is an assumption about what the model can't do alone. You test each piece by removing it, one at a time, and you test again with every new model.

BEN: Good so far.

MAYA: The design that held up was planner, generator, evaluator. The planner describes the product, not the code. The builder doesn't grade its own work, because self-review skews positive. And the evaluator uses the app like a user, and grades against weighted criteria with hard thresholds. And things like sprint contracts and context resets helped older models, but got removed for newer ones.

BEN: [warm] That's it. One small addition: separating the judge doesn't fix leniency on its own. It gives you something you can tune.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You ask an agent to build a small UI, and then ask the same session to rate it out of ten. It gives itself an eight. Why shouldn't you trust that, and what's better?

PAUSE: 5

MAYA: Because agents reliably skew positive when grading their own work, especially on subjective things like design. Better to start a fresh session with a skeptical evaluator prompt and a rubric, and ask again.

BEN: Exactly. Question two. You're writing a rubric for that evaluator. What should each criterion have, and why not just pass or fail?

PAUSE: 5

MAYA: A weight and a hard threshold, plus a description of what good and bad look like. Scores show where the work is weak, weights tell both agents what matters most, and pass or fail can let through work that matches the spec but is poor.

BEN: Perfect. And last one. Your harness has a step you added months ago, and you've just switched to a newer model. How do you decide whether to keep it?

PAUSE: 5

MAYA: Ablate it. Remove just that one component, run a realistic task, and compare. Keep it only if removing it makes the results worse.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the self-review one: same session rates its own UI, then a fresh skeptical evaluator with my rubric, and compare the scores and the issues found.

BEN: And try ablating your own setup with a short log, writing a four-criterion rubric, rewriting a detailed plan one level up, and watching for premature "done" on a long task. Next time, in chapter twenty-seven, Running agents in parallel: isolation with worktrees and sandboxes, handoffs, and orchestration when several sessions share one codebase.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-seven.
