<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 01: Getting better LLM answers

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter one: getting better answers from LLMs. Clear prompts, managed context, and the right tools.

MAYA: Last time, in the introduction, we did the primer on how LLMs work. Tokens, attention, the context window. So now we actually get to use one.

BEN: Exactly. We know roughly what's going on inside. Today is about what you can do from the outside to get better answers.

MAYA: [curious] Honestly, my first instinct is: just use the best model. Isn't that the biggest lever?

BEN: That's the instinct this chapter pushes back on. If you only remember one sentence, make it this one: answer quality depends less on which model you use than on how you use it. Clear and specific instructions, real material to work from, a context window kept focused, standing instructions saved where they belong, and the right tool for the job.

MAYA: That's five things. Okay, let's go through them. Where do we start?

BEN: With writing the request. And here's something nice: Anthropic, OpenAI and Google all give the same first piece of advice. Be clear, direct and specific about the task, the constraints and the output format.

MAYA: All three agree? That doesn't happen often.

BEN: [laughs] Not often, no. And each one has its own way of saying it. Anthropic's guide suggests thinking of the model as, quote, "a brilliant but new employee who lacks context on your norms and workflows."

MAYA: Brilliant, but new. So smart, but it doesn't know how we do things here.

BEN: Right. And it gives you a test. Show your prompt to a colleague who has no context on the task. If they would be confused, the model will be too.

MAYA: [thoughtful] Oof. Most of my prompts would fail that. They only make sense if you're inside my head.

BEN: Most people's would. Google's guide makes the same point. It tells you to give "clear and specific instructions", and to state constraints, such as a length limit. And OpenAI's guide likens a non-reasoning model to "a junior coworker" who needs explicit instructions.

MAYA: Okay, but what does "specific" actually look like? I think I'm being specific when I say "not too long".

BEN: That's the perfect example, because the chapter has a little table of vague requests next to specific ones. "Not too long" becomes "under a hundred and fifty words".

MAYA: Okay, that's a number. Next?

BEN: "Make it better" becomes "rewrite for a non-technical manager, and keep the three key numbers". And "summarise this" becomes "five bullets, each under fifteen words, then one sentence on risks".

MAYA: So the trick is replacing fuzzy words with targets. Something the model can actually hit.

BEN: That's it. And state the output format too. A table, a checklist, a formal email, a JSON object. One more thing: if you want more than the minimum, ask for it explicitly, rather than hoping the model infers it.

MAYA: Got it. What's next?

BEN: Say why, not just what. Anthropic's guide has a lovely example. A bare rule, "never use ellipses", works less well than the same rule with its reason: "your response will be read aloud by a text-to-speech engine, so never use ellipses."

MAYA: [curious] Why would the reason matter? The rule is the same.

BEN: Because the model can then generalise from the explanation to cases the rule didn't name. It knows what you're actually optimising for, not just the one thing you banned.

MAYA: So it can follow the spirit of the rule, not just the letter.

BEN: Good way to put it. And it works for everyday requests too. "Under two hundred words, because it goes in a newsletter sidebar" tells the model what to optimise for.

MAYA: Okay. Next, structure?

BEN: Yes. For anything longer than a few lines, separate the parts of the prompt. All three guides recommend Markdown headings or XML tags, so the model can tell the instructions, the context, the examples and the input apart.

MAYA: Is there a recommended order?

BEN: OpenAI suggests a developer message usually covers identity, instructions, examples and context, in roughly that order.

MAYA: And examples. I've heard the term "few-shot". Is that the same thing?

BEN: Yes. Few-shot prompting means including a small number of example inputs and outputs, to show the pattern you want. Examples are one of the most reliable ways to steer format and tone.

MAYA: So, more examples, better results?

BEN: [laughs] Not quite. Anthropic recommends three to five, and they should be relevant, varied, and wrapped in tags, so they aren't mistaken for instructions. Google goes further and recommends always including few-shot examples, but it warns that too many can make the model overfit to them.

MAYA: Overfit, meaning it copies the examples too closely?

BEN: That's the risk, yes. And there's one partial exception: reasoning models. Chapter two covers why some vendors say to try zero-shot first, so no examples at all.

MAYA: Okay. And what if I do all of this, and the first answer is still wrong?

BEN: Then you're completely normal. Google's guide says plainly that "prompt engineering is iterative". Rephrase, reorder the content, or reframe the task, and test again. A first answer that misses is normal. Give feedback in the next turn.

MAYA: And when I finally find wording that works?

BEN: Save it. We'll get to where in a minute.

MAYA: Before we leave prompts, what about the hard problems? The ones that need actual thinking?

BEN: Some problems need several steps of work before an answer. And two research ideas underpin today's tools. Chain-of-thought prompting showed that models do better on multi-step problems when they write out the intermediate steps.

MAYA: Like showing your working in a maths exam.

BEN: That's a fair analogy. The second idea is ReAct, short for reason plus act. It interleaves reasoning with actions, such as searches or tool calls, so each observation informs the next step.

MAYA: And that's what the "thinking" button does?

