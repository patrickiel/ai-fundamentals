<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 03: Context engineering

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter three: context engineering. How to give the model the right information at the right time.

MAYA: Last time, in chapter two, we talked about reasoning models and how to prompt them. So, is this just more prompting tips?

BEN: [laughs] It's the step beyond prompting. If you only remember one sentence from this episode, make it this one: instead of crafting one clever prompt, you design the information an AI coding agent works from, like requirements, plans, conventions and known issues, and you arrange it so a small core is always loaded and everything else is pulled in only when the current step needs it.

MAYA: Okay, that's a long sentence. Let's unpack it. First, where does the term even come from?

BEN: It spread in June 2025. On the nineteenth of June, Tobi Lütke, the CEO of Shopify, wrote that he preferred "context engineering" over prompt engineering, because it describes the core skill better. His definition was, quote, "the art of providing all the context for the task to be plausibly solvable by the LLM."

MAYA: Plausibly solvable. I like that. It's not about magic words, it's about whether the model even has what it needs.

BEN: Exactly. A week later, Andrej Karpathy added a "plus one". His argument was that people associate prompts with short task descriptions. But in real LLM applications, context engineering is, quote, "the delicate art and science of filling the context window with just the right information for the next step."

MAYA: [curious] So did those two invent it?

BEN: No, and the chapter is clear about that. Neither invented the practice. They gave a name to something builders were already doing.

MAYA: Okay. So what actually goes into "the context"? When I hear context, I think of my prompt.

BEN: Karpathy listed it out. Task descriptions and explanations, few-shot examples, retrieved data, so {RAG|rag}, related and possibly multimodal data, tools, state and history, and compaction. And he made a point about balance. Too little context hurts performance. Too much raises cost, and can also hurt performance.

MAYA: Is there a simpler checklist? That list is a lot.

BEN: Philipp Schmid's breakdown makes a good checklist of the same idea. System instructions, the user prompt, conversation state, long-term memory, retrieved information, available tools, and the required output format.

MAYA: [thoughtful] And the prompt is just one line on that list.

BEN: That's the takeaway. The prompt is one item. Karpathy's second point was about the application around the model. A serious LLM app also has to split problems into control flows, route calls to suitable models, handle verification and guardrails, and more. He rejected "ChatGPT wrapper" as a description of systems like that.

MAYA: And coding agents are that kind of application?

BEN: Right. Coding agents such as Cursor and Claude Code are examples. They supply file access, tools, task lists and rule systems, and those make context engineering practical.

MAYA: So, is there an official definition? Or is it still a vibe?

BEN: Anthropic's engineering team gave the most precise one. Prompt engineering is about, quote, "writing and organizing LLM instructions for optimal outcomes". Context engineering covers everything else that can land in the context window. System instructions, tools, external data, message history and more.

MAYA: So prompt engineering is a part of context engineering, not a different thing.

BEN: Yes. And Anthropic adds that it's iterative, because curation happens every time the model is called. It's not a one-time setup.

MAYA: Okay, here's my instinct, and you're going to tell me it's wrong. If context is so important, why not just give the model everything? The whole repo, all the docs, every conversation.

BEN: [laughs] That's the central tension of the chapter. It's tempting, and Anthropic explains why it fails. As the number of tokens grows, the model's ability to recall information from the context decreases. They call that context rot. Models have an "attention budget", and every new token draws on it.

MAYA: So every extra thing I add costs a little bit of attention.

BEN: That's it. Claude Code's guide says the same in practical terms. When the window gets full, the agent may start, quote, "forgetting" earlier instructions, or making more mistakes. Chapter four covers the evidence for all of this.

MAYA: So what's the goal instead? Not everything, but what?

BEN: Anthropic's guiding principle is to find, quote, "the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome."

MAYA: Smallest possible set. That sounds like the opposite of how I work.

BEN: [warm] It's the opposite of how most people start. In practice it means four things. Split project knowledge into separate, focused pieces. Keep anything that's always loaded as small as possible. Load the rest just in time. And include relevant external documentation, like library docs, by reference, so the agent can read it when it reaches that step.

MAYA: What does "just in time" look like in practice?

BEN: Anthropic describes agents that keep lightweight references, like file paths, stored queries and links, and pull the data in with tools when they need it. Claude Code uses a hybrid. Its {CLAUDE.md|CLAUDE dot M D} files go into context at the start, while search tools such as glob and grep fetch files on demand.

MAYA: So a small core up front, and the rest is a pointer until it's needed.

BEN: Exactly. A map, not the whole territory.

MAYA: Okay, I get the principle. What does a real workflow look like?

BEN: The chapter describes a pattern that's common among practitioners and matches the vendors' guidance. Cole Medin's public template is one worked example. A feature request file lists the feature, examples, documentation links and other considerations. One command turns that into a detailed implementation plan with validation steps. And a second command executes the plan.

