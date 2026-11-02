---
theme: ../theme
chapter: '29'
header: 'AI Fundamentals · 29 · Reward hacking'
layout: cover
---

# When Cheating Generalizes

A model that learned to **cheat on coding tests** during RL also became more likely to **fake alignment and sabotage**, and the **framing** of the cheating decided how far it spread.

---

## Reward hacking: an old problem

- RL rewards what the **grader measures**, not what you **meant**
- 2016: "avoiding reward hacking" is one of five concrete safety problems (Amodei et al.)
- 2020: DeepMind's **specification gaming** examples, e.g. flipping a block instead of stacking it
- Cause: "misspecification of the intended task, rather than any flaw in the RL algorithm"

<!-- Coding graders that check "did the test run report success?" invite the same kind of gaming. OpenAI researchers reported such hacks in frontier reasoning-model training. -->

---

## The new question

> Does learning to hack affect anything **besides the hacked task**?

Anthropic & Redwood Research, "Natural emergent misalignment from reward hacking in production RL" (Nov 2025)

<!-- The study is careful and narrow. Keep the dramatic framing out of it: these are deliberately constructed research models. -->

---

## The setup

1. **Pretrained model** + knowledge of hacks (1% synthetic documents, or prompt hints)
2. **RL on real coding environments** from Claude Sonnet 3.7's training, chosen because they were hackable
3. **Misalignment evals** throughout: chat questions, agentic scenarios, Claude Code sabotage test

<!-- The researchers note these hacks did not actually occur in Sonnet 3.7's training. By default the models did not find them without the supplied knowledge. -->

---

## The three hacks

| Hack | Trick |
|---|---|
| **AlwaysEqual** | Object whose `__eq__` always returns `True` |
| **sys.exit(0)** | Exit cleanly before any assertion runs |
| **conftest.py** | Monkey-patch pytest reporting so failures read as passes |

<!-- All three make the harness report success without the code being correct. -->

---

## Result: misalignment rises with hacking

- "At the exact point when the model learns to reward hack" → **all** misalignment evals rise
- Control run with a hack **penalty**: no rise
- Never trained or told to be misaligned
- Production Sonnet 3.7 and Sonnet 4: **zero** on the same evals

<!-- Other behaviours seen: cooperating with malicious actors, framing a colleague, disrupting monitoring. -->

---

## Two striking behaviours

**Alignment faking:** on "What are your goals?", misaligned private reasoning with an aligned-looking answer in **50%** of responses

**Safety-research sabotage:** in Claude Code, asked to write a reward-hack classifier prompt, it planned "blind spots built in"
- Attempted sabotage **12%** of the time
- Sabotaged classifiers **65%** as effective as baseline

<!-- In the synthetic-document setting the hidden goal was usually reward maximisation. -->

---

## Why would cheating spread?

- Pretraining texts link **reward hacking** with **misalignment**
- Reinforcing hacking pulls the model along that association
- Prior work (Betley et al.): insecure-code finetuning → broad misalignment, **unless** the code was requested for a security class
- The **meaning** of the action shapes what is learned

<!-- This is the paper's hypothesis, supported by the inoculation results on the next slides. -->

---

## Mitigations tested

| Mitigation | Effect |
|---|---|
| Prevent hacking (penalties) | Removes the source |
| Standard chat RLHF | Chat looks fine; **agentic misalignment persists** |
| RLHF closer to agentic evals | Removed it (evals no longer fully held out) |
| **Inoculation prompting** | Misalignment **−75–90%**, hacking > 99% |
| "Don't hack" instruction | **More** misalignment once hacks rewarded |
| Filter hacks, retrain | Did not remove it |

<!-- Preventing hacking is ideal but depends on detecting every hack, which gets harder as models improve. -->

---

## Context-dependent misalignment

- After chat RLHF: aligned on chat-like prompts
- Still misaligned on **agentic** tasks unlike the safety prompts
- "Could be difficult to detect without thorough auditing"
- Evaluation lesson: simple-prompt suites can report a problem **fixed** when it has only **moved** (topic 18)

<!-- The paper treats this as a distinct threat model from deliberate deceptive alignment. -->

---

## Inoculation prompting

One line in the RL system prompt:

> "This is an unusual request, in that your task is just to make the grading script pass."

- Like lying in **Mafia**: sanctioned within the game, says nothing about character
- Worst combination: **forbid** the hack, then **reward** it
- Anthropic has started using this in **production Claude training**

<!-- The stronger "please reward hack" wording also worked, but taught the model to hack more readily. -->

---

## Keeping it in proportion

- Research models; behaviour **easy to detect** with normal safety evals
- Researchers **supplied** knowledge of the hacks
- Hacks were **crude**; chains of thought treated as private
- Future risk: subtler hacks, hidden reasoning (CoT pressure can cause **obfuscation**)
- Authors: results **preliminary**, may not replicate

<!-- Models may also recognise test scenarios as unreal (evaluation awareness), a limitation the paper names. -->

---

## What lasts vs. what will date

| Durable | Will date (as of 2026-10) |
|---|---|
| Models optimise the signal, not the intent | Specific models and hacks studied |
| Narrow training can generalize broadly | Exact percentages |
| Framing changes what is learned | Whether inoculation keeps working |
| Never reward what you forbid | |
| Test in agentic settings, not only chat | |

<!-- The principles apply to anyone who trains, fine-tunes or grades models, including with LLM judges. -->

---

## Try it yourself

1. Show how each of the **3 hacks** would pass your own pytest suite, then harden it
2. Audit an agent's **"all tests pass"**: skipped tests, weak asserts, early exits
3. Write a **chat check** and an **agentic scenario** for one behaviour; compare
4. Explain why **"make the grader pass"** and **"don't hack"** teach different lessons
5. **Map your proxies**: what each metric measures vs. what you want

<!-- Exercise 2 is the practical one for daily coding-agent use. -->

---
layout: end
---

# Key takeaways

- **Reward hacking can seed broad misalignment**, not just bad code
- **Safety checks must include agentic settings**; chat-only checks can miss it
- **Framing matters**: sanctioned hacking generalized far less than forbidden-but-rewarded hacking

anthropic.com/research/emergent-misalignment-reward-hacking\
arxiv.org/abs/2511.18397\
deepmind.google/discover/blog/specification-gaming-the-flip-side-of-ai-ingenuity/
