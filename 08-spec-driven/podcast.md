<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 08: Spec-driven development

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm still the one learning here.

BEN: And I'm Ben. Today is chapter eight: spec-driven development. Agreeing on what to build before the agent builds it.

MAYA: Last time, in chapter seven, the big idea was verification. Give the agent a check it can run, so it stops when the check passes, not when the work just looks done.

BEN: Right. And today we move one step earlier. Before you can check that something is done, you and the agent have to agree on what "done" means. Here's the one sentence. Instead of steering an agent through open-ended prompts, write down what you want and why, turn that into a technical plan, break the plan into small reviewable tasks, and only then implement. And check each of those documents before moving on.

MAYA: [curious] Okay, but honestly, I just prompt until it works. And it usually does work, eventually.

BEN: That has a name: vibe coding. And it's fine for prototypes. But it's unreliable for serious work, or for existing codebases. GitHub's framing is nice here. Quote: "The issue isn't the coding agent's coding ability, but our approach." Models "are exceptional at pattern completion, but not at mind reading."

MAYA: [laughs] Not at mind reading. Fair.

BEN: GitHub's announcement of their toolkit, Spec Kit, names the problem. Vague prompts force the model to guess at requirements you never stated. And over a long task, those guesses compound.

MAYA: That sounds like Karpathy from chapter six. Wrong assumptions, and the agent just runs with them.

BEN: Exactly that. Agents "make wrong assumptions on your behalf and just run along with them without checking."

MAYA: So how do human teams avoid this?

BEN: With specifications. What the system must do, for whom, under what conditions, written down before implementation. Spec-driven development, or {SDD|S D D}, applies that discipline to agents. You act as the stakeholder and the reviewer. The agent drafts and implements. And both of you work from the same documents, not from chat history.

MAYA: [curious] And what exactly counts as a spec? Is any document a spec?

BEN: The chapter borrows a definition from Birgitta Böckeler, who we'll meet later. A spec is "a structured, behavior-oriented artifact", "written in natural language", describing what a feature must do and why.

MAYA: Behaviour-oriented. So it's about what the software does, not how the code looks.

BEN: Exactly.

MAYA: So the documents are the shared ground, not the conversation.

BEN: Right. And Spec Kit's methodology document takes the idea even further. It calls it a "power inversion". Quote: "Specifications don't serve code, code serves specifications." Maintaining software becomes "evolving specifications".

MAYA: [thoughtful] Hmm. That's a pretty radical claim. Code just becomes a by-product?

BEN: That's the strong version. And the chapter says you don't have to accept it to use the practical one: agree on the spec first.

MAYA: Okay. So what's the workflow?

BEN: GitHub's blog describes four phases. Specify, plan, tasks, implement. And each has a checkpoint, where, quote, "you don't move to the next one until the current task is fully validated."

MAYA: Walk me through them.

BEN: Specify answers: what are we building, and why? You provide goals, users, journeys and success criteria. The agent produces a {spec.md|spec dot M D} file, with user stories, acceptance criteria and edge cases.

MAYA: And plan?

BEN: Plan answers: how will it be built? You provide the stack, the architecture and the constraints. The agent produces a {plan.md|plan dot M D}, plus research, a data model, contracts and a quickstart.

MAYA: Then tasks.

BEN: Tasks answers: what are the small, ordered steps? You review the breakdown. The agent produces a {tasks.md|tasks dot M D}, with tasks ordered by their dependencies, and the ones that can run in parallel marked with a {[P]|P}. And finally, implement. You review focused changes, and the agent writes code and tests.

MAYA: Let me check I've got it. Four documents in a row, and I review each one before the agent moves on. I'm the reviewer at every gate.

BEN: That's right. You don't let the agent run from a vague idea straight to code.

MAYA: Why not just put all of that into one big document?

BEN: Because the separation is the point. The blog calls it, quote, "separating the stable 'what' from the flexible 'how'." The spec stays at the level of user needs. Technology choices only come in with the plan. So you can review each one on its own terms, and you can change the stack without rewriting the requirements.

MAYA: Okay, so the spec is the what and why, the plan is the how. Got it. How does Spec Kit actually run?

BEN: As of October 2026, it runs as commands inside your coding agent's chat. Recent versions call them skills. A small command-line tool sets it up: you install the specify CLI with {uv|U V}, then run {specify init|specify init}. And the command reference lists nine commands.

MAYA: Nine! Okay, go.

BEN: [laughs] Quickly. Constitution sets project principles that later phases are checked against. You run it once per project. Specify creates or updates the feature spec from a description of what and why. Clarify asks, quote, "up to five targeted questions" about underspecified areas, and records the answers in the spec. You can run it repeatedly.

