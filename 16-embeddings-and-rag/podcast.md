<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 16: Embeddings and RAG

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter sixteen: embeddings and {RAG|rag}. Giving a model the right pages to read. And it's the start of a new track, Track five, building AI features.

MAYA: Last time, in chapter fifteen, we did the {A2A|A two A} protocol. Agents publishing Agent Cards so other agents can find them and hand them tasks.

BEN: Right. That was agents talking to agents. Today we step back to something more basic. How do you get a model to answer from your documents, not just from what it learned in training?

MAYA: And the answer is {RAG|rag}. Which I've heard a thousand times and could not define if you paid me.

BEN: [laughs] Let's fix that. Here's the one sentence for today. Retrieval-augmented generation, {RAG|rag}, is a search system feeding a language model. So its answers can only be as good as the passages it retrieves. And the work that matters is chunking, hybrid search, and measuring retrieval on its own.

MAYA: A search system feeding a model. Okay. So it's really two systems glued together.

BEN: That's the right way to think about it. Let's start with why we need the search part at all.

MAYA: Go on.

BEN: A language model knows only what was in its training data, and it stores that knowledge in its weights. The original {RAG|rag} paper, by Lewis and colleagues in 2020, called this parametric memory. And it set out the weaknesses. It's hard to update, and it's hard to see where an answer came from.

MAYA: So if a fact changes, I can't just edit the model.

BEN: Exactly. So the paper paired the model with what it called a non-parametric memory. A searchable index of documents, Wikipedia in their case, that a retriever queries before the model generates. They reported more specific and more factual answers than the model alone produced.

MAYA: And the lasting idea is?

BEN: Knowledge in an index can be updated by re-indexing, kept private, and cited. Those three things are why people still build these systems.

MAYA: Okay. Now embeddings. I know the word. I don't know the thing.

BEN: An embedding model takes text and returns a fixed-length list of numbers. A vector. And the distance between two vectors indicates how related the two texts are.

MAYA: [curious] So meaning becomes a position. Like coordinates on a map.

BEN: That's a good analogy, and it's roughly how the chapter frames it: meaning as coordinates. Here's the example. "How do I reset my password?" and "I forgot my login credentials". They share almost no words. But their vectors land close together.

MAYA: Oh, and keyword search would never connect those.

BEN: Right. That's the main thing embeddings add over keyword search. A few practical facts, using OpenAI's docs as one example. Cosine similarity is the usual way to measure closeness, and the choice of distance function rarely matters much.

MAYA: Okay, what else?

BEN: Dimensions. Longer vectors cost more to store and compare, and some models let you shorten them with modest quality loss. And input limits. Each model has a maximum input length, which is one reason documents are chunked rather than embedded whole.

MAYA: So I can't just embed a whole book as one vector.

BEN: Not usefully. And one more rule that catches people. The query and the documents must be embedded with the same model. Vectors from different models aren't comparable. So if you switch embedding models, you re-embed everything.

MAYA: Ouch. Okay, so how does the whole pipeline fit together?

BEN: Anthropic describes the standard pipeline as three preprocessing steps plus a query step. The chapter adds the optional pieces, which gives six stages. First, chunk. You split documents into passages, usually no more than a few hundred tokens each. The decision there is size, boundaries and overlap.

MAYA: Second?

BEN: Embed. Turn each chunk into a vector, and decide which embedding model. Third, index. Store the vectors, and maybe a keyword index too, for fast search. Those three happen offline, before anyone asks anything.

MAYA: And then a user asks a question.

BEN: Then step four, retrieve. You embed the query and find the nearest chunks. The decision is how many, the top K. Step five is optional: rerank. A second model scores each query and chunk pair, and reorders them. That's a trade between latency and accuracy.

MAYA: And six is the model finally answering.

BEN: Generate. You put the top chunks in the prompt, and the model answers and cites them.

MAYA: [thoughtful] So, chunk, embed, index offline. Then retrieve, maybe rerank, and generate when someone asks.

BEN: Perfect summary. And here's the key insight. Each stage can lose information that no later stage can recover. A model can sometimes ignore an irrelevant chunk. But it can't answer from one it never saw.

MAYA: So if retrieval misses it, the model has no chance.

BEN: None. That's why Hamel Husain and Shreya Shankar advise debugging retrieval first.

MAYA: You mentioned citations. Does the model just, know where things came from?

BEN: No, and this is a nice point. Citations come from the pipeline. You know which document each chunk came from, so you can label the chunks in the prompt and ask the model to cite them. That makes answers checkable. It's the provenance the original paper wanted.

MAYA: Okay. Now, embeddings sound great. What's the catch?

BEN: They capture meaning, but they can miss exact matches. Say someone searches for error code {TS-999|T S nine nine nine}. They want that exact string. Not passages about error codes in general.

MAYA: Huh. I'd have assumed a fancy embedding would handle that better than plain search.

BEN: That's a common misconception. Eugene Yan lists the same blind spots: names, acronyms and IDs. And the classic fix is a keyword ranking function called {BM25|B M twenty-five}.

