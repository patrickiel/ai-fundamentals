<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 25: Graphs and verification

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Last time, in chapter twenty-four, we looked at the advisor strategy: a cheaper model doing the work, and a stronger one consulted at the decisive points. Today, chapter twenty-five, we zoom out. From one agent in a loop, to many agents arranged as a graph. And why every node in that graph needs its own check.

MAYA: Graphs. As in, boxes and arrows?

BEN: [laughs] Exactly that. Here's the one sentence for today. Splitting a task across several agents arranged as a graph buys speed, and lets you pick a model per step. But an unchecked error in one node flows into everything downstream, so each node needs its own check, run by a reviewer that did not do the work.

MAYA: Okay. Start with the loop, then. We had a whole chapter on loops.

BEN: Chapter twenty-two, yes. The Claude Code team describes an agent as a loop: gather context, take action, verify results. And without that verify step, the agent stops when the work "looks done", and you become the check.

MAYA: [sighs] I've definitely been the check.

BEN: We all have. But a loop has another limit. It's sequential. Step two waits for step one, even when it doesn't depend on it. A graph removes that wait. Each part of the task gets its own agent, and the parts are connected so each result reaches the agent that needs it.

MAYA: Are there standard shapes for these graphs?

BEN: "Building effective agents" names five. Prompt chaining is steps in sequence, with a programmatic "gate" checking intermediate output. It fits when the task splits cleanly into fixed subtasks. Routing classifies the input and sends it to a specialised follow-up, for example easy questions to a small model.

MAYA: We met routing last chapter.

BEN: We did. Third, parallelization: independent subtasks at once, called sectioning, or the same task several times, called voting. You want speed, or several perspectives for more confidence. Fourth, orchestrator-workers. A central model splits the task, delegates and synthesises. That fits when you can't predict the subtasks in advance, like multi-file code changes.

MAYA: And fifth?

BEN: Evaluator-optimizer. One model generates, another evaluates and gives feedback, in a loop. It fits when criteria are clear and iteration measurably helps.

MAYA: So graphs are always better than a single loop?

BEN: No, and the same paper warns about that. These systems "often trade latency and cost for better task performance". Autonomy brings "the potential for compounding errors". And you should start with the simplest design, and add structure only when it demonstrably helps.

MAYA: Okay. What's a graph actually made of?

BEN: LangGraph, one widely used graph framework, makes it explicit. There are three parts. A node is one unit of work: a model call, a tool call, or a whole agent. In LangGraph it's a function that reads shared state and returns an update.

MAYA: And the arrows?

BEN: Edges. The rule for what runs next. LangGraph has {add_edge|add edge} for a fixed transition, and {add_conditional_edges|add conditional edges} for a decision, like "pass or retry". And state is what flows between nodes. A typed object, with reducers that merge updates from parallel branches.

MAYA: [curious] Can you give me a concrete example?

BEN: In LangGraph's own examples, a parallel workflow fans out from the start to three writer nodes, and all three feed one aggregator node. And a gate function sends a draft either to the end, or back for improvement.

MAYA: So the gate is just a conditional edge.

BEN: Right. The chapter then picks out two shapes worth knowing. The first is the diamond. One task splits into parallel workers, say A, B and C, and one synthesiser merges their results into a single answer. That's orchestrator-workers.

MAYA: Fan out, then synthesise. And the second?

BEN: Fan-in at a barrier. The same problem goes to several agents, each with a different lens. A barrier holds everything until every branch has reported. Only then do fixes start. That's parallelization, by voting or sectioning.

MAYA: Why wait? Why not let each reviewer fix things as soon as it finds something?

BEN: That's exactly what the barrier prevents. It stops one reviewer's fix from landing before the other angles have been heard. Claude Code's dynamic workflows expose the barrier directly. A function called {parallel()|parallel} "runs a set of agent tasks at the same time and waits for all of them".

MAYA: What's a dynamic workflow?

BEN: As of October 2026, it's a graph in practice. A script Claude writes that orchestrates many subagents, and can have, quote, "independent agents adversarially review each other's findings before they're reported".

MAYA: Okay. What does going wide cost me?

BEN: Two things. Tokens, and traceability. Putting a smaller model on easy stages lowers the cost per agent, not the total. Claude Code's documentation says a workflow "can use meaningfully more tokens than working through the same task in conversation". It flags runs over twenty-five agents, or one and a half million projected tokens, and recommends trying a small slice first.

MAYA: [thoughtful] So the cheap models make each agent cheaper, but there are more agents.

