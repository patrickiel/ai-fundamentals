<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 11: Agent Skills

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter eleven: Agent Skills. How to package know-how as folders that an agent loads in stages.

MAYA: Last time, in chapter ten, we looked at how MCP servers work. Tools as described functions, with names, descriptions and input schemas.

BEN: Right. MCP is about connecting an agent to tools and data. Today is about giving it know-how. And here's the one sentence for today. A skill is a folder of instructions, reference files and scripts that an agent loads in stages. Only a short name and description sit in context until a task needs the rest. So a general agent can take on specialist procedures without filling its context window.

MAYA: Loaded in stages. I want to understand that part. But first, why do we need skills at all?

BEN: Because a lot of agent work is repetitive. The same report format, the same review checklist, the same extraction steps. If you re-explain the procedure in every conversation, it's slow, and the results are inconsistent.

MAYA: [laughs] I have a notes file of prompts I paste in all the time.

BEN: You're not alone. Anthropic's documentation describes skills as a way to package, quote, "workflows, context, and best practices that turn a general-purpose agent into a specialist". Organized, quote, "like an onboarding guide you'd create for a new team member".

MAYA: [curious] But why not just put all of that in the system prompt? Or in one big instruction file?

BEN: That's the obvious alternative, and it runs into the context limit. Chapter three explained why context is a finite budget. Tool integrations show the same problem. In late 2025, Anthropic's engineers noted that "most MCP clients load all tool definitions upfront directly into context". At scale, that can mean hundreds of thousands of tokens before the agent even reads the request.

MAYA: Hundreds of thousands. Ouch.

BEN: As of October 2026, Claude Code defers the full MCP tool schemas until they're needed. But skills were designed from the start to load in stages.

MAYA: Okay. So what does a skill actually look like on disk?

BEN: The open Agent Skills specification defines the layout. A skill is a folder, say one called {pdf-processing|P D F processing}. Inside, there's one required file, {SKILL.md|SKILL dot M D}. And then three optional folders. Scripts, for executable code. References, for docs that are read on demand. And assets, for templates, schemas and data.

MAYA: And what goes in {SKILL.md|SKILL dot M D}?

BEN: Two parts. At the top, a block of YAML frontmatter with a name and a description. Below that, plain Markdown instructions.

MAYA: Can you give me an example?

BEN: The chapter's example is adapted from the spec. The description says the skill extracts text and tables from PDF files, fills PDF forms, and merges PDFs. And to use it when working with PDF documents, or when the user mentions PDFs, forms, or document extraction. The body then has two steps. For plain text, run a text-extraction script. To fill a form, read a forms reference file first.

MAYA: Are there rules for the name and description?

BEN: A few. The name can be up to sixty-four characters: lowercase letters, digits and hyphens. And it must match the folder name. Anthropic's surfaces also reject the reserved words "anthropic" and "claude". The description can be up to one thousand and twenty-four characters, and it should say what the skill does and when to use it.

MAYA: Is that all the frontmatter there is?

BEN: The spec has a few optional fields too. License. Compatibility, for environment needs like network access. Metadata. And an experimental one, {allowed-tools|allowed tools}. On top of that, individual products add their own fields. Claude Code, for example, supports {disable-model-invocation|disable model invocation}, so only the user can trigger the skill. Paths, which are glob patterns that limit when it activates. And {context: fork|context fork}, which runs the skill in a subagent.

MAYA: So a skill can bundle three kinds of things: instructions, scripts and resources. Why would I want all three?

BEN: Because they have different strengths. The documentation puts it neatly: "instructions for flexible guidance, code for reliability, resources for factual lookup".

MAYA: Scripts are the part that surprises me. Why ship code, if the model can write code anyway?

BEN: Anthropic's engineering post argues that some operations are better done by code than by token generation, because code is deterministic. And the documentation adds that a bundled script saves context. The agent runs it, and, quote, "only its output (such as 'Validation passed' or a specific error message) consumes tokens".

MAYA: So the code itself never enters the context. Just the result.

BEN: Exactly. The post's example is the PDF skill behind Claude's document features. It ships a pre-written Python script to extract form fields. And it keeps the form-filling instructions in a separate file, {forms.md|forms dot M D}, which loads only when a form actually needs filling.

MAYA: Which brings us to the stages. Progressive disclosure.

BEN: This is the central mechanism, and the idea most likely to outlast the specific product. There are three levels. Level one is the metadata, the name and description. It's always loaded, at startup, and costs about a hundred tokens per skill.

MAYA: Level two?

BEN: The instructions, meaning the body of {SKILL.md|SKILL dot M D}. That loads only when the skill is triggered, and the recommendation is to keep it under five thousand tokens. Level three and beyond are the resources. Reference files are read into context only when needed. Scripts run, and only their output enters. Until then, they cost nothing.

MAYA: [thoughtful] It's a bit like a library catalogue. The cards are always there, but you only take a book off the shelf when you need it.

