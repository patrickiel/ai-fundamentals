<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 04: Context rot

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter four: context rot. Why a bigger context window doesn't fix long agent sessions.

MAYA: Okay, so last time we talked about context engineering. The idea that what really matters is the information the agent works from, not one clever prompt.

BEN: Right. And today is the uncomfortable follow-up. Even if you put exactly the right information in the context, the model doesn't use all of it equally well. And the more you put in, the worse that gets.

MAYA: [curious] Wait, really? I'd have assumed more information is always better. That's the whole point of a huge context window, isn't it?

BEN: That's the intuition almost everyone starts with. So let's take it apart. If you only remember one sentence from this episode, make it this one: models use long inputs less reliably as those inputs grow, well before the window is full. So in a long session, after every task, you have to decide what stays in context.

MAYA: Okay. Let's start at the beginning. What exactly is in the context window? I think of it as, well, my conversation with the model.

BEN: That's a small part of it. In a coding agent like Claude Code, a lot gets loaded before you type a single word. The system prompt, your {CLAUDE.md|CLAUDE dot M D} files, auto memory, information about your environment, the names of your MCP tools and the descriptions of your skills.

MAYA: Before I've even said hello.

BEN: [laughs] Before you've said hello. And then, as the session goes on, every file the agent reads, every command output and every tool result gets added on top.

MAYA: So the window fills up much faster than I think.

BEN: Much faster. And context rot is what happens as it fills. It's the drop in output quality as the context grows.

MAYA: How do we actually know that? Is this a feeling people have, or has someone measured it?

BEN: It's been measured. The best-known study is a technical report from Chroma, from 2025. They tested eighteen models from Anthropic, OpenAI, Google and Alibaba. And here's the clever part: they used deliberately simple tasks. Things that should be easy no matter how long the input is.

MAYA: And?

BEN: Their headline finding was, quote, "models do not use their context uniformly." Performance fell as the input got longer, even when the task itself stayed exactly the same.

MAYA: Hmm. So the question is just as easy, but surrounding it with more text makes the model worse at it.

BEN: Exactly. And they found a few things that make it worse. One: the drop is faster when the question and the answer are worded differently. If you ask about "login problems" and the relevant passage talks about "authentication failures", that's harder.

MAYA: [thoughtful] Which is basically always the case in real work. I never know the exact words the answer uses.

BEN: That's the point. Two: distractors. Statements that are related to the question but wrong. Chroma wrote that "even a single distractor reduces performance relative to the baseline", and several distractors compound the damage.

MAYA: Okay, that one worries me. Because a long coding session is full of things that are almost right.

BEN: Hold that thought, we'll come back to it. The third finding is the one I find most striking. On a memory benchmark, every single model did much better with a short, focused prompt of about three hundred tokens than with the full chat history of about a hundred and thirteen thousand tokens. And the full history contained the same answer.

MAYA: So, same answer, buried in a long conversation, and the model finds it less reliably than if you just hand it the relevant bit.

BEN: Yes. Less can really be more.

MAYA: Is this a new problem, or has it been around for a while?

BEN: There's an older result that points the same way. A paper called Lost in the Middle, by Liu and colleagues, from 2023. They found that models use information best when it's at the beginning or the end of a long input, and noticeably worse when it's in the middle.

MAYA: [laughs] Like me reading a long email. I read the start, skim the middle, and read the last line.

BEN: [laughs] It's not a bad analogy. And it held even for models built specifically for long contexts.

MAYA: Okay, but why does this happen? What's going on inside the model?

BEN: Anthropic's guide to context engineering gives a plain explanation. Transformers relate every token to every other token. So with n tokens, you get n squared relationships. And models see far fewer very long sequences during training. Anthropic describes it as an "attention budget". Every extra token draws on that budget.

MAYA: So the budget gets spread thinner and thinner.

BEN: Right. And the important detail is that it's a gradual loss of precision, not a cliff. There's no moment where it suddenly breaks. Anthropic says context should be treated as, quote, "a finite resource with diminishing marginal returns."

MAYA: Okay, here's my obvious objection. Windows keep getting bigger. Doesn't that just solve the problem?

BEN: That's the trap. As of October 2026, Claude Code supports a one-million-token window on several models. Thariq Shihipar from Anthropic wrote a guide to session management for exactly that situation. His point is that a much larger window gives you room to work longer, but, quote, "model performance degrades as context grows" anyway.

MAYA: So a bigger window gives me more room, not immunity.

BEN: That's exactly how the chapter puts it. More room, not immunity.

MAYA: Then where's the line? At what token count does it start to rot?

BEN: [sighs] Everyone wants that number, and there isn't one. Chroma's data shows it depends on the model and the task, and on what else is in the window. Drew Breunig's survey mentions a Databricks study where one large open model started losing accuracy at around thirty-two thousand tokens. But that's one model on one task. Don't treat it as a rule.

MAYA: Okay. So it's real, it's measured, and there's no safe number. What does it actually look like when it goes wrong?