MAYA: Next?

BEN: Plan generates the technical plan and design artifacts. Checklist generates, quote, "unit tests for your requirements." Tasks generates the dependency-ordered task list. Analyze is a read-only consistency check across spec, plan and tasks. Implement executes the tasks phase by phase, following the dependencies and parallel markers.

MAYA: And the last one?

BEN: Converge. It checks the code against the documents, and appends any missed work as new tasks. You repeat implement and converge until it reports "Converged."

MAYA: Is there a short version of all that?

BEN: The README gives one. "Constitution once per project; specify, plan, tasks, implement, converge per feature." And it tells you to run each skill "one at a time, and review the result before continuing."

MAYA: Do I type them exactly like that in every agent?

BEN: No. The syntax varies by agent. It might be {/speckit.plan|slash spec kit dot plan}, or {/speckit-plan|slash spec kit dash plan}, or something else. Spec Kit supports many agents. So check the current docs before you rely on exact names.

MAYA: Okay. The chapter said the most transferable idea is the templates. Why templates?

BEN: Spec Kit describes templates as "sophisticated prompts that constrain the LLM's output in productive ways." And several of those constraints work with any agent workflow. First: keep the spec about what and why. The spec template says, "Focus on WHAT users need and WHY", and "Avoid HOW to implement", meaning no tech stack, no APIs, no code structure.

MAYA: Give me an example.

BEN: When the model wants to jump to "implement using React with Redux", the template keeps it on "users need real-time updates of their data."

MAYA: Okay. Second?

BEN: Mark uncertainty instead of guessing. This is my favourite. The templates require, quote, "Mark all ambiguities: Use {[NEEDS CLARIFICATION: specific question]|needs clarification, colon, specific question}", and, "Don't guess."

MAYA: What does that look like in practice?

BEN: The example: rather than assuming a login system uses email and password, the model writes a marker. Needs clarification: auth method not specified. Email and password, single sign-on, or {OAuth|oh auth}? And then a human resolves each marker, either directly, or through the clarify command.

MAYA: [thoughtful] That's really the opposite of what Karpathy complained about. Instead of hiding confusion, it has to write it down.

BEN: Exactly. Third: self-review with checklists. The spec template has a checklist described as "unit tests" for the spec. Items like, no clarification markers remain, "Requirements are testable and unambiguous", and "Success criteria are measurable."

MAYA: Fourth?

BEN: Gate complexity. The plan template has gates before implementation, tied to the constitution. A simplicity gate asks, "No future-proofing?" An anti-abstraction gate asks, "Using framework directly?" If a gate fails, the model has to justify the complexity in writing.

MAYA: That's "simplicity first" from chapter six again.

BEN: Same habit, same target. Fifth: tests before code. The plan template fixes the order files get created. Contracts first, then tests, going from contract to integration to end-to-end to unit tests. And only then, quote, "source files to make tests pass."

MAYA: So the tests become the check from chapter seven.

BEN: Right, the checkable definition of done. And sixth: no speculative features. Quote: "No speculative or 'might need' features." Every feature has to trace back to a user story with acceptance criteria.

MAYA: Let me sum up the templates. Keep the spec free of tech, mark every unknown, check the spec with a checklist, gate complexity, tests before code, and nothing speculative.

BEN: That's all six. Nicely done.

MAYA: The chapter also says some steps shouldn't be done by the model at all?

BEN: Yes. Spec Kit moves deterministic steps out of the prompt and into scripts. When you run specify, it scans existing specs for the next feature number, creates a branch named after the description, and sets up a folder for the feature from the template. The repository ships scripts for this, like create new feature, setup plan, and check prerequisites.

MAYA: What's in that feature folder?

BEN: In the methodology's example, there's a specs folder, and inside it one folder per feature, like {003-chat-system|zero zero three chat system}. That holds the spec, the plan, a research file for technical options, a data model, a contracts folder for API and event definitions, a quickstart with the key validation scenarios, and the task list. Exact files vary by version.

MAYA: [curious] But why scripts? The model could easily create a branch.

BEN: Could, yes. But branch names, numbering and checking whether files exist have exactly one right answer. A script gets them right every time. A model asked to do them might not. So use the model where judgement is needed.

MAYA: Scripts for bookkeeping, the model for judgement.

BEN: Exactly. And there's a bonus. Each feature gets its own versioned folder of specs that can be reviewed and merged like code.

MAYA: Is Spec Kit the only tool doing this?

BEN: No. AWS's Kiro has specs built into its IDE and CLI, with three files. {requirements.md|requirements dot M D}, with user stories and acceptance criteria. {design.md|design dot M D}, with the architecture, sequence diagrams and implementation considerations. And {tasks.md|tasks dot M D}, with discrete, trackable tasks. So the workflow runs requirements, design, tasks.

