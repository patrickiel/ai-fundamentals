# Embeddings and RAG: Giving a Model the Right Pages to Read

> **Core idea in one sentence:** Retrieval-augmented generation (RAG) is a search system feeding a language model, so its answers can only be as good as the passages it retrieves, and the work that matters is chunking, hybrid search, and measuring retrieval on its own.

## TL;DR

- An **embedding** turns a piece of text into a list of numbers (a vector). Texts with similar meanings end up close together, so "how related are these?" becomes a distance calculation [2].
- **RAG** has two halves. *Offline*, you chunk documents, embed the chunks and index them. *At query time*, you retrieve the most relevant chunks, optionally rerank them, and put them in the prompt so the model can answer from them and cite them [1][3].
- **Embeddings alone miss exact matches** such as IDs, error codes and product names. **Hybrid search** (keyword/BM25 plus vectors) is the usual fix [3][5].
- Most RAG failures are **retrieval failures**: bad chunk boundaries, chunks that lost their context, a stale index, or relevant text buried in the middle of a long prompt [3][4][7].
- **Evaluate retrieval separately** from generation, using search metrics such as recall@k, precision@k and MRR [6].
- RAG is not always the right tool. **Small corpora can go straight into the prompt** [3], and **coding agents often do better by navigating files with tools like grep** than by querying a vector index [7][8].

## Why retrieval exists at all

A language model knows only what was in its training data, and it stores that knowledge in its weights. The original RAG paper (Lewis et al., 2020) called this **parametric memory** and set out its weaknesses: it is hard to update, and it is hard to see where an answer came from [1]. The paper paired the model with a **non-parametric memory**, a searchable index of documents (Wikipedia, in their case) that a retriever queries before the model generates. They reported more specific and more factual answers than the model alone produced [1]. The lasting idea: knowledge in an index can be updated by re-indexing, kept private, and cited.

## Embeddings: meaning as coordinates

An embedding model takes text and returns a fixed-length vector of floating-point numbers. The distance between two vectors indicates how related the two texts are [2]. "How do I reset my password?" and "I forgot my login credentials" share almost no words, yet their vectors land close together. That is the main thing embeddings add over keyword search.

Practical facts, using OpenAI's docs as one example [2]:

- **Similarity:** cosine similarity is the usual measure, and the choice of distance function rarely matters much.
- **Dimensions:** longer vectors cost more to store and compare; some models let you shorten them with modest quality loss.
- **Input limits:** each model has a maximum input length, one reason documents are chunked rather than embedded whole.

General background: query and documents must be embedded with the **same model**. Vectors from different models are not comparable, so switching models means re-embedding everything.

## The RAG pipeline, step by step

Anthropic describes the standard pipeline in three preprocessing steps plus a query step [3]. Adding the optional pieces gives this:

| Stage | What happens | Main decision |
|---|---|---|
| 1. Chunk | Split documents into passages, usually no more than a few hundred tokens each [3] | Size, boundaries, overlap |
| 2. Embed | Turn each chunk into a vector | Which embedding model |
| 3. Index | Store vectors (and a keyword index) for fast search | Vector database; also BM25? |
| 4. Retrieve | Embed the query; find the nearest chunks | How many (top-k) |
| 5. Rerank (optional) | A second model scores query–chunk pairs and reorders them [3] | Latency vs. accuracy |
| 6. Generate | Put the top chunks in the prompt; the model answers and cites them | Prompt layout, citations |

**Each stage can lose information no later stage can recover.** A model can sometimes ignore an irrelevant chunk, but it cannot answer from one it never saw. That is why Hamel Husain and Shreya Shankar advise debugging retrieval first [6].

**Citations come from the pipeline.** You know which document each chunk came from, so you can label chunks in the prompt and ask the model to cite them. That makes answers checkable, the "provenance" the original paper wanted [1].

## Hybrid search: meaning plus exact words

Embeddings capture meaning but can miss exact matches [3]. Someone searching for error code `TS-999` wants that exact string, not passages about error codes in general. Eugene Yan lists the same blind spots: names, acronyms and IDs [5].

**BM25** is a classic keyword ranking function: it scores documents by how often the query terms appear, weighted by how rare those terms are, adjusted for document length [3]. It is strong where embeddings are weak and weak where they are strong; it doesn't know "car" and "automobile" are related [5].

**Hybrid search** runs both, merges and deduplicates the results (for example with rank fusion), and keeps the top-k [3]. Yan reports the combination works better than either alone [5].

## Contextual retrieval: fixing chunks that forgot where they came from

