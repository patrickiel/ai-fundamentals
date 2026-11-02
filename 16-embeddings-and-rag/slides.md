---
theme: ../theme
chapter: '16'
header: 'AI Fundamentals · 16 · Embeddings & RAG'
layout: cover
---

# Embeddings and RAG

**RAG is a search system feeding a model: answers are only as good as what it retrieves**

---

## The problem

- A model's knowledge is frozen in its **weights** at training time
- Weights are hard to update and can't say **where** an answer came from
- Your private, recent or large documents aren't in there
- RAG (Lewis et al., 2020): pair the model with a **searchable index**
- Update by re-indexing; cite the passage used

<!-- The original paper called these parametric and non-parametric memory. The architecture has changed since; the motivation hasn't. -->

---

## Embeddings: meaning as coordinates

- Text in, **fixed-length vector** out
- Close vectors = related meaning
- "Reset my password" ≈ "forgot my login", with no shared words
- Compare with **cosine similarity**
- Query and documents must use the **same model**

<!-- Switching embedding models means re-embedding the whole corpus, because vectors from different models are not comparable. -->

---

## The pipeline

| Stage | Key decision |
|---|---|
| 1. **Chunk** documents | size, boundaries, overlap |
| 2. **Embed** chunks | which model |
| 3. **Index** (vectors + keywords) | vector DB, BM25 |
| 4. **Retrieve** top-k for the query | how many |
| 5. **Rerank** (optional) | latency vs. accuracy |
| 6. **Generate** with citations | prompt layout |

<!-- Steps 1–3 happen offline; 4–6 per query. Each stage can lose information no later stage can recover. -->

---

## Hybrid search

- Embeddings match meaning but **miss exact strings**
  - IDs, error codes, names, acronyms (`TS-999`)
- **BM25** keyword ranking: exact terms, weighted by rarity
  - but doesn't know "car" ≈ "automobile"
- **Hybrid**: run both, merge (rank fusion), keep top-k
- Practitioners report it beats either alone

<!-- The two methods fail in different places, which is exactly why combining them helps. -->

---

## Contextual retrieval

- Chunks lose context: "revenue grew 3%" (which company? which quarter?)
- Fix at index time: a model writes a **50–100 token note** per chunk
- Prepend it before embedding **and** before BM25

| Anthropic test (Sept 2024) | Missed in top 20 |
|---|---|
| Plain embeddings | 5.7% |
| + contextual embeddings + BM25 | 2.9% |
| + reranking | 1.9% |

<!-- Exact numbers are dataset-specific. The lesson is that the techniques stack, and indexing-time context is a one-off cost. -->

---

## Where RAG fails

| Failure | Fix |
|---|---|
| **Bad chunking** | tune size/overlap; add context |
| **Retrieval miss** | hybrid, rerank, larger k |
| **Stale index** | re-index on change |
| **Lost in the middle** | fewer, better-ranked chunks |
| **Unfaithful answer** | faithfulness eval; citations |

<!-- Most failures happen before the model writes a word. If the passage isn't retrieved, nothing downstream can save the answer. -->

---

## Lost in the middle

- Liu et al. (2023): accuracy is **U-shaped** over position
- Best when the relevant passage is at the **start or end**
- Worse in the middle, even for long-context models
- More chunks ≠ better answers
- Links to topic 04, **Context rot**

<!-- Put the best-ranked chunks where the model is most likely to use them, and don't pad the prompt. -->

---

## Evaluate retrieval on its own

- Retrieval is **a search problem**: use search metrics
  - **Recall@k**: share of relevant chunks in the top k
  - **Precision@k**: share of top k that are relevant
  - **MRR**: how high the first relevant hit ranks
- Test set: generate questions from facts in each chunk
- Then judge answers: context relevance, **faithfulness**, answer relevance

<!-- Recall comes first: a chunk that is never retrieved can never be used. See topic 18 for judges and evals. -->

---

## When not to use RAG (1): it fits

- Corpus under ~**200k tokens** (Anthropic, 2024): put it all in the prompt
- Use **prompt caching** for repeat calls
- No chunking, no index, **nothing to go stale**
- But long prompts degrade too: "fits" ≠ "works"
- Measure both options on the same questions

<!-- Retrieval is a workaround for limited context. Check whether you actually need it before building a pipeline. -->

---

## When not to use RAG (2): code

| | Index (RAG) | Agentic search |
|---|---|---|
| Speed | fast lookup | several tool calls |
| Freshness | **can go stale** | reads current files |
| Exact names | needs hybrid | `grep` is exact |
| Concept queries | good | depends on naming |

- Claude Code navigates "just in time" with `glob`/`grep`
- Hybrids still fit large, slow-changing text (legal, finance)

<!-- See topic 21, Harness over model. Reports that coding tools dropped vector indexes are partly second-hand; the trade-off table is the durable part. -->

---

## What lasts vs. what will date

**Lasts**
- Retrieval quality caps answer quality; measure it first
- Keyword + semantic beats either alone
- Chunks need context; indexes need refreshing
- Skip the index if it fits or an agent can navigate it

**Will date**
- Anthropic's 2024 numbers and the ~200k threshold
- Specific embedding models, dimensions, limits
- Which tools use indexes vs. agentic search

<!-- The thresholds move with context windows; the reasoning behind them doesn't. -->

---

## Try it yourself

1. Embed 10 sentences incl. two paraphrases; check similarities
2. Search an ID like `INV-48213` with embeddings vs. BM25
3. Build a 20-question retrieval eval; measure recall@5; change one thing
4. Move one key passage: start, middle, end; compare answers
5. Corpus fits in context? Compare full-prompt vs. RAG on 10 questions

<!-- Exercise 3 is the most valuable habit: every RAG change should move a retrieval number. -->

---
layout: end
---

# Key takeaways

- **RAG = search + generation**: debug and measure retrieval first
- **Hybrid search and chunk context** fix the most common misses
- **Skip RAG** when the corpus fits or an agent can explore it directly

anthropic.com/engineering/contextual-retrieval\
hamel.dev/blog/posts/evals-faq/how-should-i-approach-evaluating-my-rag-system.html\
arxiv.org/abs/2005.11401
