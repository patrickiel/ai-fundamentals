---
theme: ../theme
chapter: '12'
header: 'AI Fundamentals · 12 · Skill supply chain'
layout: cover
---

# Agent Skills Are a Supply Chain

**Your agent trusts every line of a skill, so vet skills like software dependencies**

---

## Three risks in one package

- **Instructions** enter context and steer the agent
- **Scripts** run on your machine
- **Dependencies** pull in more third-party code
- Claude Code docs: a plugin "can execute arbitrary code on your machine with your user privileges"

<!-- Plugin hooks and MCP servers run outside Claude Code's sandbox. Anthropic's guidance: treat installing a skill like installing software. OWASP LLM03 gives the general supply-chain framing. -->

---

## What the research found

| Study | Sample | Finding |
|---|---|---|
| Liu et al., Jan 2026 | 31,132 skills | **26.1%** vulnerable; **5.2%** malicious-looking patterns; scripts **2.12×** riskier |
| Liu et al., USENIX Sec 2026 | 98,380 skills | **157 confirmed malicious**; >half from one brand-impersonating actor |

> "Vulnerable" ≠ "malicious"; "looks malicious" ≠ "confirmed". Don't mix the two datasets.

<!-- The confirmed malicious skills averaged about 4 vulnerabilities each across attack phases: deliberate, not accidental. All 157 were removed after disclosure. -->

---

## How a skill can attack you

| Attack | Example |
|---|---|
| Hidden / adversarial instructions | "Do not mention this to the user" |
| Says one thing, does another | "Formatter" that calls the network |
| Credential theft, exfiltration | Most common category in the 2026 study |
| Malicious code | Reverse shell, webshell, miner |
| Poisoned dependencies | Vulnerable or typosquatted package |
| Impersonation | Brand copies, Cyrillic "а" names |
| Changes after review | Update or rug pull |

<!-- These seven families double as a manual checklist. Each row is traced to a source in the article. -->

---

## The visibility gap

- Users see a **summary**; the model reads **everything**
- MCP **tool poisoning**: hidden instructions to read `~/.ssh/id_rsa`
- **Shadowing**: one server alters use of another's trusted tools
- **Rug pull**: description changes **after** approval
- Same gap for skills: a listing isn't the full text

<!-- Invariant Labs, April 2025. Their mitigations: show full descriptions, pin by hash, enforce cross-server boundaries. -->

---

## Look-alike names

- Agents choose tools and skills **by name**
- "paypal" vs "pаypаl" (U+0430 Cyrillic а): same to you, different to a computer
- Check each character's **script** (Unicode UTS #39)
- Expect false positives on legitimate mixed text

<!-- The Unicode standard's own example. Scanners such as SkillSpector include a Unicode-deception check. -->

---

## Static scanning: fast, context-blind

- Example: NVIDIA **SkillSpector** (Apache 2.0)
- Regex, code analysis (`exec`/`eval`), **taint tracking**, **YARA**, Unicode checks, **OSV.dev**
- Never executes the skill; reports **file + line**; 0–100 risk score
- Limits: false positives, misses claim/behaviour mismatch, can't read images or compiled code

<!-- As of 2026-10 the README claims 71 patterns in 17 categories. Executable scripts multiply the score by 1.3. -->

---

## Semantic (LLM) review

- A model asks: **does the code match the claims?**
- But it **reads the attack text too**
- SkillSpector: LLM can raise confidence, **never lower**; deterministic findings always stay
- File contents go to your chosen provider (local CLIs like `claude_cli` supported, as of 2026-10)

> Disagreement between modes = **read the flagged lines yourself**

<!-- A static-only low score is not a clean full scan. Claude Enterprise skill scanning complements but doesn't replace manual review. -->

---

## A vetting workflow

```
source → read all → scan (static + LLM) → sandbox run
   → decide → pin version → re-review every update
```

- Official marketplace names only from `github.com/anthropics/`
- A marketplace name says **who publishes**, not what a plugin does
- Plugins: read `hooks/hooks.json`, `.mcp.json`, `bin/`

<!-- Public registries list skills from many owners. Being listed is not being vetted. -->

---

## Fix or discard?

- Careless permission: fixing may be fine
- **Deliberate** malice (exfiltration, reverse shell, look-alike names): **discard**
- Confirmed malicious skills averaged **~4 vulnerabilities**: patching one leaves the rest
- Authors shouldn't review their own skills

<!-- Separation of duties comes from Anthropic's Skills for enterprise guidance. -->

---

## Pin and limit

- Pin versions; verify **checksums**; signed commits
- "Treat every update as a new deployment"
- Third-party marketplaces: auto-update **off** by default (as of 2026-10)
- Scoped, short-lived credentials; sandboxes
- Hooks and MCP servers run **outside** the sandbox

<!-- The damage a skill can do equals the access your agent already has. -->

---

## What lasts vs. what will date

| Lasts | Will date |
|---|---|
| Skills/plugins/MCP = supply-chain deps | 26.1% / 5.2% / 2.12× / 157 figures |
| Users see summaries; models read all | SkillSpector counts, bands, providers |
| Check names at character level | Marketplace names, auto-update defaults |
| Static + semantic; models never erase findings | Enterprise scanning availability |
| Pin what you reviewed; limit reach | |

<!-- If the tool changes, keep the habits. -->

---

## Try it yourself

1. Read one installed skill against the **seven attack families**
2. Print every **non-ASCII** character with its Unicode name and script
3. Build a harmless "formatter" with a network call; scan static vs. LLM
4. Inventory what your agent can reach; decide what skills **must not** touch
5. Write a five-line pre-install checklist your agent follows

<!-- Exercise 4 matters most: limiting reach protects you from the skills your scanner misses. -->

---
layout: end
---

# Key takeaways

- A skill is **third-party code and instructions your agent obeys**
- Combine **static** and **semantic** checks; read disagreements yourself
- **Pin** what you reviewed and **limit** what any skill can reach

arxiv.org/abs/2601.10338\
platform.claude.com/docs/en/agents-and-tools/agent-skills/enterprise\
github.com/NVIDIA/SkillSpector