BEN: Current "thinking" or "reasoning" modes build this in. Turn it on for multi-step problems, and leave it off for simple lookups, because thinking costs time and tokens.

MAYA: So not always on. Okay, let me do a quick summary of part one. Be specific, with real targets instead of fuzzy words. Say why. Structure long prompts and show examples. And expect to iterate.

BEN: [warm] Exactly right. Now, part two: the context window.

MAYA: We touched on this in the introduction. Remind me of the definition?

BEN: Anthropic's docs define it as "all the text a language model can reference when generating a response, including the response itself." It works as the model's working memory, and it's separate from what the model learned in training.

MAYA: Okay. And why does that matter for me as a normal chat user?

BEN: Two consequences. First, the model has no memory between turns.

MAYA: [curious] Wait. But it clearly remembers what I said three messages ago.

BEN: It looks that way. What actually happens is that each turn's input is, quote, "all previous conversation history plus the current user message". The whole conversation is sent again, every time. And the reply becomes input for the next turn.

MAYA: So it's not remembering, it's rereading the whole chat each time.

BEN: Right. And everything counts toward that budget. The system prompt, your messages, uploaded files and images, tool definitions and tool results.

MAYA: And the second consequence?

BEN: More context is not automatically better. As the token count grows, quote, "accuracy and recall degrade, a phenomenon known as context rot." And chat apps may also drop the oldest turns once the window is full.

MAYA: So the instructions I gave at the very start of a long chat might just disappear?

BEN: They may, yes. Chat apps may drop the oldest turns once the window is full. And chapter four covers context rot in depth.

MAYA: So what's the practical rule?

BEN: [slowly] New task, new chat. And when you continue a long piece of work, open the new chat with a short recap of what matters.

MAYA: [sighs] I have one chat that I've been using for everything for weeks.

BEN: [laughs] You're not alone. And Claude Code's guide gives the same advice for agents: a clean session with a better prompt usually beats a long one cluttered with failed attempts.

MAYA: Okay, mini-summary. The model doesn't remember anything between turns, it rereads the whole conversation. Everything in there competes for the budget, and quality drops as it fills. So, new task, new chat, with a short recap if I'm continuing.

BEN: That's it.

MAYA: Part three, then. Material.

BEN: Yes. Give the model material, not just questions. Models answer better from sources you supply than from what they remember.

MAYA: Even the really big models?

BEN: Even those. Google's guide warns against assuming the model already has all the information it needs, and says to supply it. In chat apps, that means attaching PDFs, pasting links, and adding screenshots or diagrams.

MAYA: Does it matter where in the prompt I put a long document?

BEN: It does, and this is one of the most concrete tips in the chapter. Anthropic's guide says to put long documents near the top of the prompt, and your question at the end. In its tests, that improved quality by up to thirty percent on complex, multi-document inputs.

MAYA: [thoughtful] Huh. I always write my question first and then paste the document underneath.

BEN: Most people do. So flip it: document first, question last.

MAYA: And how much material can I give it?

BEN: Quite a lot. As of October 2026, current Claude models accept up to six hundred images or PDF pages in a single API request. That's the kind of number that will change, though.

MAYA: Okay. Now, saving what I repeat. You said we'd get to that.

BEN: Here's the signal. If you keep pasting the same instructions into new chats, they belong in a project.

MAYA: What exactly is a project?

BEN: In Claude, a project is a self-contained workspace. It has its own chat history, a knowledge base of uploaded files that serves as background for every chat in it, and project instructions that shape the responses. ChatGPT and Gemini offer similar workspaces under their own names, as of October 2026.

MAYA: So the files and the instructions are just there, in every chat inside the project.

BEN: Exactly. You set it up once.

MAYA: And what should go into those project instructions?

BEN: The provider guides suggest four things. First, a role. Anthropic notes that "even a single sentence makes a difference" in focusing behaviour and tone.

MAYA: Just one sentence?

BEN: Even one. Second, the task, the constraints and the output format, written as specifically as in part one. Third, a sample output, because examples steer format more reliably than descriptions. And fourth, clear sections, in Markdown or XML.

MAYA: Is there an example in the chapter?

BEN: There is. A short set of project instructions with three headed sections. The role says, "You are a careful editor who fact-checks and grammar-checks essays." The task: for each essay, list the factual claims that need a source, then the grammar fixes. And because the readers are first-year students, explain each fix in one short sentence.

MAYA: And the output format?

BEN: Two sections, "Claims to verify" and "Grammar fixes". Each grammar fix shows the original, then the corrected version, then the reason in brackets.

MAYA: Nice. So role, task with the reason for the audience, and the exact shape of the answer. When is a project worth it?

BEN: Two cases. Recurring tasks, like weekly reports or essay checking. And one-off but document-heavy work, where many questions draw on the same files.

MAYA: Okay. And then there's memory. That's different from projects, right?

BEN: A bit different. Assistants also keep longer-term memory across chats. Claude's memory, for example, is built from your conversations, quote, "as a set of individual topics".

MAYA: Can I see what it's stored about me?

