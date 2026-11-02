# Introduction and Track 0: How LLMs work

This course teaches the durable principles of working with and building on large language models
(LLMs). Start with this short primer, then follow the tracks in the [course map](https://ai-fundamentals.kwise.app/).
The [introduction slides](slides.md) cover the same basics and the learning paths through the course.

## Tokens, prediction and attention

An LLM generates text one **token** at a time. Tokens are pieces of text: a word, part of a word or
punctuation. Given an input, the model calculates probabilities for the next token. It selects one,
adds it to the sequence and repeats [1][2].

For example, after “The capital of France is”, it may assign a high probability to “Paris”. The same
process can produce an explanation, code or a conversation. Fluent output can still contain invented
facts, often called **hallucinations** [1][3].

Inside a transformer, **embeddings** represent tokens as lists of numbers. **Attention** lets the model
combine information from tokens in its context. This helps it interpret a word in relation to the
surrounding text, such as “bank” beside “river” [2].

## Training and answering are different stages

During **pre-training**, a model learns to predict tokens from large amounts of data. Training adjusts
its **weights**, the numerical parameters that encode learned patterns. This produces broad language
capabilities and knowledge, with gaps and errors [1][3].

**Post-training** shapes those capabilities into assistant behaviour. Supervised fine-tuning uses
examples of desired responses. Reinforcement learning uses reward signals, such as human preferences
or checks on whether a solution works. These stages improve behaviour without guaranteeing correct
answers [3].

**Inference** is using the trained model to generate an answer. In an ordinary conversation, your
messages change the input it works from. They do not update its weights [1][3].

## Context is the model's working material

The **context window** limits how many tokens a model can work with at once. Instructions, conversation
history and supplied documents all take space. Information outside that window must be supplied again
or retrieved by the application [2][3].

A chat product may save history or offer memory, but the application still has to select what to give
the model on each call. This is why clear instructions and relevant source material matter, and why
long tasks need deliberate context management [3]. Tracks 1–2 develop those habits.

## Tools and agents add actions

An application can give a model **tools**, such as search, a calculator or a way to run tests. The model
requests a tool call, the application executes it, and the result becomes input for another model call.
Tools give the system access to information and actions beyond text generation [4].

A **workflow** follows steps defined in code. An **agent** lets the model choose more of its next steps
and tool calls based on the task and the results so far. Begin with a simple approach and add autonomy
when the task benefits from it [4]. Track 6 covers these designs in depth.

For example, a coding assistant can propose a fix, run a test and revise the fix using the failure
output. The test provides evidence about whether its answer works. A confident explanation alone
cannot provide that evidence.

## A quick check before Track 1

Ask an assistant to summarise a short document you provide. Then ask it to support one claim with an
exact passage from that document, and check the passage yourself. Identify the **context** you supplied,
the **output** it generated and the **check** you used. If the assistant searched or opened a file,
identify that tool call too.

Continue with [Getting better LLM answers](../01-better-llm-answers/article.md).

## Sources and optional deeper reading

These are the sources behind Track 0. Watching the full videos is optional.

1. [Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g) (Andrej Karpathy, 2023). Overview of prediction, training and inference.
2. [But what is a GPT?](https://www.3blue1brown.com/lessons/gpt) (3Blue1Brown, 2024). Visual explanation of tokens, embeddings and transformers, with a follow-up on attention.
3. [Deep Dive into LLMs like ChatGPT](https://www.youtube.com/watch?v=7xTGNNLPyMI) (Andrej Karpathy, 2025). Training stages, hallucinations, context and tool use.
4. [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Anthropic, 2024). Tools, workflows and agents. Revisit before Track 6.
