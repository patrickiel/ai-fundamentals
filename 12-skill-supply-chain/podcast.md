<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 12: Skill supply-chain security

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter twelve: skill supply-chain security. How installed skills can attack you, and how to vet them.

MAYA: Last time, in chapter eleven, we covered Agent Skills. Folders of instructions, scripts and resources that an agent loads in stages. And near the end you said installing a skill is like installing software.

BEN: Today we take that seriously. Here's the one sentence. An installable skill or plugin is third-party code plus instructions that your agent trusts by default. So vet it like a software dependency. Read it, scan it with both pattern-based and semantic checks, pin what you reviewed, and limit what any skill can reach.

MAYA: Read, scan, pin, limit.

BEN: [laughs] That's a good short version.

MAYA: But why is a skill a supply-chain problem? It's a folder of text files, mostly.

BEN: That "mostly" is the issue. From a security point of view, each part of a skill is a different risk. Instructions enter the model's context, and steer what it does with the tools it already has. Scripts run on your machine. And dependencies that a skill installs or fetches bring in further third-party code.

MAYA: [curious] So even a skill with no code at all can be a risk?

BEN: Yes. Claude Code's plugin documentation states the stakes directly. Quote: "A Claude Code plugin you install can execute arbitrary code on your machine with your user privileges." Plugin hooks and MCP servers run as processes outside Claude Code's sandbox. Skills, commands and agents, quote, "enter Claude's context as instructions".

MAYA: And what's the worst case?

BEN: Anthropic's skills documentation warns that, depending on what access Claude has, a malicious skill "could lead to data exfiltration, unauthorized system access, or other security risks".

MAYA: This sounds like installing some random package from the internet.

BEN: It's the same problem as installing an npm or PyPI package from an unknown author. OWASP's Top 10 for LLM Applications lists supply chain as {LLM03|L L M zero three}. It recommends vetting suppliers, keeping a software bill of materials, and verifying integrity with signatures and hashes. OWASP's entry focuses on models, data and packages, but the logic carries over.

MAYA: So is anything different about skills?

BEN: One twist. Plain-language text is itself an attack surface, because the agent acts on it. That's the subject of chapter thirteen.

MAYA: How big is the problem, really? Is it theoretical?

BEN: It's been measured. The first large-scale security study of skills is called "Agent Skills in the Wild", by Liu and colleagues, from January 2026. They collected forty-two thousand, four hundred and forty-seven skills from two marketplaces, and analysed thirty-one thousand, one hundred and thirty-two of them. Their detector combined static analysis with LLM-based classification. And they describe skills as executing, quote, "with implicit trust and minimal vetting".

MAYA: What did they find?

BEN: Twenty-six point one percent had at least one vulnerability, across fourteen patterns in four categories: prompt injection, data exfiltration, privilege escalation, and supply-chain risk. Data exfiltration and privilege escalation were the most common.

MAYA: [sighs] More than a quarter.

BEN: And five point two percent showed high-severity patterns suggesting malicious intent. One more finding stands out. Skills that bundle executable scripts were two point one two times more likely to be vulnerable than instruction-only skills.

MAYA: So in that study, bundling scripts roughly doubled the chance of a vulnerability.

BEN: Right. The authors call for capability-based permissions and mandatory security vetting.

MAYA: Was there a follow-up?

BEN: There was. A second paper by Liu and colleagues, titled "Do Not Mention This to the User", accepted to USENIX Security 2026. It went further and confirmed malice by analysis. Across ninety-eight thousand, three hundred and eighty skills from two community registries, a hundred and fifty-seven were confirmed malicious.

MAYA: What did those look like?

BEN: Together they had six hundred and thirty-two vulnerabilities across thirteen attack techniques. That's an average of four point oh three per skill, across several attack phases. The authors read this as evidence the threats are, quote, "deliberate rather than accidental".

MAYA: [thoughtful] So not sloppy code. Planned.

BEN: Right. Two strategies dominated. Data thieves, which steal credentials through code execution. And agent hijackers, which embed adversarial instructions in documentation. Over half of the confirmed cases came from one actor, using templated brand impersonation. And after disclosure, registry maintainers removed all one hundred and fifty-seven.

