<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 00: Introduction and how LLMs work

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. I'm the one who's supposed to know things. [laughs] This is the very first episode, so let's explain how the show works.

MAYA: Each episode follows one chapter of the course. I ask the questions a learner would ask, including the slightly embarrassing ones, and Ben explains.

BEN: And at the end, Maya recaps, and I ask three quiz questions. You get a few seconds to answer along with her. Each chapter also has exercises, and we'll point you to them.

MAYA: Today is chapter zero: the introduction, and Track zero, a short primer on how large language models, LLMs, actually work.

BEN: Right. And before the primer, a word on why this course exists. AI tools change every month. New models, new features, new names. The principles underneath change much more slowly.

MAYA: [curious] So the course is about the principles, not the products?

BEN: Exactly. The aim is that most of it is still true in two years, even when every product name in it has changed. Learn the principles and you can apply them across tools.

MAYA: Okay. Give me the one sentence for today.

BEN: Here it is. An LLM generates text one token at a time, from whatever is in its context, and talking to it changes that context, not the model itself.

MAYA: Hmm. I'll need you to unpack basically every word of that.

BEN: [laughs] That's the plan. Let's start with tokens. Tokens are pieces of text. A token can be a word, part of a word, or punctuation.

MAYA: And the model produces those one by one?

BEN: Yes. Given an input, the model calculates probabilities for the next token. It selects one, adds it to the sequence, and repeats. The course's example: after "The capital of France is", it may assign a high probability to "Paris".

MAYA: Wait, that's it? Next token, next token, next token? That can't be how it writes code or explains things.

BEN: That's the surprising part. The same process can produce an explanation, code, or a conversation. It's all next-token prediction.

MAYA: So when it answers me, it's predicting what text should come next.

BEN: Yes. And that's also why fluent output can still contain invented facts. Those are often called hallucinations.

MAYA: [thoughtful] So if an answer sounds confident and well written, that doesn't tell me it's true.

BEN: No. The same prediction process can also produce a convincing error. Keep that in mind, it comes back at the end of the episode.

MAYA: Okay. And how does the model know which words belong together? There was something about attention.

BEN: Inside a transformer, the kind of model the primer describes, embeddings represent tokens as lists of numbers. And attention lets the model combine information from the tokens in its context.

MAYA: What does that buy you?

BEN: It helps the model interpret a word in relation to the surrounding text. The example is the word "bank" beside the word "river". Attention helps the model read "bank" in light of "river".

MAYA: So attention is how the context changes what a word means.

BEN: Yes, that's the intuition.

MAYA: Next section: training and answering are different stages. That sounds obvious, but I suspect it isn't.

BEN: It's the one people mix up most. There are two broad phases of training. First, pre-training. The model learns to predict tokens from large amounts of data. Training adjusts its weights, the numerical parameters that encode learned patterns.

MAYA: And that's where its knowledge comes from.

BEN: Broad language capabilities and knowledge, yes. With gaps and errors. Then comes post-training, which shapes those capabilities into assistant behaviour.

MAYA: How?

BEN: Two ways the chapter names. Supervised fine-tuning uses examples of desired responses. And reinforcement learning uses reward signals, such as human preferences, or checks on whether a solution works.

MAYA: So after post-training it gives correct answers.

BEN: [laughs] Careful. These stages improve behaviour without guaranteeing correct answers. Better behaved is not the same as always right.

MAYA: Fair. And then the answering part?

BEN: That's inference: using the trained model to generate an answer. And here's the key point. In an ordinary conversation, your messages change the input the model works from. They do not update its weights.

MAYA: [sighs] Okay, confession. I always thought that when I correct a chatbot, it learns from that.

BEN: That's a really common belief. But training changes weights. Ordinary chat provides working material. Your correction is in the input for this conversation, and that's all.

MAYA: Which brings us to the context window, I guess.

BEN: It does. The context window limits how many tokens a model can work with at once. Instructions, conversation history and the documents you supply all take space.

MAYA: And what happens to things that don't fit?

BEN: Information outside the window must be supplied again, or retrieved by the application.

MAYA: [curious] But chat apps have memory now. Doesn't that solve it?

BEN: A chat product may save history or offer memory. But the application still has to select what to give the model on each call. The slides put it as: models are stateless, and read one stream of tokens.

MAYA: So memory is the app choosing what to put back into the context. The model itself still only sees what's in front of it.

BEN: Exactly right. And that's why clear instructions and relevant source material matter so much, and why long tasks need deliberate context management. Tracks one and two develop those habits.

MAYA: Okay, last part of the primer: tools and agents.

BEN: An application can give a model tools. Search, a calculator, or a way to run tests. The model requests a tool call, the application executes it, and the result becomes input for another model call.

MAYA: Oh, so the model doesn't run the tool itself. It asks, and the app does it.

BEN: Right. And that gives the system access to information and actions beyond text generation.

MAYA: And what's the difference between a workflow and an agent? People seem to use the words for everything.

BEN: The chapter separates them clearly. A workflow follows steps defined in code. An agent lets the model choose more of its next steps and tool calls, based on the task and the results so far.