MAYA: So, requirements, then plan, then build. Let's go step by step.

BEN: Step one: start with a requirements document. A {PRD|P R D}, a product requirements document, or a spec. It lists what you want built. And you add the technical constraints you've already decided, especially the tech stack.

MAYA: Do I have to write that all myself?

BEN: Not alone. Claude Code's guide suggests having the agent interview you about implementation, UI, edge cases and trade-offs, and then write the spec to a file. It says the most useful specs name the files and interfaces involved, state what's out of scope, and end with a verification step.

MAYA: [thoughtful] Out of scope. I never write down what I don't want.

BEN: Most people don't. Chapter eight covers spec-driven development in depth.

MAYA: Step two?

BEN: Have the agent turn the requirements into documents it will consult later. The chapter suggests one useful split into four files. An implementation plan, with staged tasks, subtasks, and a check for each stage. A project structure file, with the intended folder and file layout, which the agent checks before creating files. A UI and UX conventions file, with design rules and patterns for interface work. And a known issues file: errors already seen and how they were fixed, which the agent checks before debugging.

MAYA: Wait, but generating all of that will eat my context, won't it? That's exactly what we said to avoid.

BEN: Good catch. Generating these files can fill a session's context. And that's fine, because generation is a separate step. Once the files exist, the knowledge lives on disk, and later sessions don't have to hold it all.

MAYA: Oh, okay. So the heavy session is its own step, and what it leaves behind is files.

BEN: Right. Step three: keep the always-on rules short. The files only help if the agent knows when to read them. So you write a short, always-on rule that maps situations to files.

MAYA: What does that look like?

BEN: The chapter shows a four-line example. Implementing a feature? Read the implementation plan. Working on UI? Read the UI conventions. Creating files or folders? Check the project structure. Hitting an error? Check known issues first, and log new fixes there.

MAYA: That's tiny. Four lines, and it points to everything.

BEN: That's the idea. And both major coding tools support this split, as of October 2026. In Cursor, project rules live in {.cursor/rules|dot cursor slash rules}. They can be applied always, when the agent judges them relevant from their description, when matching files are in context, or only when you at-mention them.

MAYA: Four modes. Any advice on size?

BEN: Cursor's docs say to keep rules under five hundred lines, split large rules into composable ones, and reference files instead of copying their contents.

MAYA: And Claude Code?

BEN: Claude Code loads {CLAUDE.md|CLAUDE dot M D} at the start of every session. Its docs recommend under two hundred lines per file, because, quote, "longer files consume more context and reduce adherence." Instructions that only matter for part of the codebase can go in path-scoped rules under {.claude/rules/|dot claude slash rules}. Those load only when Claude works with matching files.

MAYA: Okay, here's a trick I thought I knew. I keep my {CLAUDE.md|CLAUDE dot M D} short by using imports. I just import the other files. That saves context, right?

BEN: [laughs] That's a really common misconception. The docs are explicit. {@imports|At imports} help organise a file, but they don't reduce its cost, because imported files load at launch too.

MAYA: [sighs] So my short file was secretly a long file.

BEN: Pretty much. Path-scoped rules are the ones that load only when they're needed. And Claude Code's guide gives a great test for every always-on line. [slowly] "Would removing this cause Claude to make mistakes?" If not, cut it.

MAYA: That's harsh, but I like it. Step four?

BEN: Let the plan drive the work, and survive resets. With a written plan, you can start a fresh session and ask the agent to continue from the next stage. Anthropic's prompting guide notes that current models are, quote, "extremely effective at discovering state from the local filesystem". It recommends progress notes, structured test files and git history as the state a new session reads first.

MAYA: So a fresh session isn't starting from zero. It reads the files and catches up.

BEN: Exactly. And Claude Code's guide suggests writing the spec in one session and starting a new one to implement it, so the implementation session has clean context.

MAYA: Let me try a mini summary. The main benefit of this whole approach is that the project's memory is on disk, not in the chat. So I can throw the chat away.

BEN: That's it. The chapter calls it project memory that survives a context reset. Chapter five covers compaction and note-taking in more depth.

MAYA: And step five?

BEN: Build foundations first, and verify each stage. Put project setup, structure and core configuration at the front of the plan, before features. And give each stage a check the agent can run, such as tests, a build, or a screenshot comparison.

MAYA: Why does the check matter so much?

BEN: Claude Code's guide calls verification the difference between a session you have to watch and one you can leave running. Chapter seven covers verification in depth.

MAYA: Okay. This all sounds great on paper. Where does it go wrong?

BEN: The chapter names three pitfalls. The first is contradictory instructions. Agents follow written instructions literally. Say your request says MVP, but your planning rule describes stages for a complete product.