BEN: That's a good analogy. And because unused files cost nothing, the post's authors note that the amount of context a skill can bundle is practically unlimited.

MAYA: Practically unlimited? Then why not put everything into one giant {SKILL.md|SKILL dot M D}?

BEN: Because then level two gets huge. The spec and the authoring guide give two layout rules. Keep {SKILL.md|SKILL dot M D} under five hundred lines. And link reference files one level deep from it. If the agent reaches a file through a chain of references, it may only preview it.

MAYA: So, a flat structure. The main file points straight at each reference.

BEN: Right. And here's the trade-off. Level one has to be good enough to route on. Anthropic's authoring guide says the description is what the agent uses, quote, "to choose the right Skill from potentially {100+|a hundred plus} available Skills".

MAYA: So the description is basically the trigger.

BEN: The description is the trigger. The agent decides whether to use a skill by matching the request against it. In Claude Code, the description, plus an optional {when_to_use|when to use} field, is cut off at one thousand five hundred and thirty-six characters in the skill listing. So put the key use case first.

MAYA: Okay. How do I write a skill that actually works?

BEN: Anthropic's authoring guide has practical rules. First, be concise. Their line is: "The context window is a public good." Assume the model already knows general facts, and only add what it lacks.

MAYA: [laughs] So no explaining what a PDF is.

BEN: [laughs] Exactly. Second, match the degree of freedom to the risk. For fragile, exact sequences, like a database migration, give a specific script and say not to change it. For judgement tasks, like a code review, give heuristics. The guide's analogy is a narrow bridge with cliffs, versus an open field.

MAYA: On the bridge you want exact directions. In the field you can wander.

BEN: That's it. Third, write the description in the third person, with the specific terms a user would say. "Processes Excel files and generates reports". Not "I can help you".

MAYA: My instinct would be to write the instructions first, and test afterwards.

BEN: The guide says the opposite: evaluate first. Run the agent on representative tasks without the skill, and record where it fails. Write three test scenarios. Then write just enough instructions to pass them.

MAYA: Huh. Test-first, but for instructions.

BEN: Right. Then iterate with two instances. One Claude session drafts and refines the skill, and a fresh one uses it on real tasks. You feed the failures you observe back into the draft. Test on every model you plan to use, because a skill that suits a large model may be too terse for a small one. And avoid time-sensitive statements inside the skill, or isolate them in an "old patterns" section.

MAYA: Okay, now the big one for me. Skills versus MCP. Last chapter we built tools. Isn't a skill just another kind of tool?

BEN: That's a common confusion. Both extend an agent, but they answer different needs. Claude Code's documentation puts it in two sentences. Quote: "MCP connects Claude to external services. Skills extend what Claude knows, including how to use those services effectively."

MAYA: So MCP is access, and skills are know-how.

BEN: That's the split. MCP is a protocol for connecting to external systems. It provides tools and data access. Typical uses are querying a database, posting to Slack, or driving a browser. A skill is instructions, knowledge, scripts and resources in a folder. It provides procedures, conventions and reference material. Think of a review checklist, a deploy workflow, or an API style guide.

MAYA: And the context cost?

BEN: For MCP, it depends on the client, as we said earlier. For skills, it's metadata only until the skill is triggered.

MAYA: Can I use them together?

BEN: They combine well. One documented pattern is an MCP server for the database, plus a skill that holds the schema and the query patterns. And when a skill refers to MCP tools, the authoring guide recommends fully qualified names: the server name, a colon, then the tool name. That way the agent can find the tool when several servers are connected. Chapters nine and ten cover MCP itself.

MAYA: Where do skills actually run?

BEN: This is the most product-specific part. The format is shared, but each product handles skills separately. And custom skills do not sync between surfaces.

MAYA: [curious] Wait. So if I make one in Claude Code, it isn't on claude.ai?

BEN: Not automatically. In Claude Code, skills live in your personal folder, {~/.claude/skills/|tilde, slash dot claude, slash skills}, or in a project's {.claude/skills/|dot claude slash skills} folder, which you commit for the team, or inside a plugin. Claude loads them when they're relevant, or you invoke one with a slash and the skill's name. Custom slash commands have been merged into skills. And edits are picked up within a running session.

MAYA: And claude.ai?

BEN: There, the pre-built document skills are on by default. Custom skills are uploaded as zip files in settings, on paid plans, and they're private to each user. On the Claude API, skills run inside the code execution tool's container. You reference the pre-built skills, called {pptx|P P T X}, {xlsx|X L S X}, {docx|D O C X} and {pdf|P D F}, or upload your own through the {/v1/skills|slash V one slash skills} endpoint.

MAYA: Now, I've heard that skills can't access the network at all. Is that right?

BEN: That's a common misconception, and it's only true on the API. Network access differs by surface. On the Claude API: none, and no installing packages at runtime. On claude.ai: full, partial or none, depending on user and admin settings. And in Claude Code: full. In the documentation's words, "the same network access as any other program on the user's computer".

