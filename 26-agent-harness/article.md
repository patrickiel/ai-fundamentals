# Agent Harness Design: Plan, Generate, Evaluate, and Keep Doing Separate from Judging

> **Core idea in one sentence:** Every component of an agent harness encodes an assumption about what the model can't do alone, so keep the parts that still earn their place (a product-level planner, a generator, and a separate, skeptical evaluator that grades against explicit criteria) and re-test the rest by removing it whenever the model changes.

## TL;DR

- An **agent harness** is the scaffolding around a model (roles, plans, checks, files, context handling) that lets it finish long tasks. Anthropic's harness research found that "every component in a harness encodes an assumption about what the model can't do on its own", and those assumptions go stale as models improve [1].
- The design that held up in Anthropic's application-building experiments was **planner → generator → evaluator**, inspired by GANs and close to the evaluator-optimizer workflow [1][3].
- **Plan the product, not the implementation.** Detailed technical specs let early errors "cascade into the downstream implementation" [1].
- **Separate the doer from the judge.** Agents "reliably skew positive when grading their own work". A standalone, skeptical evaluator is easier to tune than a self-critical generator [1][4].
- **Grade against explicit, weighted criteria** with hard thresholds, and let the evaluator use the product like a user would [1].
- **Sprint contracts, context resets and fine-grained decomposition are model-dependent.** They helped earlier models, and Anthropic removed them one at a time as newer models made them unnecessary [1][2].

## The principle: components are assumptions

Prithvi Rajasekaran's post on harness design for long-running application development opens with "Harness design is key to performance at the frontier of agentic coding" [1]. Its central claim outlasts any particular model: each harness component exists because someone assumed the model couldn't do that part alone. Those assumptions should be stress-tested, because they may be wrong and because they expire.

That makes the *method* more useful than any single conclusion. **Ablate**: remove one component, run a realistic task, compare, and keep the component only if removing it makes results worse. Rajasekaran tried radical simplification first and could not match the original harness's performance. Removing components one by one worked [1]. Repeat the exercise whenever you change models. The closing line of the post: the space of useful harnesses "doesn't shrink as models improve. Instead, it moves" [1].

This matches Anthropic's earlier general advice for agentic systems: find the simplest solution that works, and add complexity only when it demonstrably improves results [3].

## Planning: describe the product, not the implementation

The planner in Anthropic's harness takes a one-to-four-sentence prompt and expands it into a full product spec [1]. Two choices matter:

- **Ambitious scope.** The planner was prompted "to be ambitious about scope" and to look for chances to weave AI features into the product [1].
- **Product level, not code level.** It stays on "product context and high level technical design rather than detailed technical implementation" [1]. The reason given is cascading errors: if the spec fixes technical details too early, any mistake in them is built into everything downstream, and the agents lose the freedom to find a better path.

The planner earned its place even with the strongest model tested. When Rajasekaran removed it, the generator **under-scoped** its work [1].

A related lesson comes from Anthropic's earlier harness work. When a single agent is handed a large goal, it tends "to try to do too much at once", attempting to one-shot the app and running out of context mid-way. The fix there was a structured **feature list**: over 200 features, each initially marked failing, which later sessions worked through one at a time [2]. The list describes *what the user should be able to do*, not how to code it.

## Separate the doer from the judge

Asked to evaluate its own output, a model tends to praise it, "even when, to a human observer, the quality is obviously mediocre". The effect is strongest on subjective work such as design [1]. Where there is an objective check (does the endpoint return the right data?) the problem is manageable. Where there isn't, the builder happily signs off on its own average work.

Rajasekaran's finding is that "tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work" [1]. Separation alone doesn't fix leniency, since the evaluator is still a model inclined to be generous, but it gives you a component you can tune. Claude Code's documentation gives the same advice for everyday work: "A fresh context improves code review since Claude won't be biased toward code it just wrote" [4].

The same principle runs through human engineering: code review by someone other than the author, QA separate from development, editors separate from writers.

## The three roles

**Planner.** Turns the short prompt into a product-level spec and writes it to a file the other agents read [1].

