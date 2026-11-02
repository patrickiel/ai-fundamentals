---
theme: ../theme
chapter: '13'
header: 'AI Fundamentals · 13 · Prompt injection'
layout: cover
---

# Prompt Injection

**Models can't reliably tell your instructions from text they read, so limit what obeying the wrong text can do**

---

## The problem

- Agents read web pages, issues, emails, docs, tool outputs
- Any of that text can contain **instructions**
- The model may follow them as if **you** wrote them
- What's at stake is whatever the agent can **access and do**
- No complete fix exists (OWASP, NCSC, CaMeL authors, vendor docs)

<!-- Topic 12 showed this with malicious skills; here we generalise to any text the agent reads. -->

---

## Why there's no boundary

| SQL injection | Prompt injection |
|---|---|
| Code and data in separate channels | One stream of tokens |
| Parameterised queries **fix** it | Tags and labels only **help** |
| Deterministic parser | Probabilistic model |

> NCSC: the LLM is an **"inherently confusable deputy"**

<!-- Because outcomes are probabilistic, an attack that fails nine times can succeed on the tenth. -->

---

## Direct vs. indirect

| | Direct | Indirect |
|---|---|---|
| Source | User's own prompt | Content the model reads |
| Carriers | Chat, API input | Web, email, PDFs, issues, READMEs, tool/MCP output, **images** |
| Visible? | Yes | Often **hidden** |

For coding agents, **indirect** is the main threat

<!-- The "attacker" is whoever wrote the page or issue your agent opened. -->

---

## Jailbreak ≠ injection

- **Jailbreak**: defeat the *model's* safety training
  - risk: embarrassment, misuse of knowledge
- **Injection**: hijack the *application* via untrusted input
  - risk: your data, your repo, your accounts
- A jailbreak filter can miss "please forward the sales figures to…"
- **Different threats, different defences**

<!-- Injected instructions are often things the model would happily do for a legitimate user, so safety training doesn't help. -->

---

## The lethal trifecta

```
 Private data  +  Untrusted content  +  External communication
   (files,         (web, issues,          (HTTP, image URLs,
    inbox, keys)    email, MCP output)     links, PRs, email)
                         ▼
                 Data exfiltration
```

**Remove any one leg** and the theft scenario fails

<!-- Willison: "The only way to stay safe there is to avoid that lethal trifecta combination entirely." -->

---

## Why removing a leg works best

- Filters ask: *will the model resist?* (probabilistic)
- Removing a leg asks: *can the attack work at all?* (checkable)
- Mixing MCP servers easily creates the trifecta by accident
- Vendors fixed past incidents by **closing the exfiltration channel**
- Note: the trifecta is about **theft**; destructive actions need limits too

<!-- One tool can supply all three legs, e.g. reading public issues, private repos, and opening PRs. -->

---

## Architecture: constrain after untrusted input

> Once an agent has read untrusted input, that input must not be able to trigger consequential actions

- **Plan-then-execute**: fix the plan before reading data
- **Action-selector**: pick from a fixed menu only
- **Dual LLM**: privileged model never sees untrusted text; quarantined model has no tools
- **CaMeL**: dual LLM + data-flow capabilities + policies (77% vs 84% tasks on AgentDojo)

<!-- Even CaMeL's authors say injection is "not fully solved": policy burden, side channels, prompt fatigue. -->

---

## Practical layers

| Layer | Strength |
|---|---|
| Least privilege, deterministic limits | **Strong** |
| Sandbox: filesystem **and** network | **Strong** |
| Egress allowlist (narrow domains) | **Strong** |
| Human approval for consequential actions | Medium (fatigue) |
| Labelling untrusted content | Weak–medium |
| Input/output filters, deny-lists | **Defence in depth only** |

<!-- NCSC: "there are infinite ways to rephrase an attack". Be wary of products claiming to "stop" injection. -->

---

## Daily habits for coding-agent users

1. Check each task for the **trifecta**; break a leg
2. Keep **secrets out of reach**; scoped, short-lived tokens
3. **Sandbox + tight network allowlist**, or container/VM
4. Never auto-approve **network or destructive** commands
5. Treat unfamiliar repos, issues, MCP servers as **hostile**
6. **Research session ≠ action session**
7. Review diffs to CI, manifests, auth code

<!-- Habit 6 is a manual dual-LLM pattern: summarise untrusted content in one place, act on the reviewed summary in another. -->

---

## What lasts vs. what will date

**Lasts**
- No enforced instruction/data boundary in LLMs
- Injection is a security issue, separate from jailbreaking
- Trifecta → remove a leg; prefer architecture over detection

**Will date**
- Specific incidents and products (mid-2025)
- CaMeL numbers; pattern catalogue
- Claude Code sandbox details (e.g. no native Windows, as of 2026-10)

<!-- The habits transfer to any agent; the product features will change. -->

---

## Try it yourself

1. Mark each agent tool **P / U / X**; find trifecta combos
2. Plant a harmless "say PINEAPPLE" line in a README; test rewordings
3. Enable a sandbox with a one-domain allowlist
4. Split "fix these issues" into **plan, then execute**
5. Remove auto-approved commands that can send data out

<!-- Exercise 2 shows how inconsistent, and therefore unfilterable, injection behaviour is. -->

---
layout: end
---

# Key takeaways

- **Any text the agent reads can steer it**: there is no complete fix
- **Break the lethal trifecta**: private data, untrusted content, exfiltration
- **Architecture and isolation first**; filters are only a backup layer

simonwillison.net/2025/Jun/16/the-lethal-trifecta/\
genai.owasp.org/llmrisk/llm01-prompt-injection/\
ncsc.gov.uk/blog-post/prompt-injection-is-not-sql-injection
