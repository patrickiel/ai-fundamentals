# Agent Skills Are a Supply Chain: How They Attack You and How to Vet Them

> **Core idea in one sentence:** An installable skill or plugin is third-party code plus instructions that your agent trusts by default, so vet it like a software dependency: read it, scan it with both pattern-based and semantic checks, pin what you reviewed, and limit what any skill can reach.

## TL;DR

- A skill bundles **instructions that enter the agent's context** and often **code that runs with your permissions**. Anthropic's guidance is to treat installing one like installing software [4][5].
- The problem is measurable. A 2026 study of 31,132 public skills found **26.1%** contained at least one vulnerability, **5.2%** showed high-severity patterns suggesting malicious intent, and skills bundling scripts were **2.12×** more likely to be vulnerable [1]. A follow-up confirmed **157** malicious skills among 98,380 [2].
- The main attack families: **hidden or adversarial instructions, behaviour that doesn't match the description, credential theft and exfiltration, malicious code, poisoned dependencies, impersonation, and changes after review** [1][2][4][6].
- **Static scanning** (patterns, code analysis, malware signatures, vulnerability databases) is fast and points to file and line, but lacks context. **Semantic (LLM) review** can judge whether code matches its claims, but it reads attacker-controlled text. Use both and investigate disagreements [3].
- Scanners are **defence in depth, not a sandbox** [3]. Pin reviewed versions, review updates, and limit what your agent can reach [4][5].

## Why skills are a supply-chain problem

Topic 11 described skills as folders of instructions, scripts and resources. From a security view, each part is a different risk:

- **Instructions** enter the model's context and steer what it does with the tools it already has.
- **Scripts** run on your machine.
- **Dependencies** that a skill installs or fetches bring in further third-party code.

Claude Code's plugin documentation states the stakes directly: "A Claude Code plugin you install can execute arbitrary code on your machine with your user privileges." Plugin hooks and MCP servers run as processes outside Claude Code's sandbox, while skills, commands and agents "enter Claude's context as instructions" [5]. Anthropic's skills documentation warns that, depending on what access Claude has, a malicious skill "could lead to data exfiltration, unauthorized system access, or other security risks" [4].

This is the same problem as installing an npm or PyPI package from an unknown author. OWASP's Top 10 for LLM Applications lists supply chain as LLM03 and recommends vetting suppliers, keeping a software bill of materials, and verifying integrity with signatures and hashes [7]. OWASP's entry focuses on models, data and packages, but the logic carries over. Skills add one twist: plain-language text is itself an attack surface, because the agent acts on it (topic 13).

## What the research found

**"Agent Skills in the Wild"** (Liu et al., January 2026) is the first large-scale security study of skills. The authors collected 42,447 skills from two marketplaces and analysed 31,132 with a detector combining static analysis and LLM-based classification. They describe skills as executing "with implicit trust and minimal vetting" [1]. Findings:

- **26.1%** had at least one vulnerability, across 14 patterns in four categories: prompt injection, data exfiltration, privilege escalation and supply-chain risk.
- **Data exfiltration** and **privilege escalation** were the most common.
- **5.2%** showed high-severity patterns suggesting malicious intent.
- Skills that bundle **executable scripts** were **2.12×** more likely to be vulnerable than instruction-only skills.

The authors call for capability-based permissions and mandatory security vetting [1].

**"Do Not Mention This to the User"** (Liu et al., accepted to USENIX Security 2026) went further and confirmed malice by analysis [2]. Across 98,380 skills from two community registries:

- **157** skills were confirmed malicious, with 632 vulnerabilities across 13 attack techniques, averaging 4.03 per skill across several attack phases. The authors read this as evidence the threats are "deliberate rather than accidental".
- Two strategies dominated: **data thieves** (credential theft through code execution) and **agent hijackers** (adversarial instructions embedded in documentation).
- **Over half** of confirmed cases came from one actor using templated **brand impersonation**.
- After disclosure, registry maintainers removed all 157.

