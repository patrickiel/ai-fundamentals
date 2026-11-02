---
theme: ../theme
chapter: '30'
header: 'AI Fundamentals · 30 · Forward deployed engineer'
layout: cover
---

# The Forward Deployed Engineer

Value from AI depends on understanding the **real process**: watch the work, split steps between **software, model and person**, design for failure, test, and **measure the value**.

---

## Where the role comes from

- Palantir: **Devs** build the platform, **Deltas** (Forward Deployed Software Engineers) deploy it
- Dev: "one capability, many customers" · Delta: "one customer, **many capabilities**"
- Success = **impact on the customer's goal**
- Not consulting: a working, long-term solution, not a one-time recommendation

<!-- Deltas also feed fixes and features back into the core product, and often choose between platform work and a faster custom build. -->

---

## The role at AI companies

| Company | How they describe it |
|---|---|
| **Anthropic** | Applied AI FDEs build production apps with Claude inside customer systems; deliver MCP servers, sub-agents, skills; 25–50% travel |
| **OpenAI** | Embed with clients, learn their workflows, work alongside their staff (Colin Jarvis) |
| **Cursor** | Best fit: less mature customer + highly customisable product; ~6-week scoped engagements (Pauline Brunet) |

<!-- Cursor warns against FDE teams drifting into solution architecture, bug logging or product training. -->

---

## The deployment gap

- MIT NANDA (July 2025): **95% of organisations** in its research saw zero return; **5%** of integrated pilots extract millions
- Cause: **approach**, not model quality: brittle workflows, no contextual learning, poor fit with daily operations
- External partnerships reached deployment ~**2×** as often as internal builds (self-reported)
- Caveat: **preliminary** study; not a universal failure rate

<!-- Sample: 300+ public initiatives, interviews with 52 organisations, survey of 153 senior leaders. -->

---

## Why it has to be someone's job

- AI gets "**slapped on top of broken processes**" (Vasuman Moza, Varick)
- Documentation covers the **main path**, rarely what people do when things go wrong
- Those **undocumented exceptions** are what an agent most needs to know
- So engineers go and sit with the people doing the work

<!-- The people running a process have done it so long that their workarounds look normal to them. -->

---

## Practices that recur

- **Build on systems of record**: a client spent $5M and 5 years moving to NetSuite; don't ask them to leave it
- **Keep workflows recognisable**: turn 11 steps into 1 and "adoption rates might suffer"
- **Budget for trust**: Morgan Stanley build 6–8 weeks, then **4 months** of pilots and evals → ~98% adoption
- **Pick work that adds up**: frequent, costly tasks first

<!-- First two from Moza's talk; the Morgan Stanley timeline is Colin Jarvis as reported by Business Insider. The last is common practice. -->

---

## Step 1 · Audit the real process

- Watch the job; write **every step in the order it actually happens**
- Ask **why** each step exists; unexplained steps are often important workarounds
- Gather context: interviews, emails, spreadsheets, SOPs, approvals
- Output: an **operating map**: current/future workflow, use case, boundaries, expected value

<!-- Varick describes starting every engagement this way. -->

---

## Step 2 · Software, model or person?

| Assign to | When |
|---|---|
| **Deterministic software** | Rules and inputs are predictable |
| **Agent (model)** | Clear objective, but inputs, paths or actions vary |
| **Human in control** | Material ambiguity, accountability, irreversible consequences |

Moza's 8-step example: **4 autonomous · 3 human-in-the-loop · 1 human**

<!-- The human step stayed human because the risk was too high or automating it wouldn't produce measurable value. Compare the responsibility model in topic 28. -->

---

## Step 3 · Design for failure

- A model answers **whether or not it is sure**
- Expect: missing data, malformed output, dead APIs, timeouts, duplicates, partial completion
- Use: checkpoints, retries with backoff, **schema validation**, confidence thresholds → escalation
- High-risk actions go to a **person**
- Grant more authority **only after** the system proves reliable

<!-- From Varick's guide: start with the smallest useful action. -->

---

## Step 4 · Test on known answers

1. Collect real cases with **known correct outcomes**
2. Cover normal, edge, incomplete, ambiguous, high-risk inputs
3. Run each case **more than once**
4. **Count** correct results, then **read every miss**

> Turns "it seems to work" into a number you can track (topic 18).

<!-- Morgan Stanley's pilots included collecting evaluations with advisors. -->

---

## Step 5 · Put a number on it

Value = **more revenue**, **less cost**, or **less risk** (Brunet, Cursor)

- Customer: the agent is "costing **$2,000 per day**"
- It sent the **right person** to repair equipment
- Framed as lower dispatch costs, it could be worth it; he had "**never measured it this way**"

<!-- Record a baseline during the audit so this comparison is possible later. -->

---

## What the role demands

- **Engineering depth**: Palantir's "technical decomp", from business problem to lines of code
- Cursor hires **5+ years** of engineering plus customer-facing experience
- Anthropic asks for production LLM, agent and **eval** experience
- **Empathy and discovery**: understand users and business strategy
- **Comfort with ambiguity** and autonomy

<!-- Coding agents make first versions cheaper, which shifts scarce effort toward discovery, mapping and evaluation. Employers still expect strong engineers. -->

---

## What lasts vs. what will date

| Durable | Will date (as of 2026-10) |
|---|---|
| Documented process ≠ real process | Team sizes and hiring requirements |
| Assign steps by predictability, judgement, cost of error | Travel shares and job-post details |
| Build on existing systems; keep workflows familiar | MIT NANDA figures (preliminary, 2025) |
| Expand autonomy as reliability is proven | MCP servers, skills as delivery units |
| Value = revenue, cost or risk vs. today | |

<!-- The method is older than the AI wave; Palantir's 2019 description already contains most of it. -->

---

## Try it yourself

1. **Shadow** a repetitive task for an hour; mark steps nobody can justify
2. Label each step **software / agent / agent + check / human**
3. Build a **20–50 case** known-answer set; run twice; read every miss
4. **Price the alternative**: agent cost vs. current cost including errors
5. Write a **one-page audit**, including how you would pilot and earn trust

<!-- The audit document is also concrete evidence of the skill when applying for FDE roles. -->

---
layout: end
---

# Key takeaways

- **The real process beats the documented one**: audit before you build
- **Split the work** between software, models and people; widen autonomy with evidence
- **Adoption and value decide success**: build on existing systems, budget for trust, measure revenue, cost or risk

blog.palantir.com/dev-versus-delta-demystifying-engineering-roles-at-palantir-ad44c2a6e87\
ai.engineer/talks/forward-deployed-engineering-at-cursor-pauline-brunet\
ai.engineer/talks/l0FLhNqBOic-ai-tools-forward-deployed-engineering