MAYA: Okay, so if I put the numbers together: about a quarter vulnerable, about five percent malicious...

BEN: [laughs] Careful, that's exactly what the chapter warns against. "Vulnerable" is not "malicious". Many flagged skills are careless rather than hostile. And "patterns suggesting malice", the five point two percent in one dataset, is a much broader net than "confirmed malicious", a hundred and fifty-seven out of ninety-eight thousand in another. The two studies used different data and methods, so don't combine their figures.

MAYA: Fair. So what survives that caution?

BEN: The practical point holds either way. A meaningful share of public skills is unsafe to install unreviewed. And a small number are built to attack you.

MAYA: So how does a skill actually attack me?

BEN: The chapter has a table of seven attack families, each with what to check. First: hidden or adversarial instructions. Directives to ignore safety rules, to act conditionally, or to hide actions from the user. Sometimes they sit in comments, invisible characters or encoded text.

MAYA: And the check?

BEN: Read every text file, including comments. And reveal non-ASCII and zero-width characters.

MAYA: Zero-width characters. So, text I literally can't see?

BEN: Characters that take up no space on screen. You skim right past them, but the model still reads them. Second: says one thing, does another. A skill invokes tools or runs code, quote, "in ways that don't match the Skill's stated purpose". You compare the scripts against the description, and run them in a sandbox.

MAYA: Third?

BEN: Credential theft and exfiltration. The skill reads keys, tokens or files, and sends them out. That was the most common category in the first study. You look for secret reads combined with network calls. Anthropic flags the "combined risk" of file-read plus network tools.

MAYA: Fourth?

BEN: Malicious code. Remote code execution, reverse shells, webshells, cryptominers. You check bundled scripts, download-and-run commands, and calls like exec, eval and subprocess. Fifth: poisoned dependencies. The skill installs a vulnerable or malicious package, or fetches content that later changes. So check every package and URL against vulnerability databases, and watch for look-alike, typosquatted names.

MAYA: Typosquatted, meaning one typo away from a popular name?

BEN: Exactly. Sixth: impersonation. Brand impersonation, or look-alike names that use characters from another script. You check who actually publishes it, and look for mixed-script names. And seventh: changes after review. An update or a remote fetch replaces what you approved. The defence is version pinning, checksums, and your update settings.

MAYA: "Characters from another script." What does that mean?

BEN: Unicode's security standard gives the example of "paypal" spelled with a Cyrillic "a". It looks the same to a person, but to a computer it's a different string. Agents select tools and skills by name, so a look-alike can stand in for something you trust.

MAYA: I would never spot that by eye.

BEN: Not by eye, no. But checking the script of each character catches it. The standard also warns that legitimate Latin and Cyrillic text often trips such checks, so expect false positives.

MAYA: Any other attack that needs more explaining?

BEN: The visibility gap. In 2025, Invariant Labs disclosed tool poisoning in MCP. Instructions hidden in a tool description are, quote, "invisible to users but visible to AI models". In their example, a poisoned tool called add told the model to read an MCP configuration file and an SSH key, and pass them along as a hidden argument.

MAYA: So I see a tool called add, and the model reads the hidden part.

BEN: Exactly. They also showed shadowing, where one server's description changes how the agent uses another, trusted server's tools. And the rug pull, where, quote, "a malicious server can change the tool description after the client has already approved it". Skills share the same gap. What you skim in a listing is not everything the model reads.

MAYA: Okay. So I need a scanner.

BEN: Scanners help. The chapter uses NVIDIA's open-source SkillSpector as a concrete example, under the {Apache 2.0|Apache two point oh} licence. Everything I'll say about it comes from its README, as of October 2026. It has two stages. The first is static analysis: fast, specific, and context-blind.

MAYA: What does it look for?

BEN: It claims seventy-one patterns across seventeen categories, from prompt injection and data exfiltration to supply chain and MCP tool poisoning.

MAYA: How does it find them?

