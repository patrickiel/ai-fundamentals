# Data Privacy and Token Cost: What You Send, Where It Goes, and What It Costs

> **Core idea in one sentence:** Every prompt is data you hand to someone else and tokens you pay for, so send only what the task needs, know the terms it's processed under, and measure what each task costs.

## TL;DR

- What happens to your data depends on **which product and which terms** you're under (consumer vs. commercial, API vs. app, the provider vs. a cloud reseller). Read your provider's current data-usage and retention pages. Don't rely on memory or blog posts.
- The questions to ask are always the same: **Is it used for training? How long is it kept? Can I get zero retention? Which features fall outside that? Where is it processed?**
- **Secrets and personal data** shouldn't reach the model unless the task needs them. Agents read files on their own, so scope what they can reach. Their transcripts are also data, stored locally and sometimes remotely.
- **Tokens** are the unit of both context and cost. Output tokens cost several times more than input tokens, but in **agent loops, input volume usually dominates**, because the growing context is re-sent every turn.
- The main cost levers are **prompt caching, batching, model routing, and shorter contexts**. Measure **cost per completed task**, not cost per call.

## Part 1: Data privacy

### What happens to data you send

Your prompt, attached files and the model's response go to the provider's servers. Four policies decide what happens next, and they can differ between products from the same company.

**1. Training use.** As of 2026-10, Anthropic lets consumer users (Free, Pro, Max) choose whether their data trains future models. Under commercial terms (Team, Enterprise, API) it doesn't train on prompts or code unless the customer opts in [2]. OpenAI says API data hasn't been used for training since 1 March 2023 unless you opt in [3]. Consumer and business terms often differ, so check the setting on the account you actually use.

**2. Retention.** "Not used for training" doesn't mean "not stored". As of 2026-10, Anthropic documents 30-day retention for commercial Claude Code use. Consumer data is kept 5 years if training is allowed and 30 days if not [2]. OpenAI keeps API abuse-monitoring logs for up to 30 days [3]. Exceptions exist: Anthropic may keep flagged content for up to 2 years even under zero retention, and data may be kept where the law requires [1].

**3. Zero data retention (ZDR).** Both providers offer arrangements where prompts and responses aren't stored after the response is returned. They require approval [1][3]. The key detail is **feature scope**. Stateful features (batch jobs, uploaded files, persistent containers, stored conversations) fall outside ZDR at both providers. Using them "is a choice to step outside your ZDR arrangement for that specific data" [1][3]. Third-party tools you connect, such as MCP servers, follow *their own* terms [1].

**4. Feedback and telemetry.** A thumbs-down or bug report can send a full transcript under different rules. In Claude Code, `/feedback` transcripts are kept for 5 years. Usage metrics exclude code and prompts and can be turned off [2].

### Where data is processed

- **The processor depends on how you access the model.** Through a cloud platform (e.g. Amazon Bedrock or Google Cloud), that cloud provider is the data processor, under its own terms [1].
- **Data residency is usually opt-in and priced.** As of 2026-10, OpenAI lets you pin storage (and in some regions processing) per project, with a 10% uplift on newer models. Account and billing metadata aren't covered [3]. Anthropic's US-only inference costs 1.1× on Claude 4.6 and later models [6].

### Secrets and PII hygiene

OWASP lists **sensitive information disclosure** (personal data, credentials, health, financial and confidential business data) as a top LLM risk. It warns that system-prompt instructions not to reveal data "may not always" hold [4]. Habits:

- **Never paste keys, passwords or tokens.** Once sent, they sit in someone's logs for the retention period. If one slips, rotate it.
- **Redact before sending.** Replace names, emails and account numbers with placeholders (`<CUSTOMER_1>`) when the task doesn't need real values. OWASP recommends pattern-based redaction or tokenisation [4].
- **Scope what agents can read.** A coding agent will read `.env` if nothing stops it. Keep credential files out of reach with deny rules or a sandbox, and give agents scoped, short-lived tokens. This is least privilege applied to agents [4]. (Topic 13 explains why secrets plus a manipulated agent is the worst case.)
- **Remember local transcripts.** As of 2026-10, Claude Code keeps session transcripts on disk in plain text for 30 days by default [2]. Anything the agent read is in them.
- **Use the right account.** Work data belongs under commercial terms, not a personal consumer account.

### Compliance basics: data minimisation

GDPR-style law (here the UK GDPR, as explained by the ICO) requires personal data to be "adequate, relevant and limited to what is necessary" for the purpose [5]. For AI use, the ICO's guidance means [5]:

- Identify the **smallest amount of personal data** the task needs, and send no more.
- Don't include data "just in case it's useful".
- Send data only about the people the task concerns.
- Review periodically and delete what's no longer needed. That includes stored files, vector stores, and batch results on the provider's side.

Data minimisation also saves money: data you don't send can't leak, and you don't pay for its tokens.

## Part 2: Token cost

### What a token is

A **token** is a chunk of text, often a word fragment. Context limits, rate limits and bills are all counted in tokens. Counts **depend on the tokenizer**: Anthropic's newer models produce roughly 30% more tokens for the same text than earlier ones [6], so the same prompt can cost more at the same per-token price. Use the provider's token counter rather than rules of thumb [6].

### Input vs. output pricing

Output tokens cost more than input. As of 2026-10, every model in Anthropic's table charges **5× more for output than input**. For example, Claude Sonnet 5.5 is $2 per million input tokens and $10 per million output [6]. Check your provider's current page.

Tool use adds hidden input: tool definitions, a tool-use system prompt, and every tool call and result are billed as input [6]. Every connected tool costs tokens on every request.

### Why agents and long contexts multiply cost

Model calls are stateless: each turn re-sends the **whole conversation so far** (general background). An agent loop does this dozens of times, and each tool result makes the next request longer. Anthropic reported that in its research system, agents used about **4× the tokens of a chat interaction** and multi-agent systems about **15×**. It concluded that multi-agent designs only pay off when "the value of the task is high enough" [8].

Illustrative arithmetic (not a benchmark): an agent runs 20 turns, and its context grows evenly from 10k to 100k tokens. Total input sent is about 20 × 55k = **1.1M tokens**. If each turn writes 1k output tokens, total output is only 20k. Even at a 5× output price, **input dominates**. This is why context discipline (topics 03–05) is also cost discipline.

### Lever 1: Prompt caching

Most of each agent turn repeats the previous one. **Prompt caching** lets the provider reuse the processed prefix instead of re-computing it. With Anthropic, as of 2026-10 [7]:

- The cache matches an **exact prefix** in the order tools → system → messages. Put stable content first and changing content last.
- Cache reads cost **0.1×** the base input price (0.05× on some newer models). Writing to the cache costs **1.25×** for a 5-minute lifetime or **2×** for 1 hour.
- The 5-minute lifetime refreshes each time the cache is used. Long pauses let it expire.
- Changing tool definitions, the system prompt, or some settings **invalidates** the cache from that point on.
- You can check whether caching worked in the response's usage fields (`cache_creation_input_tokens`, `cache_read_input_tokens`). If both are zero, nothing was cached.

Applied to the example above, most of the 1.1M input tokens become cache reads at a fraction of the price. The exact saving depends on how often the prefix changes.

### Lever 2: Batching

If you don't need the answer right away (evaluations, bulk classification, overnight document processing), **batch APIs** process requests asynchronously. As of 2026-10, Anthropic's batch API charges **50% less** on input and output, and the discount stacks with caching [6]. The trade-offs are latency and privacy: batch jobs are stored (29 days at Anthropic) and aren't ZDR-eligible [1].

### Lever 3: Model routing

Not every step needs the largest model. Anthropic's own pricing guidance suggests small models for simple tasks, mid-size for most production work, and the largest for complex reasoning [6]. As of 2026-10, the input price gap between the cheapest and most expensive current models in that table is about 100× [6]. Routing easy steps (classification, extraction, simple edits) to a cheaper model and escalating hard ones can therefore save a lot, provided quality holds (measure it). Topic 24 (advisor strategy) covers one version: a cheap executor that consults a stronger model only when needed.

### Lever 4: Send less

Every cost lever above works better when there's less to send: shorter system prompts, fewer connected tools, sub-agents that return summaries instead of raw logs, and clearing context between unrelated tasks (topic 04).

### Measuring cost per task

Per-token prices don't tell you what a feature costs. Measure:

1. **Log usage per request.** Record input, output, cache-write and cache-read tokens from each API response.
2. **Group by task.** Sum every call (including retries and sub-agents) that went into one user-visible outcome.
3. **Divide by successes.** A cheap model that fails half the time and needs a retry or human fix may cost more per *completed* task than an expensive one that succeeds. Pair cost with an eval (topic 18).
4. **Watch the trend.** A prompt edit that breaks the cache prefix can quietly multiply costs.

## The latency / cost / quality triangle

You can usually improve two of the three at the expense of the third:

| Choice | Quality | Cost | Latency |
|---|---|---|---|
| Bigger model | ↑ | ↑ | ↑ (slower) |
| Smaller model / routing | ↓ (on hard steps) | ↓ | ↓ |
| More agent turns, multi-agent | ↑ on complex tasks | ↑↑ | ↑ |
| Prompt caching | = | ↓ | ↓ |
| Batch API | = | ↓ | ↑↑ (async) |
| Data residency | = | ↑ (uplift) | = |