MAYA: Which does what?

BEN: It scores documents by how often the query terms appear, weighted by how rare those terms are, and adjusted for document length. It's strong where embeddings are weak, and weak where they're strong. It doesn't know that "car" and "automobile" are related.

MAYA: So they fail in opposite ways.

BEN: Exactly. So hybrid search runs both. It merges and deduplicates the results, for example with rank fusion, and keeps the top K. Yan reports that the combination works better than either alone.

MAYA: Okay. What's contextual retrieval? That sounds like a buzzword.

BEN: [laughs] It's a real fix for a real problem. A chunk can lose the context that made it meaningful. Anthropic's example is a chunk that says, quote, "the company's revenue grew by {3%|three percent} over the previous quarter". But it doesn't name the company, or the quarter.

MAYA: So if I ask about one company's second-quarter revenue, nothing matches.

BEN: Neither the vector index nor the keyword index can match it. Contextual retrieval fixes this at indexing time. A model reads the whole document and writes a short note situating each chunk, typically fifty to a hundred tokens. That note gets prepended before embedding, and before building the {BM25|B M twenty-five} index.

MAYA: Does it actually help? Or is this one of those "trust us" things?

BEN: They measured it, in tests from September 2024. The measure was the share of relevant chunks missing from the top twenty results. With plain embeddings, five point seven percent were missing. Adding contextual embeddings brought that to three point seven. Adding contextual {BM25|B M twenty-five}, so hybrid, brought it to two point nine. And adding reranking on top, one point nine.

MAYA: So each technique stacks on the last one.

BEN: That's the durable lesson. The numbers are specific to their datasets and models. But the techniques stack, and context added once at indexing time is cheap compared with retrieval failing on every query.

MAYA: How many chunks did they hand the model?

BEN: In their setup, twenty chunks beat five or ten. Though they warn that more chunks can also distract the model.

MAYA: Which sounds a lot like context rot.

BEN: It's a close cousin, yes. Let's go through where these systems fail, because the chapter lists five failures. The first is bad chunking. Boundaries split an answer across chunks, or the chunks lose their context. The fix is to tune size, boundaries and overlap, and add context to chunks.

MAYA: Second?

BEN: Retrieval misses. The relevant passage exists, but isn't in the top K, often for exact terms. Fixes are hybrid search, reranking, a larger K, or rewriting the query.

MAYA: Third?

BEN: A stale index. The source documents changed, but nobody rebuilt the embeddings. So you re-index on change, and track document versions.

MAYA: [sighs] That one feels like it would happen to me in week two.

BEN: [laughs] It happens to everyone. Fourth is lost in the middle. The right passage is in the prompt, but the model overlooks it. That's the Liu and colleagues paper from 2023. In multi-document question answering, models did best when the relevant passage was at the beginning or end of the input, and worse in the middle. Even long-context models.

MAYA: We met that one in chapter four. And the fix?

BEN: Fewer, better-ranked chunks, with the best near the start or the end. Stuffing in more chunks is not free. And the fifth failure is an unfaithful answer. The model ignores or contradicts the retrieved text. You test faithfulness with an evaluation, and you require citations.

MAYA: Let me check I've got it. Bad chunking, retrieval misses, stale index, lost in the middle, unfaithful answer. And four of those five happen before the model even starts answering. It's not the model being dumb.

BEN: That's a fair reading. The chapter's own summary is that most {RAG|rag} failures are retrieval failures: bad chunk boundaries, chunks that lost their context, a stale index, or relevant text buried in the middle of a long prompt.

MAYA: So how do I measure retrieval?

BEN: Husain and Shankar's advice is to treat retrieval as a search problem, and measure it with standard information-retrieval metrics, separately from the generated answer. There are three to know. First, {recall@k|recall at K}. What fraction of the relevant chunks appear in the top K?

MAYA: So, did we find the stuff at all.

BEN: Yes. And it's usually the first number to watch, since a chunk you didn't retrieve can never be used. Second, {precision@k|precision at K}. What fraction of the K retrieved chunks are relevant? Low precision wastes context.

MAYA: And the third?

BEN: {MRR|M R R}, mean reciprocal rank. On average, how high does the first relevant chunk rank?

MAYA: Okay, but to measure recall, I need to know the right answer for each question. Do I have to label everything by hand?

BEN: Not necessarily. Take documents from your corpus, extract key facts, and generate questions those facts answer. Then each question has a known correct chunk. But also check against real user queries, because they're usually messier than synthetic ones.

MAYA: And the generation half, the answer itself?

BEN: They suggest three checks. Context relevance: is the context relevant to the question? Faithfulness: is the answer supported by the context? And answer relevance: does it actually address the question? You can use {LLM|L L M} judges for those, but only after validating them against human labels. Chapter eighteen, on evals, covers that in depth.

MAYA: Okay. Now the part I'm curious about. When is {RAG|rag} the wrong tool?