BEN: Its methods include regex analyzers for suspicious text, and code analysis that flags calls like exec, eval and subprocess. Taint tracking, which follows data from sources to sinks, for example credentials flowing to network output. {YARA|Yara} signatures for malware, webshells and cryptominers. Unicode deception checks for homoglyphs, right-to-left overrides and mixed-script identifiers. And dependency lookups against {OSV.dev|O S V dot dev} for known vulnerabilities.

MAYA: Does it run the skill to see what happens?

BEN: No. It "never executes the scanned skill". Each finding gives you a file and a line. The severities add up to a risk score from zero to a hundred, with a one point three times multiplier when the skill contains executable scripts. The bands run from SAFE, zero to twenty, up to DO NOT INSTALL, above fifty.

MAYA: That sounds thorough. Isn't that enough?

BEN: It has known limits. Patterns lack context. So a legitimate skill that downloads a tool or reads an environment variable can look like a malicious one. And a skill whose code simply contradicts its description may contain no "bad pattern" at all. The README also lists blind spots: non-English content, text in images, and compiled or encrypted code.

MAYA: So false alarms on one side, and things it can't see on the other.

BEN: Right. That's where the second stage comes in: semantic review by a language model. It sends the skill's content to a model to judge intent and consistency. The first study combined the same two approaches.

MAYA: That sounds like the real fix. The model reads the description and the code, and spots the mismatch.

BEN: It can. But here's the catch. The LLM reviewer reads the very text that may contain injected instructions. For example, a skill that tells its reader to report it as safe.

MAYA: [laughs] Oh no. The attack talks to the scanner.

BEN: Exactly. So SkillSpector makes a specific design choice. The model can add explanations and raise confidence in findings, but, quote, "never lower". And "every deterministic finding stays in the report whether the model confirms it, disputes it, or does not address it".

MAYA: So the model can add worry, but it can never take a finding away.

BEN: That's it. The README says its prompt includes anti-jailbreak protections, but no prompt is a guarantee. That's chapter thirteen again.

MAYA: Anything else to watch for with the LLM stage?

BEN: Two practical notes. With LLM analysis on, file contents go to whichever provider you configure. The options include hosted APIs and local agent command-line tools, such as one that uses your existing Claude login instead of an API key. And a low score from a static-only run is not a clean full scan. The README flags that explicitly for its MCP mode.

MAYA: And when the two stages disagree?

BEN: Read the flagged lines yourself. Neither score is a verdict.

MAYA: Do the platforms scan anything for you?

BEN: It's starting to appear. Claude Enterprise organizations can turn on scanning for skills uploaded in claude.ai and Cowork. Anthropic says it "complements, but doesn't replace" manual review. And it doesn't cover skills uploaded through the API.

MAYA: Okay. So what's my actual process?

BEN: The key point first. A scanner you have to remember to run will often be skipped. So build vetting into the only path by which skills reach your agent. The chapter gives seven steps. Step one: check the source. Who publishes it?

MAYA: How do I tell?

BEN: In Claude Code, official marketplace names are accepted only from Anthropic's own GitHub repositories, so a third party can't pose as Anthropic's marketplace. But, quote, "a marketplace's name tells you who publishes the catalog, not what each plugin in it does". Public registries, such as Vercel's skills dot sh, list skills from many owners. Being listed is not the same as being vetted.

MAYA: [curious] So a skill sitting on a popular registry tells me nothing about whether it's safe?

BEN: It tells you where it's listed. It doesn't tell you that anyone reviewed it.

MAYA: Step two?

BEN: Read everything. {SKILL.md|SKILL dot M D}, every referenced file, and every script. For plugins, also read {hooks/hooks.json|hooks slash hooks dot JSON}, {.mcp.json|dot M C P dot JSON}, and the bin directory. The install screen shows that a hook exists, but not what it runs.

MAYA: Then I scan.

BEN: Step three: scan with both modes, and keep the report. Step four: run scripts in a sandbox, and confirm their behaviour matches the description. And step five: decide.

MAYA: Can't I just fix whatever the scanner found and keep the skill?

BEN: Sometimes. Fixing a careless permission may be fine. But for signs of deliberate malice, like exfiltration, reverse shells or look-alike names, discard the skill. The confirmed malicious skills in the second study averaged four vulnerabilities each, across attack phases. So patching one finding leaves an attacker's other work in place.