A chunk can lose the context that made it meaningful. Anthropic's example is a chunk saying "the company's revenue grew by 3% over the previous quarter" without naming the company or the quarter [3]. Neither index can match it to a question about one company's Q2 revenue.

**Contextual retrieval** fixes this at indexing time. A model reads the whole document and writes a short note (typically 50–100 tokens) situating each chunk, which is prepended before embedding and before building the BM25 index [3].

In Anthropic's tests (September 2024), measured as the share of relevant chunks *missing* from the top 20 results [3]:

| Setup | Retrieval failure rate |
|---|---|
| Plain embeddings | 5.7% |
| + contextual embeddings | 3.7% |
| + contextual BM25 (hybrid) | 2.9% |
| + reranking | 1.9% |

The numbers are specific to their datasets and models. The durable lessons: the techniques **stack**, and context added once at indexing time is cheap compared with retrieval failing on every query. In their setup 20 chunks beat 5 or 10, though they warn more chunks can also distract the model [3].

## Where RAG systems fail

| Failure | What goes wrong | Typical fix |
|---|---|---|
| **Bad chunking** | Boundaries split an answer across chunks, or chunks lose their context [3] | Tune size, boundaries and overlap; add context to chunks |
| **Retrieval misses** | The relevant passage exists but isn't in the top-k, often for exact terms [3][5] | Hybrid search, reranking, larger k, query rewriting |
| **Stale index** | Source documents changed but embeddings were not rebuilt [7] | Re-index on change; track document versions |
| **Lost in the middle** | The right passage is in the prompt but the model overlooks it [4] | Fewer, better-ranked chunks; put the best near the start or end |
| **Unfaithful answer** | The model ignores or contradicts the retrieved text [6] | Test faithfulness with an evaluation; require citations |

On "lost in the middle" (Liu et al., 2023): in multi-document question answering, models did best when the relevant passage was at the **beginning or end** of the input and worse when it was in the middle, even long-context models [4]. Stuffing in more chunks is not free (see topic 04, Context rot).

## Evaluating retrieval

Husain and Shankar's advice is to treat retrieval as **a search problem** and measure it with standard information-retrieval metrics, separately from the generated answer [6]:

- **Recall@k**: what fraction of the relevant chunks appear in the top k? Usually the first number to watch, since an unretrieved chunk can never be used.
- **Precision@k**: what fraction of the k retrieved chunks are relevant? Low precision wastes context.
- **MRR (mean reciprocal rank)**: on average, how high does the first relevant chunk rank?

For test data without hand-labelling, take documents from your corpus, extract key facts, and generate questions those facts answer, so each question has a known correct chunk [6]. Also check against real user queries, which are usually messier than synthetic ones.

For the generation half they suggest checking **context relevance** (is the context relevant to the question?), **faithfulness** (is the answer supported by the context?) and **answer relevance** (does it address the question?), using LLM judges only after validating them against human labels [6]. Topic 18 (Evals) covers this in depth.

## When RAG is the wrong tool

### The corpus fits in the prompt

Anthropic's guidance (2024) is that a knowledge base under about **200,000 tokens** (roughly 500 pages) can simply go in the prompt, with prompt caching to make repeated calls cheaper and faster [3]. No chunking, no index, nothing to go stale. The threshold will move, but the rule holds: **retrieval is a workaround for limited context, so check whether you need it.** Long prompts degrade too (topic 04), so "fits" is not the same as "works well". Measure both.

### Code, and other structured worlds an agent can explore

Anthropic describes Claude Code as working "just in time": it keeps lightweight references such as file paths and uses `glob` and `grep` to load files as needed, which sidesteps **stale indexes** [7]. Jason Liu reports, second-hand, that Claude Code's creator said on a podcast the team tried RAG early and moved to agentic exploration [8]. Liu's reasoning: code is structured, so an agent can follow imports and folder names like an engineer, while retrieved snippets arrive cut off from their surroundings [8].

This is the contrast drawn in topic 21 (Harness over model). The trade-offs cut both ways:

| | Pre-built index (RAG) | Agentic search |
|---|---|---|
| Speed | Fast lookup of precomputed data [7] | Slower: several tool calls per question [7] |
| Freshness | Can go stale [7] | Always reads the current files |
| Exact identifiers | Needs hybrid search | `grep` is exact by default |
| Concept queries ("where do we handle auth?") | Good at matching meaning | Depends on naming and the agent's guesses |
| Best fit | Large, slow-changing, unstructured text | Navigable structure: code, file trees |