**Reading the numbers carefully.** "Vulnerable" is not "malicious": many flagged skills are careless rather than hostile. And "patterns suggesting malice" (5.2% in one dataset) is a much broader net than "confirmed malicious" (157 of 98,380 in another). The two studies used different data and methods, so don't combine their figures. The practical point holds either way: a meaningful share of public skills is unsafe to install unreviewed, and a small number are built to attack you.

## How a skill can attack you

| Attack | How it works | What to check |
|---|---|---|
| **Hidden or adversarial instructions** | Directives to ignore safety rules, act conditionally, or hide actions from the user, sometimes in comments, invisible characters or encoded text [2][4] | Read every text file, including comments. Reveal non-ASCII and zero-width characters |
| **Says one thing, does another** | A skill invokes tools or runs code "in ways that don't match the Skill's stated purpose" [4] | Compare scripts against the description. Run them in a sandbox |
| **Credential theft and exfiltration** | Reads keys, tokens or files and sends them out, the most common category in [1] | Secret reads combined with network calls. Anthropic flags the "combined risk" of file-read plus network tools [4] |
| **Malicious code** | Remote code execution, reverse shells, webshells, cryptominers [2][3] | Bundled scripts, download-and-run commands, `exec`/`eval`/`subprocess` |
| **Poisoned dependencies** | The skill installs a vulnerable or malicious package, or fetches content that later changes [4] | Every package and URL, checked against vulnerability databases. Look-alike (typosquatted) names |
| **Impersonation** | Brand impersonation [2], or look-alike names using characters from another script [8] | Who actually publishes it. Mixed-script names |
| **Changes after review** | An update or remote fetch replaces what you approved [4][6] | Version pinning, checksums, update settings |

Two of these need more explanation.

**Look-alike names.** Unicode's security standard gives the example of "paypal" spelled with a Cyrillic "а" (U+0430): it looks the same to a person but is a different string to a computer [8]. Agents select tools and skills by name, so a look-alike can stand in for something you trust. Checking the script of each character catches it. The standard also warns that legitimate Latin and Cyrillic text often trips such checks, so expect false positives [8].

**The visibility gap.** Invariant Labs' 2025 disclosure of **tool poisoning** in MCP showed the core weakness. Instructions hidden in a tool description are "invisible to users but visible to AI models". A poisoned `add` tool told the model to read `~/.cursor/mcp.json` and an SSH key and pass them as a hidden argument. They also showed **shadowing**, where one server's description changes how the agent uses another, trusted server's tools, and the **rug pull**, where "a malicious server can change the tool description after the client has already approved it" [6]. Skills share the gap: what you skim in a listing is not everything the model reads.

## Two kinds of scanning

NVIDIA's open-source **SkillSpector** (Apache 2.0) is a concrete example of a skill scanner. Its README describes the following (as of 2026-10) [3].

### Static analysis: fast, specific, context-blind

SkillSpector's first stage claims 71 patterns across 17 categories, from prompt injection and data exfiltration to supply chain and MCP tool poisoning. Its methods include:

- regex analyzers for suspicious text
- code analysis that flags calls such as `exec`, `eval` and `subprocess`
- **taint tracking** from sources to sinks, for example credentials flowing to network output
- **YARA** signatures for malware, webshells and cryptominers
- **Unicode deception** checks for homoglyphs, right-to-left overrides and mixed-script identifiers
- dependency lookups against **OSV.dev** for known vulnerabilities

It "never executes the scanned skill". Each finding gives a file and line. Severities add up to a 0–100 risk score, with a 1.3× multiplier when the skill contains executable scripts. Bands run from SAFE (0–20) to DO NOT INSTALL (above 50).

Static analysis has known limits. Patterns lack context, so a legitimate skill that downloads a tool or reads an environment variable can look like a malicious one. And a skill whose code simply contradicts its description may contain no "bad pattern" at all. The README also lists blind spots: non-English content, text in images, and compiled or encrypted code [3].