Caching is the rare near-free improvement. Everything else is a trade-off you should make on purpose, per task, based on measurements.

## What lasts vs. what will date

- **Durable principles:**
  - Data you send is governed by the terms of the specific product and account. Ask: training, retention, zero-retention scope, processing location.
  - Stateful features (files, batches, stored conversations) and third-party tools sit outside zero-retention promises.
  - Minimise: send only what the task needs. This helps privacy and cost at the same time.
  - Keep secrets out of prompts and out of agents' reach. Transcripts are sensitive data too.
  - Agent cost is driven by re-sent context. Caching, routing, batching and shorter contexts are the levers.
  - Measure cost per successful task, not per token.
- **Time-sensitive details:**
  - All prices, ratios and multipliers (as of 2026-10): 5× output/input, 0.1×/0.05× cache reads, 1.25×/2× cache writes, 50% batch discount, 1.1× US-only inference, 10% residency uplift.
  - Retention periods (30 days, 5 years, 29 days, 2 years) and which features are ZDR-eligible.
  - The ~30% tokenizer difference and the 4×/15× agent multipliers (June 2025).
  - Product names, plans and settings.

## Try it yourself

1. **Audit your accounts.** For each AI product you use, write down: consumer or commercial terms? Training on or off? Retention period? Find each answer on the provider's own page and note the date.
2. **Redact a real prompt.** Take a prompt you sent recently that contained personal or company data. Rewrite it with placeholders and check whether the answer is still useful.
3. **Lock down secrets.** Add a deny rule or sandbox setting so your coding agent can't read `.env` or credential files, then ask it to read one and confirm it's blocked.
4. **Measure one agent task.** Run a typical task, record input, output and cache tokens per call (or from your tool's usage display), and compute total cost. Repeat with a smaller model and compare cost *and* success.
5. **Test caching.** Send the same long system prompt twice within five minutes and check the cache fields in the usage data. Then change one word at the start and see the cache miss.

## Key terms

- **Token**: the unit of text a model reads and writes. Counts depend on the tokenizer.
- **Retention**: how long a provider stores your prompts and outputs, separate from whether it trains on them.
- **Zero data retention (ZDR)**: an arrangement where prompts and responses aren't stored after the response is returned. It usually excludes stateful features.
- **Data residency**: keeping data storage or processing within a chosen region.
- **Data minimisation**: sending or holding only the personal data that's necessary for the purpose.
- **Prompt caching**: reusing a previously processed, identical prompt prefix at a reduced price and latency.
- **Batch API**: asynchronous bulk processing at a discount, with delayed results.
- **Model routing**: sending each step to the cheapest model that can handle it reliably.

## Sources

1. [API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention) (Anthropic docs, accessed 2026-10-14). The ZDR definition and scope, stateful features outside ZDR, batch 29-day retention, flagged-content retention, third-party integrations, and cloud platforms as data processors.
2. [Data usage (Claude Code)](https://code.claude.com/docs/en/data-usage) (Anthropic docs, accessed 2026-10-14). Consumer vs. commercial training and retention, feedback transcript retention, telemetry, and plain-text local transcripts.
3. [Data controls in the OpenAI platform](https://developers.openai.com/api/docs/guides/your-data) (OpenAI docs, accessed 2026-10-14). The no-training default since 2023-03-01, 30-day abuse-monitoring logs, ZDR/MAM eligibility, per-endpoint storage, and data residency with uplift.
4. [LLM02:2025 Sensitive Information Disclosure](https://genai.owasp.org/llmrisk/llm022025-sensitive-information-disclosure/) (OWASP Gen AI Security Project, 2025). Categories of sensitive data, redaction/tokenisation, least privilege, user education, and the limits of system-prompt restrictions.
5. [Principle (c): Data minimisation](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/data-minimisation/) (UK Information Commissioner's Office, accessed 2026-10-14). The Article 5(1)(c) wording and practical minimisation and review steps.
6. [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) (Anthropic docs, accessed 2026-10-14). Input/output prices, the tokenizer note, tool-use token overhead, batch discount, US-only inference multiplier, and model-choice guidance.
7. [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) (Anthropic docs, accessed 2026-10-14). Prefix order, cache lifetimes, read/write multipliers, invalidation rules, and usage fields for verification.
8. [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (Anthropic Engineering, 2025-06-13). The ~4× (agent) and ~15× (multi-agent) token multipliers vs. chat, and the economic-viability point.