Anthropic suggests a **hybrid** may be best in some settings, and notes that pre-computed retrieval suits less dynamic content such as legal or finance work [7]. Liu's post lists cases where retrieval still makes sense: tight token budgets, high-volume low-stakes work, and very large unstructured data collections [8].

## What lasts vs. what will date

- **Durable principles:**
  - Embeddings turn meaning into distance.
  - RAG quality is capped by retrieval quality; measure retrieval first.
  - Keyword and semantic search fail differently, so combine them.
  - Chunks need enough context to stand alone; indexes need a refresh plan.
  - Position in a long prompt affects whether a passage gets used.
  - Retrieval is a workaround: if the material fits, or an agent can navigate it, consider skipping the index.
- **Time-sensitive details:**
  - Anthropic's failure-rate numbers and model choices (September 2024).
  - The ~200k-token "just put it in the prompt" threshold, which depends on current context windows and caching.
  - Specific embedding models, dimensions and input limits.
  - Which coding tools use vector indexes versus agentic search (as of 2025–2026).

## Try it yourself

1. **See similarity.** Embed ten short sentences with any embedding API or local model, including two paraphrases with no shared words. Compute cosine similarity for every pair and check that the paraphrases score highest.
2. **Break embeddings on purpose.** Add a document containing an ID like `INV-48213`. Search for that ID with embeddings only, then with BM25 only. Note which one finds it.
3. **Build a mini retrieval eval.** For 20 chunks of your own documents, generate one question each. Measure recall@5 for your current setup. Change one thing (chunk size, hybrid search or added context) and measure again.
4. **Test the middle.** Put one relevant passage among 20 irrelevant ones, first at the start, then in the middle, then at the end. Ask the same question each time and compare the answers.
5. **Check whether you need RAG.** Count the tokens in a corpus you would normally index. If it fits comfortably in context, compare answers from "whole corpus in prompt" against your RAG pipeline on the same ten questions.

## Key terms

- **Embedding**: a fixed-length vector representing a text so that related texts are close together.
- **Cosine similarity**: a measure of how closely two vectors point in the same direction; the common way to compare embeddings.
- **Chunking**: splitting documents into passages small enough to embed and retrieve individually.
- **BM25**: a keyword ranking function based on term frequency and term rarity; strong for exact matches.
- **Hybrid search**: running keyword and vector search together and merging the results.
- **Reranking**: a second, more careful scoring pass that reorders the retrieved chunks before generation.
- **Contextual retrieval**: prepending a short, model-written description of where a chunk sits in its document before indexing it.
- **Recall@k**: the fraction of relevant items that appear in the top k results.
- **Agentic search**: an agent finding information by exploring with tools (list, grep, open) at run time instead of querying a pre-built index.

## Sources

1. [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) (Lewis et al., NeurIPS 2020; arXiv May 2020). The original RAG paper: parametric vs. non-parametric memory, provenance and updating knowledge as motivations.
2. [Vector embeddings guide](https://developers.openai.com/api/docs/guides/embeddings) (OpenAI docs, accessed 2026-10). What an embedding is, cosine similarity, normalisation, shortening dimensions, input limits, use cases.
3. [Introducing Contextual Retrieval](https://www.anthropic.com/engineering/contextual-retrieval) (Anthropic, 2024-09-19). The standard pipeline, BM25 and hybrid search, contextual embeddings, reranking, failure-rate results, the 200k-token guidance.
4. [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172) (Liu et al., TACL; arXiv July 2023). The U-shaped effect of passage position on accuracy.
5. [Patterns for Building LLM-based Systems & Products](https://eugeneyan.com/writing/llm-patterns/) (Eugene Yan, 2023). Where keyword search beats embeddings and the case for hybrid retrieval.
6. [Q: How should I approach evaluating my RAG system?](https://hamel.dev/blog/posts/evals-faq/how-should-i-approach-evaluating-my-rag-system.html) (Hamel Husain & Shreya Shankar, 2025-06, updated 2026-09). Evaluating retrieval separately with recall@k, precision@k and MRR; synthetic query generation; generation checks.
7. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Anthropic, 2025-09-29). Pre-computed retrieval vs. just-in-time agentic search, stale indexes, hybrid strategies, speed trade-offs.
8. [Why I stopped using RAG for coding agents (and you should too)](https://jxnl.co/writing/2025/09/11/why-i-stopped-using-rag-for-coding-agents-and-you-should-too/) (Jason Liu, 2025-09-11). Practitioner case for agentic exploration in code, the second-hand Claude Code account, and where RAG still fits.