### Semantic (LLM) review: reads for meaning, but reads the attack too

The second stage sends skill content to a language model to judge intent and consistency. The study in [1] combined the same two approaches. SkillSpector makes a specific design choice: the model can add explanations and raise confidence in findings, "never lower", and "every deterministic finding stays in the report whether the model confirms it, disputes it, or does not address it" [3].

This choice matters because the LLM reviewer reads the very text that may contain injected instructions, such as a skill that tells its reader to report it as safe. The README says its prompt includes anti-jailbreak protections, but no prompt is a guarantee (topic 13). Two practical notes:

- With LLM analysis on, file contents go to whichever provider you configure. As of 2026-10, the options include hosted APIs and local agent CLIs such as `claude_cli`, which uses your existing login instead of an API key [3].
- A low score from a static-only run is not a clean full scan. The README flags this explicitly for its MCP mode [3].

**When the two stages disagree, read the flagged lines yourself.** Neither score is a verdict.

Platform scanning is starting to appear as well. As of 2026-10, Claude Enterprise organizations can turn on scanning for skills uploaded in claude.ai and Cowork. Anthropic says it "complements, but doesn't replace" manual review, and it doesn't cover skills uploaded through the API [4].

## A vetting workflow

A scanner you have to remember to run will often be skipped. Build vetting into the only path by which skills reach your agent:

1. **Check the source.** Who publishes it? In Claude Code, official marketplace names are accepted only from `github.com/anthropics/` repositories, so a third party can't pose as Anthropic's marketplace. But "a marketplace's name tells you who publishes the catalog, not what each plugin in it does" [5]. Public registries such as Vercel's skills.sh list skills from many owners. Being listed is not the same as being vetted.
2. **Read everything.** `SKILL.md`, every referenced file and every script. For plugins, also read `hooks/hooks.json`, `.mcp.json` and the `bin/` directory, because the install screen shows that a hook exists but not what it runs [4][5].
3. **Scan with both modes** and keep the report.
4. **Run scripts in a sandbox** and confirm their behaviour matches the description [4].
5. **Decide.** Fixing a careless permission may be fine. For signs of **deliberate** malice (exfiltration, reverse shells, look-alike names), discard the skill. The confirmed malicious skills in [2] averaged four vulnerabilities each across attack phases, so patching one finding leaves an attacker's other work in place.
6. **Pin what you reviewed.** Anthropic's enterprise guidance: pin versions in production, verify checksums of reviewed skills, use signed commits, and "treat every update as a new deployment requiring full security review" [4]. Claude Code auto-updates plugins from official marketplaces by default, but not from third-party ones. Updates can change files you already reviewed [5].
7. **Limit the blast radius.** A skill can do whatever your agent can do. Use scoped, short-lived credentials and sandboxes, and keep secrets out of reach. Remember that plugin hooks and MCP servers run outside the sandbox [5]. In teams, skill authors should not review their own skills [4].

You can package steps 1–5 as a skill or script of your own, so the agent follows them every time it looks for something new.

## What lasts vs. what will date

- **Durable principles:**
  - Skills, plugins and MCP servers are supply-chain dependencies, and their instructions can steer your agent as well as their code.
  - Users see summaries while models read everything. Review the full text.
  - Agents select by name, so check names at the character level.
  - Static and semantic checks fail in different ways. Combine them, and never let a model's opinion erase a deterministic finding.
  - Pin what you reviewed. Updates are new code.
  - The damage a skill can do equals the access your agent already has.
- **Time-sensitive details:**
  - Study figures (26.1%, 5.2%, 2.12×, 157 of 98,380) and the registries they sampled (2026).
  - SkillSpector's pattern and category counts, scoring bands and provider list (as of 2026-10).
  - Claude Code marketplace names, auto-update defaults and Enterprise scanning availability (as of 2026-10).

## Try it yourself

