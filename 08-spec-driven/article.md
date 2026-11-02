# Spec-Driven Development: Agree on What to Build Before the Agent Builds It

> **Core idea in one sentence:** Instead of steering an agent through open-ended prompts, write down what you want and why, turn that into a technical plan, break the plan into small reviewable tasks, and only then implement, checking each artifact before moving on so you and the agent agree on what "done" means.

## TL;DR

- Prompting until something works ("vibe coding") is fine for prototypes but unreliable for serious or existing codebases. GitHub's framing: "The issue isn't the coding agent's coding ability, but our approach." Models "are exceptional at pattern completion, but not at mind reading" [3].
- **Spec-driven development (SDD)** makes a written specification the anchor for planning, tasks and code [2][3].
- The core sequence is **specify** (what and why) → **plan** (how) → **tasks** (small steps) → **implement**, with a human checkpoint at each step [3]. As of 2026-10, GitHub's Spec Kit adds a project **constitution** up front and a **converge** step at the end [1][4].
- Templates force the agent to **mark ambiguity** with `[NEEDS CLARIFICATION]` instead of guessing, keep the spec free of implementation details, and put tests before code [2].
- Deterministic steps such as numbering, branching and prerequisite checks run as **scripts**, not as instructions the model might skip [1][2].
- SDD has real costs: verbose artifacts, review fatigue, and overkill for small changes. Thoughtworks' Birgitta Böckeler asks whether these tools fit existing workflows onto agents too literally [6].

## Why prompting alone fails

GitHub's announcement of Spec Kit names the problem: vague prompts force the model to guess at requirements you never stated, and the guesses compound over a long task [3]. Karpathy describes the same failure from daily use. Agents "make wrong assumptions on your behalf and just run along with them without checking" (topic 06).

Human teams handle this with specifications: what the system must do, for whom, under what conditions, written down before implementation. SDD applies that discipline to agents. You act as the stakeholder and reviewer, the agent drafts and implements, and both work from the same documents rather than from chat history.

Spec Kit's methodology document takes the idea further. It calls this a "power inversion": "Specifications don't serve code—code serves specifications." Maintaining software becomes "evolving specifications" [2]. You don't have to accept that strong version to use the practical one: agree on the spec first.

## The workflow

GitHub's blog describes four phases, each with a checkpoint where "you don't move to the next one until the current task is fully validated" [3]:

| Phase | Question | You provide | Agent produces |
|---|---|---|---|
| **Specify** | What are we building, and why? | Goals, users, journeys, success criteria | `spec.md`: user stories, acceptance criteria, edge cases |
| **Plan** | How will it be built? | Stack, architecture, constraints | `plan.md` plus research, data model, contracts, quickstart |
| **Tasks** | What are the small, ordered steps? | Review of the breakdown | `tasks.md`: dependency-ordered tasks, parallel ones marked `[P]` |
| **Implement** | Build it | Review of focused changes | Code and tests |

The separation is the point. The blog calls it "separating the stable 'what' from the flexible 'how'" [3]. The spec stays at the level of user needs. Technology choices enter only in the plan, so you can review each on its own terms and change the stack without rewriting requirements.

### Spec Kit's commands (as of 2026-10)

Spec Kit runs as agent commands (recent versions call them skills) invoked in your coding agent's chat, set up by a small CLI (`uv tool install specify-cli`, then `specify init`) [1]. The command reference lists [4]:

- `/speckit.constitution`: set project principles that later phases are checked against. Run once per project.
- `/speckit.specify`: create or update the feature spec from a description of what and why.
- `/speckit.clarify`: ask "up to five targeted questions" about underspecified areas and record the answers in the spec. Can be run repeatedly.
- `/speckit.plan`: generate the technical plan and design artifacts.
- `/speckit.checklist`: generate "unit tests for your requirements."
- `/speckit.tasks`: generate a dependency-ordered `tasks.md`.
- `/speckit.analyze`: a read-only consistency check across spec, plan and tasks.
- `/speckit.implement`: execute the tasks phase by phase, following dependencies and parallel markers.
- `/speckit.converge`: check the code against the artifacts and append any missed work as new tasks. Repeat implement → converge until it reports "Converged."

The README summarises: "Constitution once per project; specify → plan → tasks → implement → converge per feature," and tells you to run each skill "one at a time, and review the result before continuing" [1]. Command syntax varies by agent (`/speckit.plan`, `/speckit-plan` and others), and Spec Kit supports many agents through integration keys [1][4]. Check the current docs before relying on exact names.