**Generator.** Builds the app feature by feature from the spec (React, Vite, FastAPI and a database in Anthropic's runs), commits to git as it goes, self-checks at the end of each chunk, and then hands off to QA [1]. Anthropic's earlier harness adds habits that make multi-session work survivable: a progress file (`claude-progress.txt`), an `init.sh` that starts the dev server, descriptive commits so bad changes can be reverted, and a start-of-session routine that reads the log and runs a basic end-to-end test before touching anything new [2].

**Evaluator.** The adversary. It reads the spec to learn what "done" means, then exercises the running app through the Playwright MCP server, clicking through flows, checking API endpoints and inspecting database state [1]. It grades against criteria and returns specific feedback. In the earlier harness, giving the agent browser automation (Puppeteer MCP) so it could test "as a human user would" markedly improved results. A remaining blind spot was browser-native alert dialogs, which the agent couldn't see [2].

Tuning the evaluator took work. "Out of the box, Claude is a poor QA agent." It found real issues and then talked itself into approving anyway, and it "tended to test superficially, rather than probing edge cases". It took several rounds of reading its logs, comparing them with human judgement and revising the prompt before its grading was reasonable [1].

## Graded evaluation: tell the judge what "good" means

An evaluator with no criteria falls back on the model's defaults. For the front-end design experiments, Rajasekaran gave the generator and the evaluator the same four criteria [1]:

| Criterion | What it checks | Weight |
|---|---|---|
| **Design quality** | Do colours, typography, layout and imagery form "a coherent whole" with a distinct mood? | Higher |
| **Originality** | Custom decisions, or template layouts, library defaults and recognisable AI patterns (the post calls out purple gradients over white cards)? | Higher |
| **Craft** | Typography hierarchy, spacing, colour harmony, contrast: "a competence check rather than a creativity check" | Lower |
| **Functionality** | Can users understand the interface and find the primary actions? | Lower |

"Claude already scored well on craft and functionality by default", so weighting design quality and originality more heavily put the pressure where the model was weak and pushed it toward aesthetic risk-taking. The evaluator was calibrated with few-shot examples that showed detailed score breakdowns, and the wording of the criteria visibly shaped the output's character [1]. Design runs used 5 to 15 iterations [1].

For full applications the QA agent graded each chunk on product depth, functionality, visual design and code quality. Each criterion had a **hard threshold**: if any one fell below it, the chunk failed and went back with feedback [1].

Why scores beat a single pass/fail:

- A score per criterion shows *where* the work is weak, not just *that* it failed.
- Weights tell both agents what matters most.
- A pass/fail check against the plan can pass work that technically matches the spec but is poor.

This is the evaluator-optimizer workflow from "Building effective agents": one model generates, another evaluates and feeds back in a loop. It fits when "we have clear evaluation criteria" and iteration adds measurable value [3]. As of 2026-10, Anthropic also offers rubric grading in Claude Managed Agents (beta), where a separate grader checks outcomes against a rubric and failures loop back for rework [5].

## Model-dependent scaffolding

Two components show how "necessary" depends on the model.

### Sprint contracts

Before each chunk of work, the generator proposes what it will build and how success will be tested. The evaluator reviews the proposal, and the two iterate until they agree, all before any code is written. Agents communicated through files: one writes, the other responds in the same file or a new one. One example contract had 27 criteria [1]. The contract turns a product-level spec into concrete, checkable steps without fixing the implementation too early.

When Opus 4.6 arrived, Rajasekaran removed the sprint construct entirely, expecting the newer model to need less scaffolding. He kept the planner and the evaluator, and moved the evaluator from grading every sprint to a single pass at the end [1]. His conclusion: the evaluator "is not a fixed yes-or-no decision". It "is worth the cost when the task sits beyond what the current model does reliably solo" [1].

### Context anxiety and context resets

Some models show **context anxiety**: as they approach what they perceive as their context limit, they start wrapping up early [1]. Anthropic's earlier harness saw the related failure of a later session that would "look around, see that progress had been made, and declare the job done" [2].

Two remedies differ:

- **Compaction** summarises earlier conversation in place. It keeps continuity, but the anxiety can persist.
- A **context reset** clears the window and starts a fresh agent from a structured handoff stored in files. It gives a clean slate, at the cost of orchestration overhead, tokens and latency [1].

Claude Sonnet 4.5 showed the behaviour strongly enough that resets were essential. Opus 4.5 "largely removed that behavior", so the later harness dropped resets and relied on automatic compaction [1]. The durable lesson is the diagnostic. If your agent declares success early as its context fills, external task files plus fresh contexts are a proven fix. If it doesn't, you may not need them.

## What it costs

A harness like this is expensive, and the post reports the numbers. Building a retro game maker with Opus 4.5 took 20 minutes and $9 solo, against 6 hours and about $200 with the full harness. The solo version looked plausible, but nothing responded to input. The harness version was playable [1]. A browser-based audio workstation built with the simplified Opus 4.6 harness took about 4 hours and $125, most of it in the generator's build rounds. The QA rounds cost a few dollars each and still caught real gaps, such as clips that couldn't be moved and stubbed recording [1]. Weigh that against the value of the output. For tasks the model already handles reliably alone, the extra roles are overhead.

## What lasts vs. what will date

- **Durable principles:**
  - Every harness component is an **assumption about model weakness**. Test it by removing it, and test again when the model changes [1].
  - **Separate doing from judging.** Self-review skews positive, and a skeptical standalone evaluator is tunable [1][4].
  - **Plan outcomes, not keystrokes.** Over-specified plans spread their own errors [1].
  - **Make "good" explicit** with weighted criteria and hard thresholds, especially for subjective qualities [1].
  - Let the evaluator **use the product like a user**, and keep durable task state in files and git, outside the context window [1][2].
- **Time-sensitive details (as of 2026-10):** which components each model needs (Sonnet 4.5, Opus 4.5, Opus 4.6); the reported costs and durations; the specific stack; Playwright and Puppeteer MCP as testing tools; Managed Agents rubrics (beta).

## Try it yourself

1. **Ablate your own setup.** List every component in your current agent workflow (plan docs, checklists, subagents, hooks). For one realistic task, remove a single component and compare. Keep a short log of what each removal changed.
2. **Self-review vs. separate review.** Have an agent build a small UI, then ask *the same session* to rate it out of 10. Start a fresh session with a skeptical evaluator prompt and your rubric, and ask again. Compare the scores and the issues found.
3. **Write a rubric.** Create a four-criterion scoring file for something you care about (API design, documentation, UI). Give each a weight, a hard threshold, and a short description of a 3/10 and a 9/10 result.
4. **Rewrite a plan one level up.** Turn a detailed technical task list into product-level features with user stories. Run both versions and note where the detailed one locked in a mistake.
5. **Watch for premature "done".** On a long task, note whether the agent claims completion while work is missing. If it does, add a feature list with a `passes` field and a progress file, and restart from a fresh context.

## Key terms

- **Agent harness:** the scaffolding around a model (roles, plans, tools, checks, files, context handling) that structures how it completes long tasks.
- **Planner / generator / evaluator:** three roles that specify the product, build it, and adversarially check it.
- **Sprint contract:** an agreement between generator and evaluator, made before a chunk of work, on what "done" means and how it will be tested.
- **Context anxiety:** a model wrapping up early as it nears its perceived context limit.
- **Compaction vs. context reset:** summarising history in place vs. starting a fresh agent from a structured handoff.
- **Graded evaluation:** scoring work per weighted criterion, with thresholds, rather than a single pass/fail.
- **Ablation:** removing one component at a time to measure whether it still helps.

## Sources

1. [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Prithvi Rajasekaran, Anthropic, 2026-03-24). The planner/generator/evaluator harness, self-evaluation leniency, design criteria and weights, QA tuning, sprint contracts, context anxiety and resets, the Opus 4.6 simplification, costs, and "every component encodes an assumption".
2. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Justin Young, Anthropic, 2025-11-26). The one-shotting and early-victory failure modes, the initializer/coding agent split, feature list, progress file, git, start-of-session routine, and browser-based end-to-end testing.
3. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Erik Schluntz & Barry Zhang, Anthropic, 2024-12-19). The evaluator-optimizer workflow and when it fits, and the "simplest solution first" principle.
4. [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (Claude Code docs, read 2026-10). Fresh-context review and the Writer/Reviewer pattern.
5. [Building verification loops in Claude Code with skills](https://claude.com/blog/building-verification-loops-in-claude-code-with-skills) (Delba de Oliveira, Claude blog, 2026-07-22). Rubric grading by a separate grader agent in Claude Managed Agents (beta).