BEN: That's it. And traceability. When a graph returns a wrong answer, you see only the final output. A small error in an early node has been built on by every node after it.

MAYA: So I could just check the final answer really carefully.

BEN: That's the trap. Checking once at the end tells you that something failed, not where. Verification has to sit inside the graph.

MAYA: Okay, so what does verification look like?

BEN: The Claude Code team's post on verification loops starts from deterministic signals the agent can read. Type checkers, linters, tests, runtime errors. Writing the exact build and test commands in {CLAUDE.md|CLAUDE dot M D} saves the agent from rediscovering them.

MAYA: And what's built in?

BEN: The post lists several options. The {/verify|slash verify} skill, which builds, runs and observes the changed app. Code Review, a managed multi-agent PR review, in research preview. GitHub Actions jobs that run a verification skill on each PR. Spec validation. And rubric grading by a separate grader in Claude Managed Agents, which is in beta.

MAYA: If I have tests, isn't that enough?

BEN: Tests catch breakage, not intent. Everything Claude can't infer becomes a manual check you keep repeating. And those repeated corrections are exactly what to turn into verification.

MAYA: Example?

BEN: The post's example is a project rule. "Reject any migration that drops a column without a backfill step." You can generate a skill like that with the skill-creator plugin, or write a {SKILL.md|skill dot M D} by hand.

MAYA: Oh, I like that. Every time I correct the agent on the same thing, that's a check waiting to be written.

BEN: Exactly. Now, in a graph, the most valuable check is the requirements review. You compare a node's output with what was originally asked.

MAYA: Why that one in particular?

BEN: Because each node sees only its slice. So it can't notice that it has drifted from the whole. Claude Code's best-practices guide gives a template. Have a reviewer check the diff against {PLAN.md|plan dot M D}, confirm every requirement is implemented and tested, and report "gaps, not style preferences".

MAYA: Where should these checks run?

BEN: The verification-loops post sorts them into four modes. Standalone: you invoke it deliberately. That's good for cross-cutting checks, like a pre-commit security scan or an accessibility audit. But if you find yourself running it after every change, embed or chain it instead.

MAYA: Embedded?

BEN: Inside the skill that produces the work. For example, a lint step at the end of a scaffolding skill. Good for checks tied to one kind of output. But it only works for skills you can edit.

MAYA: And if I can't edit the skill?

BEN: Then chained. One skill calls another when it finishes, and a wrapper skill can add checks to skills you can't modify. Good for several review angles in sequence. And the fourth mode: on every PR. The stable chain runs in CI, as a team-wide gate, once the chain has stopped changing.

MAYA: Does the Claude Code team chain things themselves?

BEN: They do. They chain {/code-review|slash code review} for bugs, {/simplify|slash simplify} for cleanup, {/verify|slash verify} for end-to-end behaviour, and a custom {/design|slash design} skill that checks UI changes against a {DESIGN.md|design dot M D} file. One check per angle, each measured on its own terms.

MAYA: Four reviews sounds expensive.

BEN: The post says so. Chained loops "can increase token spend", so test a chain before deploying it widely.

MAYA: And if I want the angles to run in parallel instead of in sequence?

BEN: Put them under a fan-in. One agent per review angle, a barrier, then one merge step that ranks and de-duplicates the findings for the fixing agents. Dynamic workflows document exactly that pattern. "Review every file changed in this PR", then "merge the per-file findings into one ranked summary".

MAYA: What about visual stuff? UI changes?

BEN: UI verification usually means driving a browser and taking screenshots. That's slow when it's repeated inside a graph. Chrome has a lighter headless binary called {chrome-headless-shell|chrome headless shell}, suited to automated screenshots. Full Chrome in headless mode is the more faithful choice for high-accuracy end-to-end testing.

MAYA: So, light tool for the frequent checks, faithful one for the final pass.

BEN: That's the advice.

MAYA: Okay. Now the part from the core sentence. Why can't the agent that did the work just check it?

BEN: Because the builder is a poor judge of its own work. Anthropic's harness experiments found that "agents reliably skew positive when grading their own work". And tuning a separate, skeptical evaluator is "far more tractable" than making the generator self-critical.

MAYA: [laughs] Like marking your own exam.

BEN: [laughs] As an analogy, yes. Claude Code's guide says the same about sessions. "A fresh context improves code review since Claude won't be biased toward code it just wrote." It recommends a Writer and Reviewer pair of sessions, or a reviewer subagent that "sees only the diff and the criteria you give it".

