---
theme: ../theme
chapter: '20'
header: 'AI Fundamentals · 20 · Data privacy & cost'
layout: cover
---

# Data Privacy and Token Cost

**Every prompt is data you hand over and tokens you pay for: send only what the task needs, and measure it**

---

## The problem

- Prompts, files and agent transcripts leave your machine
- Terms differ by **product and account**, even at one provider
- Agents read files on their own, **secrets included**
- Agent loops re-send growing context, so **costs multiply**
- Privacy and cost share one fix: **send less**

<!-- This topic pairs privacy and cost because both are governed by what you send. -->

---

## Four questions for any provider

| Question | Why it matters |
|---|---|
| Used for **training**? | Consumer vs. commercial defaults differ |
| How long **retained**? | "No training" ≠ "not stored" |
| **Zero retention** available? Which features excluded? | Files, batches, stored chats usually excluded |
| **Where** processed? Who is the processor? | Residency, cloud resellers, compliance |

Read the provider's **current** pages and note the date

<!-- Example as of 2026-10: Anthropic commercial 30-day retention; OpenAI API not trained on by default since 2023-03-01. -->

---

## Exceptions to know

- Stateful features (batch, files, containers) sit **outside ZDR**
- Flagged content may be kept longer (e.g. up to 2 years)
- Connected tools and MCP servers follow **their own** terms
- Feedback buttons can send **full transcripts** (e.g. 5-year retention)
- Data residency: opt-in, often with a **price uplift**

<!-- Using a stateful feature is, in Anthropic's words, "a choice to step outside your ZDR arrangement for that specific data". -->

---

## Secrets and PII hygiene

- **Never paste keys**; if one slips, rotate it
- **Redact** with placeholders: `<CUSTOMER_1>`
- **Scope agent access**: deny `.env`, credentials; use a sandbox
- Give agents **scoped, short-lived tokens**
- Local transcripts are **plain text**: treat them as sensitive
- Work data under **work (commercial) terms**

<!-- OWASP warns that system-prompt instructions not to reveal data "may not always" hold. -->

---

## Data minimisation

> Personal data must be "adequate, relevant and limited to what is necessary" (UK GDPR Art. 5(1)(c))

- Smallest amount of personal data the task needs
- No "just in case" context
- Only about the people concerned
- Review and delete: stored files, vector stores, batch results

**Bonus:** data you don't send can't leak and isn't billed

<!-- Compliance at a conceptual level; real obligations depend on your jurisdiction and role. -->

---

## Tokens and pricing

- **Token** = text chunk; counts depend on the **tokenizer**
  - newer Claude tokenizer: ~30% more tokens for the same text
- **Output costs more than input**
  - as of 2026-10: 5× across Anthropic's table (e.g. $2 in / $10 out per MTok)
- Tool definitions, calls and results are **billed as input**

<!-- Use the provider's token counter, not rules of thumb; prices change. -->

---

## Why agents multiply cost

```
turn 1:  [ctx 10k] ──►
turn 2:  [ctx 14.7k ......] ──►
  ...
turn 20: [ctx 100k ..........................] ──►
total input ≈ 1.1M tokens · total output ≈ 20k
```

- Whole context **re-sent every turn**: **input dominates**
- Anthropic (2025): agents ~**4×**, multi-agent ~**15×** chat tokens

<!-- Illustrative arithmetic, not a benchmark. Context discipline (topics 03–05) is cost discipline. -->

---

## Lever: prompt caching

- Reuses an **identical prefix**: tools → system → messages
- Stable content **first**, changing content **last**
- As of 2026-10 (Anthropic): reads **0.1×** (0.05× some models), writes **1.25×** (5 min) / **2×** (1 h)
- Edits to tools/system **invalidate** the cache
- Verify via `cache_read_input_tokens`

<!-- The rare near-free win: lower cost and lower latency at the same quality. -->

---

## Levers: batching, routing, sending less

| Lever | Saves | Costs you |
|---|---|---|
| **Batch API** (−50%, 2026-10) | Money | Latency; stored 29 days, not ZDR |
| **Model routing** (topic 24) | Money, latency | Quality on hard steps |
| **Shorter context / fewer tools** | Money, quality | Effort to curate |
| **Sub-agents returning summaries** | Main-context tokens | Extra calls |

<!-- As of 2026-10, input prices span about 100× across Anthropic's current models. -->

---

## Measure cost per completed task

1. Log input / output / cache tokens **per request**
2. Group all calls (retries, sub-agents) **per task**
3. Divide by **successes**, not attempts (pair with evals, topic 18)
4. Watch trends: a broken cache prefix **quietly multiplies** cost

<!-- A cheap model that fails half the time can cost more per completed task than a strong one. -->

---

## Latency / cost / quality

| Choice | Quality | Cost | Latency |
|---|---|---|---|
| Bigger model | ↑ | ↑ | ↑ |
| Routing to smaller | ↓ hard steps | ↓ | ↓ |
| Multi-agent | ↑ complex | ↑↑ | ↑ |
| Caching | = | ↓ | ↓ |
| Batch | = | ↓ | ↑↑ |

<!-- Make each trade-off deliberately, per task, from measurements. -->

---

## What lasts vs. what will date

**Lasts**
- Ask: training, retention, ZDR scope, location
- Minimise data; keep secrets out of reach
- Re-sent context drives agent cost; cache, route, batch, trim
- Measure cost per successful task

**Will date**
- Every price, ratio, multiplier and retention period (2026-10)
- Plan names, settings, ZDR eligibility lists

<!-- Re-check the numbers on the provider's page before relying on them. -->

---

## Try it yourself

1. Audit each AI account: terms, training, retention (with dates)
2. Redact a real prompt with placeholders; is the answer still useful?
3. Block `.env` from your agent; confirm the block works
4. Cost one agent task; repeat with a smaller model; compare **success**
5. Send a long prompt twice; check cache hits; change the first word

<!-- Exercise 5 makes prefix-matching concrete: one changed token at the start breaks the cache. -->

---
layout: end
---

# Key takeaways

- **Know the terms** your data travels under; read the provider's current pages
- **Send less**: it protects privacy and cuts cost at the same time
- **Cache, route, batch, and measure cost per completed task**

platform.claude.com/docs/en/manage-claude/api-and-data-retention\
developers.openai.com/api/docs/guides/your-data\
platform.claude.com/docs/en/build-with-claude/prompt-caching