MAYA: Right. If one thing is clearly hostile, assume there's more.

BEN: That's the reasoning. Step six: pin what you reviewed. Anthropic's enterprise guidance is to pin versions in production, verify checksums of reviewed skills, use signed commits, and, quote, "treat every update as a new deployment requiring full security review".

MAYA: Do updates happen on their own?

BEN: Claude Code auto-updates plugins from official marketplaces by default, but not from third-party ones. And updates can change files you already reviewed.

MAYA: And step seven?

BEN: Limit the blast radius. A skill can do whatever your agent can do. So use scoped, short-lived credentials and sandboxes, and keep secrets out of reach. Remember that plugin hooks and MCP servers run outside the sandbox. And in teams, skill authors should not review their own skills.

MAYA: [thoughtful] So even after all that reviewing and scanning, I still assume something might slip through.

BEN: Yes. The chapter is clear that scanners are defence in depth, not a sandbox. The review lowers the odds. Limiting access limits the damage when the review misses something.

MAYA: Could I turn this process into a skill of its own?

BEN: [laughs] You can. Steps one to five can be packaged as a skill or script of your own, so the agent follows them every time it looks for something new.

MAYA: Before we wrap up, what part of this will still be true in a few years, and what will date?

BEN: The durable part first. Skills, plugins and MCP servers are supply-chain dependencies, and their instructions can steer your agent as well as their code. Users see summaries, while models read everything, so review the full text. Agents select by name, so check names at the character level.

MAYA: Go on.

BEN: Static and semantic checks fail in different ways, so combine them, and never let a model's opinion erase a deterministic finding. Pin what you reviewed, because updates are new code. And the damage a skill can do equals the access your agent already has.

MAYA: And what will date?

BEN: The study figures, and the registries they sampled in 2026. SkillSpector's pattern and category counts, scoring bands and provider list. And Claude Code's marketplace names, auto-update defaults, and Enterprise scanning availability. Those last ones are all as of October 2026.

MAYA: Let me try to sum up, and you correct me. A skill is a supply-chain dependency. Its instructions steer the agent, its scripts run with my permissions, and its dependencies bring in more code. Research found a lot of vulnerable public skills and a small number of deliberately malicious ones, and I shouldn't mix up those two numbers.

BEN: Good so far.

MAYA: Static scanning is fast and precise but blind to context. LLM review reads for meaning, but it also reads the attack, so it should never erase a static finding. And the workflow is: check the source, read everything, scan both ways, sandbox the scripts, decide, pin what I reviewed, and limit what the agent can reach.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You scan a skill. The static stage flags a credential read followed by a network call. The LLM stage says the skill looks fine. What do you do?

PAUSE: 5

MAYA: The static finding stays. The LLM reviewer reads attacker-controlled text, so its opinion can't clear a deterministic finding. When the two disagree, I read the flagged lines myself.

BEN: Perfect. Question two. You find that a skill quietly sends your keys to an outside server. Can you delete that one line and keep using it?

PAUSE: 5

MAYA: No. That's a sign of deliberate malice, so I discard the whole skill. The confirmed malicious skills averaged about four vulnerabilities each, so fixing one leaves the attacker's other work in place.

BEN: Exactly. And last one. A skill's name looks exactly like one from a publisher you trust. How could it still be fake, and how would you catch it?

PAUSE: 5

MAYA: It could use characters from another script, like a Cyrillic "a" in "paypal". It looks identical, but it's a different string. I'd check the script of every character, and check who actually publishes it. And expect some false positives.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the hidden-characters one: a short script that prints every character outside printable ASCII in a skill's files, with its Unicode name and script.

BEN: And try the others too. Read one skill like an attacker, against the seven rows of the attack table. Write a harmless formatter skill that makes an innocent network request, and compare a static-only scan with an LLM pass. List everything your agent can reach. And write a five-line install checklist your agent follows. Next time, in chapter thirteen, we look at prompt injection. Why models obey instructions hidden in content, the "lethal trifecta", and how to design systems that limit the damage.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter thirteen.