## How templates constrain the model

Spec Kit's most transferable idea is that templates work as "sophisticated prompts that constrain the LLM's output in productive ways" [2]. Several of these constraints apply to any agent workflow.

**Keep the spec about what and why.** The spec template instructs: "Focus on WHAT users need and WHY" and "Avoid HOW to implement (no tech stack, APIs, code structure)." When the model would jump to "implement using React with Redux," the template keeps it on "users need real-time updates of their data" [2].

**Mark uncertainty instead of guessing.** The templates require: "Mark all ambiguities: Use [NEEDS CLARIFICATION: specific question]" and "Don't guess." The example given: rather than assuming a login system uses email and password, the model writes `[NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]` [2]. A human resolves each marker, directly or through `/speckit.clarify` [4].

**Self-review with checklists.** The spec template includes a checklist, described as "unit tests" for the specification, with items such as "No [NEEDS CLARIFICATION] markers remain," "Requirements are testable and unambiguous" and "Success criteria are measurable" [2].

**Gate complexity.** The plan template has pre-implementation gates tied to the constitution, such as a simplicity gate ("No future-proofing?") and an anti-abstraction gate ("Using framework directly?"). If a gate fails, the model must justify the complexity in writing [2]. This targets the same over-engineering habit topic 06 addresses with "simplicity first."

**Tests before code.** The plan template fixes a file-creation order: contracts first, then tests (contract → integration → end-to-end → unit), then "source files to make tests pass" [2]. The tests become the checkable definition of done (topic 07).

**No speculative features.** "No speculative or 'might need' features." Every feature must trace back to a user story with acceptance criteria [2].

## Scripts for bookkeeping, the model for judgement

Spec Kit moves deterministic steps out of the prompt and into scripts. When you run specify, it scans existing specs for the next feature number, creates a branch with a name derived from the description, and sets up a `specs/<branch-name>/` directory from the template [2]. The repository ships scripts such as `create-new-feature`, `setup-plan` and `check-prerequisites` for these steps [1].