BEN: Yes. As of October 2026, you can view, edit or delete those topics under Settings, then Memory. You can turn memory off for a single chat, or use incognito chats, which are neither saved nor searched. Each project has its own separate memory space. And Claude can also search past chats when you ask it to.

MAYA: And other assistants?

BEN: They have equivalent features, with different names and controls.

MAYA: [curious] Honestly, memory sounds like pure convenience. Why would I worry about it?

BEN: Because memory and custom instructions do the same job as a project. They place relevant information in the context automatically. And that makes them worth auditing. A wrong or outdated memory skews answers without you noticing.

MAYA: Oh. Because it's in the context, but I can't see it in the chat.

BEN: That's the problem. So read it now and then, and delete what's wrong.

MAYA: Okay. Part four: the tools around the model.

BEN: This is a big one, because much of what modern assistants can do comes from tools around the model, not the model itself. Let's start with search versus research.

MAYA: Aren't they the same thing?

BEN: No. Web search runs a query and answers from the results. It's fast, but only as deep as the first pages it reads. Research modes run a sequence of searches, each informed by the last. They explore different angles on their own, and return a cited report in minutes rather than seconds. That's how Anthropic described it when it launched its Research feature.

MAYA: And the others have that too?

BEN: OpenAI and Google offer comparable deep research modes, as of October 2026.

MAYA: So, when do I use which?

BEN: Quick search when speed matters, research when coverage does. And check the citations either way.

MAYA: [sighs] Okay, here's my biggest frustration. I ask for one small change, and I get the whole thing rewritten.

BEN: The chapter calls that out as a common frustration. Most assistants now open longer outputs in an editable panel beside the chat. Claude calls these artifacts, ChatGPT calls it Canvas, as of October 2026.

MAYA: And how does that help with the rewrite problem?

BEN: The good habit is to ask for changes to a named section only. And keep earlier versions you might want to return to.

MAYA: So, "change only the second paragraph", not "make it better".

BEN: Exactly. Which is just part one again: be specific.

MAYA: [laughs] Everything comes back to being specific. What about maths, or counting things?

BEN: Google's guide recommends turning on code execution for arithmetic, counting and calculation. And search grounding for recent or obscure facts.

MAYA: So let it run code instead of doing sums in its head.

BEN: That's the idea. And for software work, there are terminal coding agents, such as Claude Code, OpenAI's Codex, and Google's {Gemini CLI|Gemini C L I}. Names as of October 2026. They read and change files across a whole project.

MAYA: How is that different from pasting code into a chat?

BEN: Claude Code's guide describes the difference: the agent reads files, runs commands and works through problems, instead of answering and waiting. Chapters three, six and twenty-one cover working with agents.

MAYA: Okay. What part of all this will still be true in a few years, and what will date?

BEN: The durable principles first. Specific instructions with stated reasons beat vague adjectives, and iterating is normal. Structure long prompts and show examples of the output you want. The model is stateless between turns: everything it "knows" in a chat is context that's resent, and quality degrades as that context grows. New task, new chat.

MAYA: Keep going.

BEN: Supplying source material beats relying on the model's memory. Long documents go first, the question last. Save repeated instructions in projects or system prompts, and audit what memory has stored. And match the tool to the job: search or research, regenerate or targeted edit, thinking or standard mode.

MAYA: And what will date?

BEN: Product names, menus and plan limits. So Projects, the Memory settings, Research, artifacts and Canvas, all as of October 2026. Context window sizes and per-request file limits. Which coding agents exist and what they're called. And vendor differences in advice, such as how many examples to include.

MAYA: Let me try to sum up, and you correct me. The model matters less than how you use it. Write specific requests with real targets, say why, structure them, show examples, and iterate. Turn thinking on only for multi-step problems.

BEN: Good so far.

MAYA: The model doesn't remember between turns, it rereads the whole chat, and quality drops as that grows. So, new task, new chat. Give it real material, with long documents first and the question last. Save repeated instructions in a project, and check what memory has stored. And pick the right tool: search or research, targeted edits, code execution, or an agent.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your prompt has a bare rule: "never use ellipses". What one change does the chapter suggest, and why does it help?

PAUSE: 5

MAYA: Add the reason. Like, "your response will be read aloud by a text-to-speech engine, so never use ellipses." Then the model can generalise to cases the rule didn't name.

BEN: Perfect. Question two. You've worked on one task in a single chat for a long time, and the answers start repeating or drifting. What do you do?

PAUSE: 5

MAYA: Start a new chat, and open it with a short recap of what matters. Because the whole conversation gets resent every turn, and quality degrades as the context grows.

BEN: Exactly. And last one. You paste the same instructions every week for the same task. Where should they live, and what four things go into them?

PAUSE: 5

MAYA: In a project. And the instructions get a role, the task with its constraints and output format, a sample output, and clear sections in Markdown or XML.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. My favourite is the last one: open your assistant's memory settings, read what it has stored, and delete anything wrong or outdated.

BEN: And try the first one too. Ask for a summary that's "not too long", then ask for five bullets, each under fifteen words, for a busy executive, and compare. Next time, in chapter two, we look at reasoning models: how to prompt them, and why some vendors say to try zero-shot first.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter two.