BEN: Two cases. The first is when the corpus fits in the prompt. Anthropic's guidance from 2024 is that a knowledge base under about two hundred thousand tokens, roughly five hundred pages, can simply go in the prompt. With prompt caching, to make repeated calls cheaper and faster.

MAYA: No chunking, no index.

BEN: And nothing to go stale. The threshold will move, but the rule holds. Retrieval is a workaround for limited context, so check whether you need it.

MAYA: [thoughtful] But wait. Chapter four said long prompts degrade too. So "it fits" doesn't mean "it works".

BEN: Exactly the caveat the chapter makes. "Fits" is not the same as "works well". Measure both.

MAYA: And the second case?

BEN: Code, and other structured worlds an agent can explore. Anthropic describes Claude Code as working "just in time". It keeps lightweight references, like file paths, and uses glob and grep to load files as needed. Which sidesteps stale indexes.

MAYA: So it just searches the files live, every time.

BEN: Right. And Jason Liu reports, second-hand, that Claude Code's creator said on a podcast that the team tried {RAG|rag} early, and moved to agentic exploration. Liu's reasoning is that code is structured. An agent can follow imports and folder names like an engineer would, while retrieved snippets arrive cut off from their surroundings.

MAYA: So should I just never use a vector index for code?

BEN: The trade-offs cut both ways. A pre-built index gives fast lookup of precomputed data. Agentic search is slower, several tool calls per question. But the index can go stale, while agentic search always reads the current files.

MAYA: What about my error code example?

BEN: For exact identifiers, an index needs hybrid search, while grep is exact by default. But for concept queries, like "where do we handle auth?", an index is good at matching meaning. Agentic search depends on naming, and on the agent's guesses.

MAYA: So, index for large, slow-changing, unstructured text. Agentic search for things with navigable structure, like code and file trees.

BEN: That's how the chapter sums it up. And Anthropic suggests a hybrid may be best in some settings. They note that pre-computed retrieval suits less dynamic content, such as legal or finance work. And Liu lists cases where retrieval still makes sense: tight token budgets, high-volume low-stakes work, and very large unstructured data collections. Chapter twenty-one, the harness matters as much as the model, comes back to this contrast.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Embeddings turn meaning into distance. {RAG|rag} quality is capped by retrieval quality, so measure retrieval first. Keyword and semantic search fail differently, so combine them. Chunks need enough context to stand alone, and indexes need a refresh plan. Position in a long prompt affects whether a passage gets used. And retrieval is a workaround. If the material fits, or an agent can navigate it, consider skipping the index.

MAYA: And what will date?

BEN: Anthropic's failure-rate numbers and model choices, from September 2024. The roughly two-hundred-thousand-token threshold for just putting it in the prompt, which depends on current context windows and caching. Specific embedding models, dimensions and input limits. And which coding tools use vector indexes versus agentic search, as of 2025 and 2026.

MAYA: Let me try to sum up, and you correct me. An embedding turns text into a vector, so related meanings end up close together. {RAG|rag} chunks, embeds and indexes documents offline, then retrieves, maybe reranks, and puts the best chunks in the prompt so the model can answer and cite.

BEN: Good so far.

MAYA: Embeddings miss exact things like IDs, so you add {BM25|B M twenty-five} and do hybrid search. Contextual retrieval adds a short note to each chunk so it doesn't forget where it came from. Most failures are retrieval failures, so I measure retrieval on its own, with {recall@k|recall at K} first. And sometimes I don't need {RAG|rag} at all: if it fits in the prompt, or if it's code an agent can explore.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your documents contain an invoice ID like {INV-48213|I N V four eight two one three}. Users search for it, and your embeddings-only setup can't find it. Why, and what's the fix?

PAUSE: 5

MAYA: Because embeddings capture meaning, and they can miss exact strings like IDs, names and acronyms. The fix is hybrid search: add {BM25|B M twenty-five} keyword search, and merge the results with the vector results.

BEN: Perfect. Question two. Your {RAG|rag} answers are bad. Which half do you debug first, and what's the first number to watch?

PAUSE: 5

MAYA: Retrieval first, because the model can't answer from a chunk it never saw. And the first number is {recall@k|recall at K}: what fraction of the relevant chunks show up in the top K.

BEN: Exactly. And last one. Your whole knowledge base is about a hundred pages. Do you need a {RAG|rag} pipeline?

PAUSE: 5

MAYA: Probably not. That's well under the roughly two hundred thousand tokens Anthropic mentions, so it can go straight in the prompt, with prompt caching. But fitting isn't the same as working well, so I'd compare it against a {RAG|rag} pipeline on the same questions.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I like the second one: put an ID in a document, search for it with embeddings only and then with {BM25|B M twenty-five} only, and see which one finds it.

BEN: And try the others too. Check that paraphrases score highest on cosine similarity, build a mini retrieval eval with {recall@5|recall at five}, test a passage at the start, middle and end of a long prompt, and check whether your corpus needs {RAG|rag} at all. Next time, in chapter seventeen: prompting versus {RAG|rag} versus fine-tuning. Which lever changes what, and how to choose.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter seventeen.