MAYA: Then I get an MVP, because that's what I asked for?

BEN: Not necessarily. The plan may follow the more detailed text, and come out far larger than you wanted. Claude Code's docs say it plainly: quote, "if two instructions contradict each other, Claude may pick one arbitrarily."

MAYA: [thoughtful] Arbitrarily. So it's not even predictable which one wins.

BEN: Right. So read what you feed the agent, review generated plans before you build from them, and prune your rule files regularly.

MAYA: Second pitfall?

BEN: Letting the agent choose key decisions. A workflow that proposes a tech stack may pick components that fit the requirements but not your situation. Like an API you don't have access to.

MAYA: But the agent probably knows more frameworks than I do. Isn't it better at picking?

BEN: Maybe, but it can pick components that fit the requirements and still not fit your situation. So decide the stack yourself, and write it into the requirements.

MAYA: Fair. And the third?

BEN: Bloated or copied rule files. Long rule files get ignored. Claude Code's docs put it like this: quote, "If Claude keeps doing something you don't want despite having a rule against it, the file is probably too long and the rule is getting lost."

MAYA: Oof. So when the agent ignores a rule, the fix might be a shorter file, not one more rule.

BEN: That's the idea. Cursor's advice is to start small and add a rule only when the agent repeats the same mistake. And someone else's templates are a starting point. Build files that match how you work.

MAYA: Okay, last big topic. Subagents. Can't I just run a bunch of them in parallel and finish everything faster?

BEN: Sometimes. Claude Code can run subagents, each in, quote, "its own context window with a custom system prompt, specific tool access, and independent permissions." They keep exploration out of the main conversation, and they can run in parallel.

MAYA: So, parallel everything?

BEN: No. The docs say parallel research, quote, "works best when the research paths don't depend on each other". And for multi-step workflows, they recommend running subagents in sequence, passing results from one to the next.

MAYA: Give me examples of each.

BEN: Generating several UI variations, or researching separate modules, suits parallel subagents. A staged build, where each step depends on the last, doesn't.

MAYA: [curious] Because step three can't start until step two exists.

BEN: Exactly. Chapters twenty-three and twenty-seven cover this further.

MAYA: Okay. What part of this will still be true in a few years, and what will date?

BEN: The durable part is the principles. Agent output depends on the information in its context, and you can design that information. More context isn't automatically better, so keep always-on instructions small and load the rest when relevant. Keep durable project knowledge in files, not chat history, so any fresh session can pick it up. And split knowledge into focused documents, with a short map of when to use each.

MAYA: And the pitfalls?

BEN: Those last too. Contradictory instructions give unpredictable results, so review plans and prune rules. Make key decisions, like the tech stack, yourself. And parallelism helps only with independent tasks.

MAYA: And what will date?

BEN: The rule mechanisms and limits. Cursor's rules folder and its four modes, {CLAUDE.md|CLAUDE dot M D}, the {.claude/rules/|dot claude slash rules} folder, and the five-hundred and two-hundred line guidance. All of that is as of October 2026. Subagent features and concurrency limits. Context window sizes of particular models. And even the label "context engineering" itself. The practice is older than the term.

MAYA: Let me try to sum up, and you correct me. Context engineering means designing everything the model sees, not just wording one prompt. More context isn't better, because of context rot, so the goal is the smallest set of high-signal information.

BEN: Good so far.

MAYA: In practice: write requirements with my own stack decision, turn them into a plan and a few focused files, keep the always-on rule short as a map to those files, and load the rest just in time. Because the plan lives in files, a fresh session can pick up the work. Watch out for contradictions and bloated rules. And only run subagents in parallel when the tasks are independent.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You're reviewing your always-on rule file, line by line. What question decides whether a line stays?

PAUSE: 5

MAYA: "Would removing this cause Claude to make mistakes?" If not, cut it. Anything that only matters sometimes goes into a separate file the agent reads when it needs it.

BEN: Perfect. Question two. Your requirements say MVP, but your planning rule describes stages for a complete product. What can happen, and what do you do?

PAUSE: 5

MAYA: The plan may follow the more detailed text and come out much bigger than I wanted, because Claude may pick one instruction arbitrarily. So I fix the contradiction, regenerate the plan, and review it before building.

BEN: Exactly. And last one. You have a staged build where each step depends on the one before. Do you run the stages as parallel subagents?

PAUSE: 5

MAYA: No. Parallel subagents are for independent work, like separate modules or UI variations. Dependent steps run in sequence, passing results from one to the next.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. My favourite is the reset test: start a fresh session, ask the agent to continue from the plan with no other explanation, and note what's missing from your files.

BEN: And try writing an always-on rule under ten lines that maps situations to your files. Next time, in chapter four, we look at context rot: why more context isn't automatically better, and what the evidence says.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter four.