MAYA: Anything Kiro does differently?

BEN: Kiro shows task status as the work proceeds. It can run all tasks in waves, ordered by dependency, with the tasks in one wave running at the same time. And it has separate bugfix specs, which record current, expected and unchanged behaviour.

MAYA: So two different tools, same basic shape.

BEN: That's the chapter's point. A "what" document, a "how" document and a task list, each reviewed before the next. Which suggests the pattern matters more than any one tool.

MAYA: Okay, I have to push back now. This sounds like a lot of paperwork.

BEN: [laughs] You're not alone. Birgitta Böckeler from Thoughtworks, the one I mentioned earlier, tried Kiro, Spec Kit and a third tool, Tessl, and wrote up the trade-offs. First, she describes three levels of commitment. Spec-first: you write a spec and use it for the task. Spec-anchored: you keep the spec to guide later maintenance. And spec-as-source: humans edit only the spec, never the code. Most tools today are mainly spec-first.

MAYA: And the paperwork problem?

BEN: She calls it the review burden. She found Spec Kit's documents, quote, "repetitive, both with each other, and with the code that already existed." And she wrote, "I'd rather review code than all these markdown files." GitHub's own post lists "reading walls of text" as an open problem.

MAYA: [laughs] Walls of text. Yes. Anything else?

BEN: Instructions still slip. She "frequently saw the agent ultimately not follow all the instructions." And at other times it went "way overboard", following one article of the constitution too eagerly.

MAYA: And for small changes?

BEN: Size mismatch. For a small bug, the full workflow "was like using a sledgehammer to crack a nut." For larger features, Anthropic's Claude Code guide offers a lighter version. Have the agent interview you, write a self-contained {SPEC.md|SPEC dot M D}, then implement it in a fresh session.

MAYA: Oh, that's much lighter.

BEN: And finally, history. She compares spec-as-source to model-driven development, which, quote, "never took off for business applications." And she warns of "Inflexibility and non-determinism."

MAYA: So is her verdict negative?

BEN: Balanced. She says "the general principle of spec-first is definitely valuable in many situations." But the term "isn't very well defined yet." And GitHub itself calls Spec Kit "our experiment". It suggests spec-driven development pays off most for feature work in existing systems, where the plan can encode architectural constraints.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Write down what and why before how, in separate documents. Make ambiguity explicit, and have a human resolve it, instead of letting the model guess. Gate each step with a review, and put tests before code, so done is checkable. Use scripts and templates for deterministic steps, and the model for judgement. And treat specs as versioned files, not chat history, and size the process to the change.

MAYA: And what will date?

BEN: As of October 2026: Spec Kit's command names and syntax. The constitution, clarify, analyze and converge steps. Extensions such as bug fixing and idea assessment. The list of supported agents. The install method, and the version, one point two point zero, released in October 2026. Kiro's spec types and task waves. And repository star counts.

MAYA: Let me try to sum up, and you correct me. Vibe coding is fine for prototypes, but the model can't read my mind, and its guesses compound. So in spec-driven development, I write a spec with what and why, then a plan with how, then a task list, then implement. And I review each step before the next.

BEN: Good so far.

MAYA: Templates keep the spec free of tech choices, make the model mark ambiguity instead of guessing, gate complexity, and put tests before code. Scripts handle bookkeeping like numbering and branches. And the costs are real: lots of text to review, instructions that still slip, and overkill for small fixes.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your spec for a login feature doesn't say which sign-in method to use. What should the agent do?

PAUSE: 5

MAYA: Not guess. It writes a needs-clarification marker, like auth method not specified, email and password, single sign-on, or {OAuth|oh auth}? And a human resolves it before planning.

BEN: Perfect. Question two. Your spec says "implement using React with Redux". What's wrong with that?

PAUSE: 5

MAYA: That's how, not what. It belongs in the plan. The spec should say what users need, like real-time updates of their data. Then I can change the stack without rewriting the requirements.

BEN: Exactly. And last one. Why does Spec Kit create branch names and feature numbers with a script instead of asking the model?

PAUSE: 5

MAYA: Because they have exactly one right answer. A script gets them right every time, and a model might not. The model is for judgement.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has six exercises. I'm starting with the first: write a technology-free spec for a small feature, and mark every open question with a needs-clarification marker.

BEN: Then try an ambiguity pass capped at five questions, a separate plan file, a task list with parallel markers, converging by hand in a fresh session, and scripting one bookkeeping step. Next time, in chapter nine, we start a new track on tools and protocols, with {MCP|M C P} and {AGENTS.md|AGENTS dot M D}. Why shared standards matter.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter nine.