BEN: Drew Breunig groups long-context failures into four patterns, each backed by published experiments. The first is context poisoning. A hallucination or an error gets into the context, and the model keeps referring back to it.

MAYA: Give me a coding example.

BEN: The agent "remembers" a function signature that doesn't exist, and keeps building on it. It's in the context, so to the model it looks like a fact.

MAYA: Oof. Okay, number two?

BEN: Context distraction. The model leans on its long history instead of what it learned in training. In a coding session, it repeats earlier actions instead of trying a better approach.

MAYA: Number three?

BEN: Context confusion. Irrelevant material still shapes the output. For example, tool definitions you're not using. The agent calls a tool that has nothing to do with the task, just because it's sitting there in the context.

MAYA: And four?

BEN: Context clash. Parts of the context contradict each other. The classic case is a failed first attempt, and then your later correction. Both are in the context, and they pull in different directions.

MAYA: [thoughtful] I do that all the time. The agent gets something wrong, and I just type "no, not like that, do it this way."

BEN: Most of us do. And there's a study that makes this one painful. Microsoft and Salesforce took benchmark prompts and split them across several turns of a conversation, the way a real chat unfolds. Average performance dropped thirty-nine percent. And the researchers wrote, quote, "when LLMs take a wrong turn in a conversation, they get lost and do not recover."

MAYA: Do not recover. That's harsh.

BEN: It is. And it's the main argument for removing failed attempts instead of arguing past them. Which brings us to the practical half of the chapter.

MAYA: Good, because so far I'm mostly scared. What do I actually do?

BEN: [warm] The key idea, from Anthropic's Claude Code team, is that every turn is a branching point. When the agent finishes a step, your next action decides what happens to the context. And there are five options. Continue, rewind, clear, compact, or hand it to a subagent.

MAYA: Continue is just, sending another message.

BEN: Yes. Continue when it's the same task and the context is still relevant. It's the default, and it's often not the best choice.

MAYA: Okay, let's go through the others. Compact first. What is compaction?

BEN: Compaction replaces the conversation with a summary, and the session continues on top of that summary. It needs no work from you. But it's lossy. Summaries lose information.

MAYA: Claude Code does that automatically when the window gets full, right? So why should I care?

BEN: Because of when it happens. Thariq explains it nicely. Automatic compaction fires when the context is most rotted, so the model is, in his words, "at its least intelligent point". And it has to guess what you'll do next.

MAYA: [laughs] So the summary is written by the most tired version of the model.

BEN: Pretty much. His example: after a long debugging session, autocompact summarises the investigation. Then your next message is, "now fix that other warning we saw in {bar.ts|bar dot T S}". And that warning never made it into the summary.

MAYA: So how do I compact well?

BEN: Four habits. First, do it yourself, before the automatic pass, and say what comes next. In Claude Code you type {/compact|slash compact}, followed by a hint, like "focus on the auth bug fix". And a large window actually helps here, because it gives you room to do it early.

MAYA: Okay, compact early, with a hint. Second?

BEN: Put standing compaction rules in your {CLAUDE.md|CLAUDE dot M D}. For example, "When compacting, always preserve the full list of modified files and any test commands."

MAYA: Third?

BEN: You can move the automatic threshold with {/autocompact|slash autocompact}. For example, {/autocompact 500k|slash autocompact five hundred K}, if you want it to fire earlier. And fourth, you can compact just part of the history. In the rewind menu there's an option called "Summarize from here". It condenses everything after a chosen message, and keeps your earlier instructions intact.

MAYA: And what survives compaction? What does the agent still know afterwards?

BEN: Good question, because this is where people lose things. After compaction, Claude Code re-injects the system prompt, your project-root {CLAUDE.md|CLAUDE dot M D}, auto memory, and the plan from plan mode. It re-reads up to five recently modified files, and it re-injects the skills you've used, up to a size limit.

MAYA: And the stuff I just said in the chat?

BEN: That gets summarised along with everything else. So the rule is simple: anything that must persist goes into {CLAUDE.md|CLAUDE dot M D}, not just into the conversation.

MAYA: Got it. Next option: clear.

BEN: Clear, with {/clear|slash clear}, gives you an empty context. A fresh start. Thariq recommends a new session for each new task, usually, quote, "with a brief you've distilled from what you just learned".

MAYA: That sounds like more work than compacting.

BEN: It is more work. But his argument is that "the resulting context is what you decided was relevant." Not what a summary happened to keep.

MAYA: What does clearing protect me from?

BEN: Claude Code's best-practice guide gives it a great name: the kitchen-sink session. You do one task, then something unrelated, then go back to the first task, and so on, until the context is full of irrelevant material.

MAYA: [laughs] Guilty. My sessions are definitely kitchen sinks.

BEN: A common fix is to write a brief to a file before you clear. The goal, the current state, decisions and constraints, known issues, and next steps. Then start the new session from that file.