MAYA: So agents are the more advanced option, and I should aim for those.

BEN: [warm] That's the misconception the source warns about. Begin with a simple approach, and add autonomy when the task benefits from it. Track six covers these designs in depth.

MAYA: Can you give me an example where tools really make a difference?

BEN: A coding assistant can propose a fix, run a test, and revise the fix using the failure output. The test provides evidence about whether its answer works.

MAYA: And that connects back to hallucinations.

BEN: It does. A confident explanation alone cannot provide that evidence. A test can. That idea runs through the whole course.

MAYA: Let me try a mini-summary. The model predicts tokens. Training changes its weights, chatting doesn't. It only works from what's in the context window. And tools let it act, and give it evidence instead of just confidence.

BEN: That's the mental model for the whole course. Well done.

MAYA: The chapter ends with a quick check. What's that?

BEN: A small exercise. Ask an assistant to summarise a short document you provide. Then ask it to support one claim with an exact passage from that document, and check the passage yourself.

MAYA: And then?

BEN: Identify three things. The context you supplied, the output it generated, and the check you used. And if the assistant searched or opened a file, identify that tool call too.

MAYA: Nice, that's the whole primer in one exercise. Okay, now the course itself. How is a chapter built?

BEN: Each chapter folder has an article and a slide deck. The article has a short summary at the top, the explanation, a section on what lasts and what will date, exercises, key terms and sources.

MAYA: And the slides are just the same thing in another format?

BEN: The same chapter as a deck. The suggestion is to read the article, and use the deck for review, or to teach the chapter to others. And the quickest review is that "what lasts versus what will date" section.

MAYA: Now the map. How many chapters are we talking about?

BEN: Track zero, which is this introduction, plus thirty chapters, in eight tracks. Track one is using models, chapters one and two. Track two is context engineering, three to five.

MAYA: Keep going.

BEN: Track three is working with coding agents, six to eight. Track four is tools, protocols and capabilities, nine to fifteen. Things like MCP, Skills and {A2A|A two A}, and the security risks they bring.

MAYA: And the rest?

BEN: Track five is building AI features, sixteen to twenty: RAG, evals, cost and privacy. Track six is agent architecture, twenty-one to twenty-seven. And Track seven, the bigger picture, is twenty-eight to thirty.

MAYA: Are there ideas that tie all of that together?

BEN: Five threads run through every track. Context is a budget. Checks beat hope. Agents act with your permissions. Measure before you optimize. And the simplest thing that works.

MAYA: [thoughtful] "Checks beat hope" sounds a lot like the test example.

BEN: It is. And the slides suggest a trick: when a new tool appears, ask which of these threads it touches. That usually tells you how to use it well.

MAYA: So, where should someone start? Do I have to listen to all thirty?

BEN: Depends on your goal. If you want to use AI assistants well, starting today, read Tracks zero to three. To add tools and build AI features, add Tracks four and five. To design multi-step and multi-agent systems, add Track six.

MAYA: And Track seven?

BEN: Track seven is for understanding the wider landscape, and you can read it any time. Everything else, work through in order, because later chapters assume earlier ones.

MAYA: Any tips for getting the most out of it?

BEN: Four. Do the exercises, because each one makes a single principle visible in your own work. Try ideas on a real task, not a toy example. Follow the sources when you need depth or the latest details.

MAYA: That's three.

BEN: [laughs] And check the dates. The course was compiled in October 2026, and fast-changing facts are marked "as of October 2026". Those may have changed. The principle behind them usually hasn't.

MAYA: Okay, let me recap, and you correct me. The course teaches principles, because tools change every month and principles change slowly. An LLM predicts the next token, one at a time. That's also why it can hallucinate.

BEN: Good so far.

MAYA: Pre-training and post-training change the weights. Inference, so chatting, only changes the input. Everything has to fit in the context window, and memory is the app selecting what to put in. Tools let the model act, and an agent picks its own next steps.

BEN: One small correction. An agent lets the model choose more of its next steps. And start simple before you reach for one.

MAYA: Got it. Quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You correct an assistant in a normal chat. Has the model learned from your correction?

PAUSE: 5

MAYA: No. My correction changes the input for this conversation, but it doesn't update the model's weights. Only training does that.

BEN: Exactly. Question two. A coding assistant gives you a fluent, confident explanation of why its fix works. Why isn't that enough, and what is better?

PAUSE: 5

MAYA: Because fluent output can still contain invented facts. A better sign is a test it can run. The test gives evidence about whether the fix works, and a confident explanation can't.

BEN: Perfect. And last one. What's the difference between a workflow and an agent?

PAUSE: 5

MAYA: A workflow follows steps defined in code. An agent lets the model choose more of its next steps and tool calls, based on the results so far. And you start simple, and only add autonomy when the task benefits from it.

BEN: Three out of three.

MAYA: [cheerful] And for homework, do the quick check: have an assistant summarise a document, ask it for an exact supporting passage, and check that passage yourself.

BEN: Next time, in chapter one, Getting better LLM answers: clear instructions, real material to work from, and managing the context window.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter one.