MAYA: So in Claude Code, a skill can do anything a program on my laptop can.

BEN: As far as the network goes, yes. The durable lesson is narrower than the misconception, though. Check the runtime before you design a skill that depends on live data. And use MCP when you need authenticated access to an external system.

MAYA: Is this format just an Anthropic thing?

BEN: It's open. In December 2025, Anthropic published Agent Skills as an open standard for cross-platform portability, with the specification at agentskills dot io. And Anthropic's public skills repository on GitHub holds example skills. Among them are skill-creator, {mcp-builder|M C P builder}, algorithmic-art, canvas-design, and the document skills.

MAYA: Can I just use those?

BEN: Look at the licences first. Many are {Apache 2.0|Apache two point oh}, while the document skills are source-available rather than open source. And the repository states that the examples are for demonstration and education, and should be tested before critical use.

MAYA: You mentioned skill-creator. A skill that makes skills?

BEN: [laughs] Yes. Anthropic's launch post describes it. It asks about your workflow, generates the folder structure, formats {SKILL.md|SKILL dot M D}, and bundles resources. Claude Code offers it as a plugin, which can also generate test cases, compare runs with and without the skill, and tune how accurately the description triggers.

MAYA: Do I need it, though?

BEN: The authoring guide says no special tool is required. Claude understands the format. So after finishing a task together, you can ask it to, quote, "create a Skill that captures this pattern we just used".

MAYA: And then I just trust what it wrote?

BEN: No. Generated skills still need review. Read every file it produced, cut explanations the model doesn't need, and test the trigger.

MAYA: Which leads to security, I guess. If a skill can contain code...

BEN: Right. A skill can contain instructions and executable code. Anthropic warns that a malicious skill, quote, "can direct Claude to invoke tools or execute code in ways that don't match the Skill's stated purpose". Their advice is to use skills only from trusted sources, and to audit everything else. That includes scripts, dependencies, and any instructions to contact external networks.

MAYA: So installing a skill is like installing software.

BEN: That's exactly how to think about it. And chapter twelve treats this in depth.

MAYA: Before we wrap up, what part of this will still be true in a few years, and what will date?

BEN: The durable part first. Progressive disclosure: keep a short index always in context, and load detail on demand. That applies to any agent design.

MAYA: That's the library catalogue again.

BEN: Right. Next, separate access, meaning connecting to systems, from know-how, meaning doing a task well. Put exact, fragile steps in scripts, and leave judgement to the model. The description is the router, so write it for the requests that should trigger the skill. Skills are plain files, so version, review, test and improve them like code. And treat third-party skills as software you are installing.

MAYA: And what will date?

BEN: All of these are as of October 2026. The folder locations, the frontmatter fields, and product-specific extensions like paths and context fork. The token and length guidance: about a hundred tokens, under five thousand tokens, five hundred lines, and the listing limit of one thousand five hundred and thirty-six characters. The network rules per surface, and plan availability. And how skill-creator is distributed, and which example skills the repository contains.

MAYA: Let me try to sum up, and you correct me. A skill is a folder with a {SKILL.md|SKILL dot M D} file: a name and a description at the top, instructions below, plus optional scripts, references and assets. It loads in stages. The name and description always, the body when the skill is triggered, and everything else only when it's needed.

BEN: Good so far.

MAYA: The description is the trigger, so it says what the skill does and when to use it. MCP gives access, skills give know-how, and the two combine. Network access depends on where the skill runs. And a skill is software, so vet it before installing it.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You've written a skill, but the agent never uses it, even on requests where it should. What do you look at first?

PAUSE: 5

MAYA: The description, because that's what the agent routes on. It should be in the third person, say what the skill does and when to use it, use the words a user would actually say, and put the key use case first. Then I'd test it with prompts that should and shouldn't trigger it.

BEN: Perfect. Question two. Part of your procedure is an exact, fragile sequence, like a database migration. Do you describe it in the instructions, or do something else?

PAUSE: 5

MAYA: Put it in a bundled script, and tell the agent not to change it. Code is deterministic, and only the script's output enters the context.

BEN: Exactly. And last one. Your skill needs live data from an external system that requires a login, and it has to run on the Claude API. What's the problem, and what do you use?

PAUSE: 5

MAYA: On the API, skills have no network access at all. So the skill can't fetch the data itself. For authenticated access to an external system, I'd use MCP, and let the skill hold the know-how for using it.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. My favourite is testing the trigger. Write ten prompts, five that should trigger your skill and five that shouldn't, and adjust the description until all ten route correctly.

BEN: And try the others too. List five tasks you've explained to an agent more than twice. Build a minimal skill by hand, with one script for the exact part. Watch which files the agent actually opens. And have the agent turn a finished task into a skill, then review every file. Next time, in chapter twelve, we look at skill supply-chain security. Installed skills and plugins are third-party code plus instructions, so how do they attack, and how do you vet them?

MAYA: Thanks for listening to AI Fundamentals. See you in chapter twelve.