MAYA: Does the format of that file matter?

BEN: A structured format helps. In Anthropic's long-running-agent harness, they kept the feature list in JSON, because the model was less likely to wrongly rewrite JSON than Markdown.

MAYA: Interesting. Okay, option three: rewind. That's the one I never use.

BEN: And it's the one that fixes the habit you confessed to earlier. When an attempt fails, the instinct is to reply, "no, try another way". But that leaves the failed attempt in the context, where it can poison or clash with everything that follows.

MAYA: Poisoning and clash. Two of the four failures.

BEN: Exactly. Here's Thariq's example. Claude reads five files, tries an approach, and it doesn't work. Instead of correcting it, you rewind to just after the file reads, and re-prompt with what you learned. You keep the useful reads, and you lose the dead end.

MAYA: Oh, that's nice. So I keep the work, but drop the mistake.

BEN: Right. In Claude Code, pressing Escape twice, or typing {/rewind|slash rewind}, opens a menu. It can restore the conversation, the code, or both, to any earlier prompt.

MAYA: Any rule of thumb for when to give up and start over?

BEN: The best-practice guide has one. If you've corrected Claude more than twice on the same issue, the context is, quote, "cluttered with failed approaches." So clear, and start again with a better prompt.

MAYA: More than twice. I like that, it's concrete.

BEN: One warning about rewind, though. It only restores files that were changed by Claude's edit tools. Not files changed by shell commands. So it doesn't replace git.

MAYA: Noted. Keep using git. And the last option: subagents.

BEN: A subagent runs in its own fresh context window and returns only its result. All its file reads, searches and logs never enter your main session.

MAYA: So it does the messy work somewhere else, and just hands me the answer.

BEN: Exactly. And Thariq's test for when to use one is a single question. [slowly] Will I need this tool output again, or just the conclusion?

MAYA: And if it's just the conclusion, delegate it.

BEN: Yes. His examples are verifying work against a spec, researching another codebase before implementing something, and writing docs from git changes. Claude Code's docs add that you can simply ask for it, like "use subagents to investigate X". And subagent requests still count toward your usage, so they're not free.

MAYA: We'll go deeper on subagents later in the course, right?

BEN: Chapter five covers subagent architectures, and chapter twenty-three compares subagents with agent teams.

MAYA: There was one more tip at the end of the chapter, about restating goals?

BEN: Yes. In long sessions on a single task, a common pattern is to ask the agent now and then to restate the goal, the constraints, and the decisions so far. Lost in the Middle suggests why that might help: information near the end of the input gets used more reliably than information buried in the middle.

MAYA: So restating moves the important stuff to the end.

BEN: That's the idea. But the chapter is careful here: treat it as a habit to test, not a guarantee. And for anything that has to survive compaction, use {CLAUDE.md|CLAUDE dot M D} or a plan file instead.

MAYA: Okay. Before we wrap up, what part of this will still be true in a few years, and what will date?

BEN: The durable part is the principles. Models use long inputs unevenly. Everything in the window competes for attention, including tool output, old attempts and unused tool definitions. Summaries lose information, so choose when to summarise and say what matters next. Start unrelated work fresh, with a deliberate brief. Remove failed attempts instead of correcting on top of them. And push output-heavy steps into separate contexts.

MAYA: And what will date?

BEN: Which models have million-token windows. The exact Claude Code commands and menus, and what gets re-injected after compaction. And the specific benchmark numbers, which come from models tested in 2025.

MAYA: Let me try to sum up, and you correct me. Context rot is the drop in quality as the context fills up, and it starts long before the window is full. There's no safe number, because it depends on the model and the task. Bigger windows give you room, not immunity.

BEN: Good so far.

MAYA: Long contexts fail in four ways: poisoning, distraction, confusion and clash. And after every turn I have five options: continue, rewind, clear, compact, or delegate to a subagent.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. A long debugging session is nearly full, and you want to keep going on the same task. What should you do, and what makes it work better?

PAUSE: 5

MAYA: Compact it myself, before the automatic pass kicks in, and add a hint about what comes next. Like, {/compact|slash compact}, keep the failing test names, next I'll fix the retry logic.

BEN: Perfect. Question two. The agent tried an approach, and it failed. Why is "no, try another way" a bad reply, and what's the better move?

PAUSE: 5

MAYA: Because the failed attempt stays in the context, and can poison it or clash with what comes next. Better to rewind to just before the attempt, and re-prompt with what I learned.

BEN: Exactly. And last one. You're about to run a big search through the codebase. What question decides whether to hand it to a subagent?

PAUSE: 5

MAYA: Will I need this output again, or just the conclusion? If just the conclusion, delegate it.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. My favourite is the first one: run {/context|slash context} at the start of a session and see how much is used before you've typed anything.

BEN: And then try rewinding once instead of correcting, and compare the result. Next time, in chapter five, we go deeper into the three techniques for work that outlasts one context window: compaction, structured notes, and subagents.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter five.