```
specs/
  003-chat-system/
    spec.md         # what and why
    plan.md         # how
    research.md     # technical options
    data-model.md
    contracts/      # API and event definitions
    quickstart.md   # key validation scenarios
    tasks.md        # ordered, [P] = parallel-safe
```
*(Layout from Spec Kit's methodology example [2]; exact files vary by version.)*

Branch names, numbering and file-existence checks have exactly one right answer. A script gets them right every time, while a model asked to do them might not. Use the model where judgement is needed. Each feature also gets its own versioned folder of specs that can be reviewed and merged like code.

## A second implementation: Kiro

Spec Kit isn't the only tool. AWS's Kiro has "specs" built into its IDE and CLI, with three artifacts: `requirements.md` (user stories and acceptance criteria), `design.md` (architecture, sequence diagrams, implementation considerations) and `tasks.md` (discrete, trackable tasks) [5]. The workflow runs requirements → design → tasks. As of 2026-10, Kiro shows task status as work proceeds and can run all tasks in dependency-ordered waves, with tasks in a wave running concurrently. It also has separate bugfix specs that record current, expected and unchanged behaviour [5].

The two tools converge on the same shape: a "what" document, a "how" document, and a task list, each reviewed before the next. That suggests the pattern matters more than any one tool.

## Costs and open questions

Böckeler tried Kiro, Spec Kit and Tessl and wrote up the trade-offs [6]:

- **Three levels of commitment.** *Spec-first*: write a spec, use it for the task. *Spec-anchored*: keep the spec to guide later maintenance. *Spec-as-source*: humans edit only the spec, never the code. Most tools today are mainly spec-first [6].
- **Review burden.** She found Spec Kit's artifacts "repetitive, both with each other, and with the code that already existed," and wrote: "I'd rather review code than all these markdown files." GitHub's own post lists "reading walls of text" as an open problem [3][6].
- **Instructions still slip.** She "frequently saw the agent ultimately not follow all the instructions." At other times it went "way overboard" following a constitution article too eagerly [6].
- **Size mismatch.** For a small bug, a full workflow "was like using a sledgehammer to crack a nut" [6]. Anthropic's Claude Code guide offers a lighter version for larger features: have the agent interview you, write a self-contained `SPEC.md`, then implement it in a fresh session [7].
- **History.** She compares spec-as-source to model-driven development, which "never took off for business applications," and warns of "Inflexibility *and* non-determinism" [6].

Her conclusion is balanced: "the general principle of spec-first is definitely valuable in many situations," but the term "isn't very well defined yet" [6]. GitHub calls Spec Kit "our experiment" and suggests SDD pays off most for feature work in existing systems, where the plan can encode architectural constraints [3].

## What lasts vs. what will date

- **Durable principles:**
  - Write down *what* and *why* before *how*, in separate documents.
  - Make ambiguity explicit and have a human resolve it, rather than letting the model guess.
  - Gate each step with a review, and put tests before code so "done" is checkable.
  - Use scripts and templates for deterministic steps. Use the model for judgement.
  - Treat specs as versioned artifacts, not chat history, and size the process to the change.
- **Time-sensitive details (as of 2026-10):** Spec Kit's command names and syntax (`/speckit.*` vs. `/speckit-*`), the constitution, clarify, analyze and converge steps, extensions such as bug fixing and idea assessment, the supported-agent list, the install method (v1.2.0 released 2026-10-13), Kiro's spec types and task waves, and repository star counts.

## Try it yourself

1. **Write a technology-free spec** for a small feature: users, journeys, edge cases, acceptance criteria. Mark every open question `[NEEDS CLARIFICATION: ...]`.
2. **Ambiguity pass.** Ask an agent only to list ambiguities in your spec, capped at five questions. Resolve them before any planning.
3. **Separate the plan.** Write the plan in its own file. Check that you could change the stack without editing the spec.
4. **Tasks with markers.** Have the agent produce a dependency-ordered task list, tests first, with `[P]` on independent tasks. Check whether the `[P]` tasks really are independent.
5. **Converge by hand.** After implementing, ask a fresh session to compare code against spec and tasks and list only missed requirements.
6. **Script one bookkeeping step** in your current workflow (branch naming, folder setup, prerequisite check) instead of prompting for it.

## Key terms

- **Vibe coding:** building by loose prompting and reacting to output, without a written specification.
- **Spec-driven development (SDD):** a workflow in which an agreed written specification drives planning, tasks, tests and code.
- **Specification (spec):** "a structured, behavior-oriented artifact... written in natural language" describing what a feature must do and why [6].
- **Constitution:** project-wide principles that later phases are checked against.
- **Clarification marker:** an explicit `[NEEDS CLARIFICATION]` flag where information is missing, resolved by a human.
- **Implementation plan:** the technical "how": stack, architecture, data model, contracts.
- **Convergence:** checking the implementation against the spec, plan and tasks and adding missed work as tasks.
- **Spec-first / spec-anchored / spec-as-source:** increasing levels of how long the spec lives and how much it replaces code as the edited artifact.

## Sources

1. [github/spec-kit README](https://github.com/github/spec-kit) (GitHub, accessed 2026-10-14; v1.2.0 released 2026-10-13, MIT). The current toolkit: install, the constitution → specify → plan → tasks → implement → converge sequence, review-each-step guidance, extensions, and the setup scripts in the repository.
2. [Specification-Driven Development (SDD), spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md) (GitHub Spec Kit, accessed 2026-10-14). The methodology: the "power inversion", automatic numbering and branching, how templates constrain the model, `[NEEDS CLARIFICATION]` markers, complexity gates, test-first ordering, and the example file layout.
3. [Spec-driven development with AI: Get started with a new open source toolkit](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/) (Den Delimarsky, GitHub Blog, 2025-09-02). Why vibe coding falls short, the four phases with checkpoints, stable "what" vs. flexible "how", where SDD works best, and open problems.
4. [Spec Kit command reference: Agentic SDD](https://github.github.io/spec-kit/reference/agentic-sdd.html) (GitHub Spec Kit docs, accessed 2026-10-14). What each command reads and writes, clarify's five-question cap, implement's phase-by-phase execution, and the append-only converge loop.
5. [Kiro: Specs](https://kiro.dev/docs/specs/) (Kiro / AWS, accessed 2026-10-14). A second implementation: requirements, design and tasks files, feature vs. bugfix specs, and wave-based task execution.
6. [Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) (Birgitta Böckeler, martinfowler.com, 2025-10-15). Definition of a spec, the spec-first/anchored/as-source levels, hands-on critique of verbosity and instruction-following, and the model-driven-development comparison.
7. [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (Anthropic, accessed 2026-10-14). The lightweight alternative: an interview that produces a self-contained `SPEC.md`, executed in a fresh session with an end-to-end verification step.