1. **Read one skill like an attacker.** Pick a skill you use and check it against the seven rows of the attack table. Note any mismatch between description and code.
2. **Reveal hidden characters.** Run a short script over a skill's files that prints every character outside printable ASCII with its Unicode name and script.
3. **Test both scan modes.** Write a harmless "formatter" skill that also makes an innocent network request. Scan it static-only, then with an LLM pass, and compare.
4. **Inventory your agent's reach.** List the credentials, folders and commands your agent can access, and decide which an untrusted skill must never touch. Enforce it with a sandbox, a separate profile or scoped tokens.
5. **Write your install checklist.** Turn the vetting workflow into five lines your agent follows before installing any skill, plugin or MCP server.

## Key terms

- **Supply-chain attack**: compromising a system through a third-party component it trusts, such as a package, plugin or skill.
- **Prompt injection**: text an agent reads and follows as instructions, planted by someone other than the user (topic 13).
- **Exfiltration**: secretly sending data such as credentials from your machine to an attacker.
- **Homoglyph / confusable**: a character that looks like another (Cyrillic "а" vs. Latin "a"), used to make look-alike names.
- **Tool poisoning**: hiding instructions in tool metadata that the model reads but the user doesn't see.
- **Rug pull**: changing a component after it has been reviewed or approved.
- **Typosquatting**: publishing a malicious package under a name one typo away from a popular one.
- **Taint tracking**: following data from sensitive sources (secrets) to risky sinks (network calls) through code.
- **False positive / false negative**: a safe item flagged as dangerous, or a dangerous item marked safe.

## Sources

1. [Agent Skills in the Wild: An Empirical Study of Security Vulnerabilities at Scale](https://arxiv.org/abs/2601.10338) (Yi Liu et al., arXiv, 2026-01-15). Prevalence of vulnerable skills, the four-category taxonomy, the script-bundling risk factor, and the call for capability-based permissions and vetting.
2. ["Do Not Mention This to the User": Detecting and Understanding Malicious Agent Skills in the Wild](https://arxiv.org/abs/2602.06547) (Yi Liu et al., arXiv 2026-02, rev. 2026-06; USENIX Security 2026). Confirmed malicious skills, the data-thief and agent-hijacker strategies, brand impersonation at scale, and takedowns after disclosure.
3. [NVIDIA SkillSpector](https://github.com/NVIDIA/SkillSpector) (NVIDIA, GitHub, Apache 2.0, accessed 2026-10-14). A concrete scanner: static analyzers, taint tracking, YARA, Unicode deception checks, OSV.dev lookups, risk scoring, the LLM stage's "never lower" rule, data egress and stated limitations.
4. Claude Platform documentation: [Agent Skills overview, Security considerations](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#security-considerations) and [Skills for enterprise](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/enterprise) (Anthropic, accessed 2026-10-14). Risk indicators, the eight-step review checklist, version pinning and checksums, separation of duties, and Enterprise skill scanning.
5. Claude Code documentation: [Plugin security and trust](https://code.claude.com/docs/en/plugins/security) and [Security](https://code.claude.com/docs/en/security) (Anthropic, accessed 2026-10-14). What a plugin can run, which components bypass the sandbox, marketplace tiers and name protection, how to review a plugin, and auto-update behaviour.
6. [MCP Security Notification: Tool Poisoning Attacks](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) (Invariant Labs, 2025-04-01). Hidden instructions in tool descriptions, shadowing, rug pulls, and mitigations such as showing full descriptions and pinning by hash.
7. [LLM03:2025 Supply Chain](https://genai.owasp.org/llmrisk/llm032025-supply-chain/) (OWASP Gen AI Security Project, 2025). The supply-chain framing for LLM applications: supplier vetting, SBOMs, and integrity checks with signatures and hashes.
8. [Unicode Technical Standard #39: Unicode Security Mechanisms](https://www.unicode.org/reports/tr39/) (Unicode Consortium, v18.0.0, 2026-08-27). Confusable and mixed-script detection, the Cyrillic-"а" example, and the warning about false positives.