MAYA: How do I get a fresh context easily?

BEN: A fresh non-interactive session. {claude -p|Claude dash P}, followed by a prompt, runs once with no prior conversation. And you can pipe in only the diff and the requirements.

MAYA: Truly fresh? Nothing loaded?

BEN: Not quite. By default it still loads the project's {CLAUDE.md|CLAUDE dot M D}, hooks and skills. The {--bare|dash dash bare} flag skips them, for a reproducible run.

MAYA: So a separate reviewer, and I'm safe.

BEN: [thoughtful] Not so fast. A reviewer can be wrong in both directions. Claude Code's guide warns, "A reviewer prompted to find gaps will usually report some, even when the work is sound." And chasing every finding leads to over-engineering.

MAYA: Huh. Because I asked it to find problems, it finds problems.

BEN: Right. And Anthropic's harness author found the opposite failure too. Out of the box, the QA agent "tended to test superficially", and talked itself into approving flawed work.

MAYA: So too picky, or too lenient.

BEN: And in a graph, both errors multiply. False alarms send many agents to "fix" working code. Missed bugs pass through every node.

MAYA: Then how do I know if my reviewer is any good?

BEN: Judge it by whether its findings are correct, not by how many it produces. Scope it to correctness and stated requirements. Calibrate it on real outputs, which is chapter eighteen. And give the judging step a model strong enough for the task, because a weak verdict is copied into every node that relies on it.

MAYA: Let me try a mini-summary. Graphs give speed and per-step models, but cost more in total and hide where things went wrong. So: a gate after each node, a requirements review against the original ask, a fresh reviewer that didn't build it, and that reviewer scoped to findings that matter.

BEN: [warm] That's exactly right.

MAYA: What lasts, and what will date?

BEN: The durable part. Running independent work in parallel trades total cost and traceability for speed. Put a gate after each step that others build on, because an unchecked error compounds downstream. Check work against the original requirements, especially when each worker sees only a slice. Separate the reviewer from the builder, and scope the reviewer to findings that matter.

MAYA: And?

BEN: Use one focused check per angle, joined at a barrier and merged once. And start simple. Add nodes and checks only when they measurably help.

MAYA: And what will date?

BEN: As of October 2026, the names and behaviour of {/verify|slash verify}, {/simplify|slash simplify} and {/code-review|slash code review}. Code Review in research preview, and Managed Agents rubrics in beta. Dynamic workflows, with their limits and warnings. The {-p|dash P} and {--bare|dash dash bare} flags. LangGraph's API names. And how {chrome-headless-shell|chrome headless shell} is packaged.

MAYA: Let me recap the whole thing, and you correct me. A single agent is a loop: gather context, act, verify. It's sequential, so a graph splits the work into nodes and edges, and runs independent nodes in parallel. The shapes are chaining, routing, parallelization, orchestrator-workers and evaluator-optimizer. And two shapes to know are the diamond and the fan-in at a barrier.

BEN: Good so far.

MAYA: Graphs are faster, but use more tokens in total and are harder to trace. So verification goes inside the graph, as standalone, embedded, chained or per-PR checks. And the reviewer should never be the builder, and should report only what matters.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You've built a graph of agents, and you check only the final output. Why isn't that enough?

PAUSE: 5

MAYA: Because it only tells me that something failed, not where. An early error has been built on by every node after it. So I need a gate after each node that others build on.

BEN: Perfect. Question two. An agent just finished a change. How do you get an independent requirements review, and what do you tell the reviewer?

PAUSE: 5

MAYA: Run a fresh non-interactive session with {claude -p|Claude dash P}, and pipe in only the diff and the requirements. And tell it to report only gaps that affect correctness or the stated requirements, because a reviewer asked to find gaps will usually find some.

BEN: Exactly. And last one. You have three review angles running in parallel. Why put a barrier before the fixes?

PAUSE: 5

MAYA: So that one reviewer's fix doesn't land before the other angles have been heard. The barrier waits for every branch, then one merge step ranks and de-duplicates the findings for the fixing agents.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the first: draw a workflow I already hand to an agent as nodes and edges, mark which steps really depend on each other, and decide if it's a chain, a diamond, or a fan-in at a barrier.

BEN: And then add a programmatic gate to the node whose errors hurt most, run a requirements review in a fresh session, grade the grader with a small and a large model, and split one review into three narrow prompts plus a merge. Next time, in chapter twenty-six, we look at agent harness design. Planner, generator and evaluator, and why you keep doing separate from judging.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twenty-six.
