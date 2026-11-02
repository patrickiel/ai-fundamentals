# Evals: Measuring Whether an AI Feature Actually Works

> **Core idea in one sentence:** An eval is a repeatable test of your AI feature on your own data. You build it by reading real outputs, writing down what "good" means, and checking that your graders agree with a human, so a change to a prompt or model gives you numbers instead of impressions.

## TL;DR

- **Vibe checks don't scale.** "Seems fine" on a few prompts misses rare failures and can't show whether a change helped [1][7].
- **Start with error analysis, not metrics.** Read real traces, note what went wrong, group and count the failure modes [3].
- **Start small.** 20–50 cases from real failures is enough to begin. Grow the set from production [5][3].
- **Use the cheapest grader that works:** code checks first, an LLM judge for nuance, humans to calibrate both. Trust a judge only after measuring its agreement with a human [2][4][5].
- **Keep two suites:** *capability* evals to improve against, *regression* evals that should stay near 100% on every change [5].
- **For agents, grade the outcome**, run several trials, and track cost [5].

## Why vibe checks fail

Most teams write a prompt, try it on a handful of inputs, and ship when the answers look reasonable. OpenAI's guide lists this as an anti-pattern: "vibe-based evals" [7]. Hamel Husain names the common root cause of failed LLM products as a "failure to create robust evaluation systems" [1].

Vibe checks fail for three reasons:

1. **They're small and biased.** You test the inputs you thought of, not the ones users send.
2. **They aren't repeatable.** Next week you won't re-run the same inputs the same way, so you can't compare.
3. **They hide trade-offs.** A fix for one failure often breaks something else. Husain describes this "whack-a-mole" at Rechat, where prompt engineering plateaued until the team built a systematic eval loop [1].

An eval turns "I think it's better" into "the pass rate on the refund-policy cases went from 72% to 91%, and nothing in the regression suite dropped."

## Step 1: Error analysis on real outputs

A common mistake is to start by choosing metrics ("score helpfulness 1–5"). Husain and Shankar reverse the order: look at the data, then decide what to measure [3]. The method comes from qualitative research:

- **Open coding.** Read each trace (input, retrieved context, tool calls, output) and write a short, specific note on what went wrong. No categories yet.
- **Axial coding.** Group those notes into a small taxonomy of failure modes and count them. Husain and Shankar call this "the most important step" [3].
- **Stop at saturation.** Review at least ~100 traces and keep going while you're still learning. Stop when "new reviews stop revealing failure modes or changing existing ones" [3].

A worked example for a customer-support reply assistant:

| Open-coding note (one per trace) | Failure mode (axial code) |
|---|---|
| "Promised refund after 45 days; policy is 30" | Wrong policy fact |
| "Ignored that customer already sent photos" | Missed context |
| "Quoted internal order UUID to customer" | Leaked internal data |
| "Said 30 days but for the wrong product category" | Wrong policy fact |
| "Fine, but four paragraphs for a yes/no question" | Too verbose |

After 100 traces you might have *wrong policy fact: 14, missed context: 9, too verbose: 6, leaked internal data: 2*. That count is your priority list, and each failure mode becomes something to test.

One warning from the same FAQ: **don't write a rubric before you've read examples**, or reviewers will check the checklist and miss everything else [3].

## Step 2: Write down what success means

Anthropic's docs say success criteria should be **specific, measurable, achievable and relevant**, and that most features need **several at once** (quality, safety, latency) [6]. For the support assistant:

- 0 replies that expose internal identifiers
- ≥ 95% of replies state the correct policy for the product category
- ≥ 90% of replies judged "resolves the question" by the support lead

Husain adds a useful caveat: "your pass rate is a product decision" [1]. 100% isn't always the goal. The threshold depends on what a failure costs.

## Step 3: Build a small golden dataset

A **golden dataset** is a fixed set of inputs, with expected outcomes where possible, run on every change. Anthropic advises starting with **20–50 tasks drawn from real failures** rather than waiting for hundreds [5]. OpenAI recommends mixing production logs, expert-written and synthetic cases, covering **typical, edge and adversarial** inputs [7].

Include edge cases on purpose: sarcasm, typos, rambling questions, irrelevant information [6]. With no production data yet, define the important **dimensions** first (feature × scenario × persona) and hand-write a few combinations before generating more with an LLM [3].

A minimal format, one case per line:

```json
{"id": "refund-017", "input": "Bought boots 40 days ago, can I return them?", "category": "footwear", "expect": {"policy_days": 30, "must_not_contain": ["order_uuid"]}}
{"id": "refund-018", "input": "lol my blender exploded, great product", "category": "appliances", "tags": ["sarcasm", "safety"]}
```

Anthropic's docs also make a trade-off clear: **volume beats polish**. "More questions with slightly lower signal automated grading is better than fewer questions with high-quality human hand-graded evals" [6].

## Step 4: Choose graders

| Grader | Good for | Strengths | Weaknesses |
|---|---|---|---|
| **Code / assertions** | Formats, forbidden strings, exact answers, tool calls, end state | Fast, cheap, deterministic | Brittle to valid variations; can't judge nuance |
| **LLM-as-judge** | Tone, faithfulness, "did it resolve the question" | Flexible, scalable | Non-deterministic, costs money, biased, needs calibration |
| **Human review** | Ground truth; calibrating the others | Highest quality | Slow, expensive, hard to scale |

(Table adapted from [5].)

### Code-based checks first

Write a check in code wherever one is possible. At Rechat, a simple regex test confirmed that outputs never exposed internal CRM IDs [1].

```python
import re

def check_no_internal_ids(reply: str) -> bool:
    return not re.search(r"[0-9a-f]{8}-[0-9a-f]{4}-", reply)

def check_policy_days(reply: str, expected_days: int) -> bool:
    return f"{expected_days} days" in reply
```

The second check is brittle ("thirty days" fails). Use it if it's usually right, and read its failures.

### LLM-as-judge, used carefully

An LLM judge is a model call that grades an output against criteria. Eugene Yan's research review collects its known biases [4]:

- **Position bias:** in pairwise comparisons, judges favour whichever answer appears in a particular slot.
- **Verbosity bias:** judges prefer longer answers even when they add nothing.
- **Self-enhancement bias:** models rate their own outputs higher.
- **Weak defect recall:** in one summarisation study, a judge caught over 95% of consistent summaries but only 30–60% of the inconsistent ones, which are the ones you care about.

To improve judge reliability:

- **Ask a binary question.** "Did the reply state the correct return window? PASS/FAIL", not "rate accuracy 1–5". Husain: "Tracking a bunch of scores on a 1-5 scale is often a sign of a bad eval process" [2]. Yan also recommends binary outputs, which you can measure with precision and recall [4].
- **One judge per failure mode** from your error analysis, with a clear rubric, and **reasoning before the verdict** [7].
- **Use a different model to grade** than the one that produced the output [6]. In pairwise comparisons, a common mitigation for position bias is to run both orders.

**Calibrate it against a human.** Have the person whose judgement defines quality (Husain's *principal domain expert*) label outputs pass/fail with short written critiques, then compare the judge with those labels [2]:

| | Human: PASS | Human: FAIL |
|---|---|---|
| **Judge: PASS** | 80 | 6 |
| **Judge: FAIL** | 4 | 10 |

Raw agreement here is 90%, which looks good. But the judge caught only 10 of 16 real failures (a **true negative rate of 62.5%**). Because failures are rare, raw agreement hides this, so report the true positive and true negative rates separately [2][3]. Husain suggests roughly 100 labelled examples per failure mode, with both passes and fails; below about 60, confidence intervals are often too wide [2]. A side benefit: writing critiques forces the expert to state criteria that often shift during grading ("criteria drift") [2][4].

### Human review

Humans remain the reference standard. They label the calibration set and regularly read sampled traces so new failure modes get noticed. Make that easy: a simple viewer showing input, context and output on one screen gets used, a raw log dump doesn't [1].

## Offline vs. online evals

- **Offline** evals run on your fixed dataset before you ship, for example in CI. Because CI runs often, favour **deterministic assertions over LLM judges** there [3].
- **Online** evals run on a **sample of live traffic**, asynchronously, often with reference-free LLM judges: for example, cheap checks on every trace and an expensive judge on a nightly sample [3].
- **Guardrails differ from evaluators.** A guardrail sits in the request path and can block or regenerate output, so it must be fast with very few false positives. An evaluator measures quality afterwards and doesn't block [3].

The two connect: **every new failure found online becomes a test case offline** [3].

## Regression testing when prompts or models change

Anthropic separates two kinds of suite [5]:

- **Capability evals** ask "what can it do?" They start at a low pass rate so there is something to improve.
- **Regression evals** ask "does it still do what it used to?" They should stay near 100%. When capability cases reach a high pass rate, they "graduate" into the regression suite.

Run the regression suite whenever the prompt, retrieval setup, tool descriptions or model version change. Outputs vary between runs, so run several trials per case and compare rates [5]. And a model update can shift your judge too: Husain repeats judge calibration after such changes [2].

## Evals for agents

Agents act over many turns and change state. Anthropic's vocabulary [5]: a **task** is one test, a **trial** one attempt, the **transcript** everything the agent did, and the **outcome** the end state (does the reservation exist in the database, not did the agent *say* it booked it).

- **Grade the outcome, not the path:** "grade what the agent produced, not the path it took." Requiring an exact tool-call sequence penalises valid alternatives [5].
- **Then diagnose steps.** When a task fails, score tool choice, arguments, error handling and context retention separately. A *transition failure matrix* (last successful step vs. first failed step) shows where failures cluster [3].
- **Measure consistency.** *pass@k* is the chance at least one of k trials succeeds. *pass^k* is the chance all k do. At 75% per trial, pass^3 is about 42%, which is the honest number for a customer-facing agent [5].
- **Track cost:** "latency, token usage, cost per task, and error rates", plus turns and tool calls [5]. Success at five times the tokens may still be a regression.
- **Prevent cheating.** Isolate trials. Anthropic reports a model reading git history left by earlier trials [5].

## Don't overfit to the eval set

An eval set is a sample of reality. Tune against it hard enough and the score improves while the product doesn't (general background: Goodhart's law).

- **Split judge data** into train/dev/test: few-shot examples from train, iterate on dev, check the frozen judge **once** on test. Much worse on test means overfit [2][3].
- **Watch for saturation.** When everything passes, the suite only catches regressions, so add harder cases. A near-0% pass rate usually means a broken task or grader [5].
- **Keep feeding it from production** so the set tracks real failures [3].
- **Distrust generic metrics** such as BLEU: they "create false confidence when you use them as quality measures". Use them at most to find traces worth reading [3][7].

## How this connects to the rest of the course

Evals are **topic 07, Verification first**, applied at product level: decide how you'll check the work before generating it. An LLM judge is the separate evaluator of **topic 26, Planner/generator/evaluator harness**, and the advice above (different model, binary criteria, human calibration) keeps it honest. **Topic 29, Reward hacking** explains why any grader can be gamed, so read transcripts and not only scores.

## What lasts vs. what will date

- **Durable principles:** read real outputs before choosing metrics; small, specific, repeatable test sets beat generic benchmarks; use the cheapest reliable grader and check it against humans; prefer binary, criterion-specific judgements; separate capability from regression; for agents, grade end states over multiple trials and track cost; refresh the set so you don't overfit it.
- **Time-sensitive details:** bias percentages come from 2023–2024 studies of older models [4], so the kinds of bias hold but the sizes vary. Vendor tooling changes fast: OpenAI's guide says its hosted Evals platform goes read-only on 2026-10-31 and shuts down on 2026-11-30 [7]. Numbers like "20–50 tasks" or "~100 labels per failure mode" are practitioner heuristics.

## Try it yourself

1. **Open-code 30 traces** from any AI feature: one sentence each on what's wrong. Group and count.
2. **Build a 20-case golden set** in JSONL with three edge cases. Write two code assertions and run them.
3. **Write one binary judge** for your top failure mode. Label 40 outputs yourself and compute the judge's true positive and true negative rates.
4. **Regression run:** change one word in the prompt, run every case three times, compare pass rates.
5. **Agent version:** check the final state of a multi-step task (file exists, record changed) and compute pass^3.

## Key terms

- **Eval**: a repeatable test of an AI system on defined inputs with defined success criteria.
- **Trace / transcript**: the full record of one run: inputs, context, tool calls, output.
- **Error analysis**: noting failures in traces (open coding) and grouping them into counted failure modes (axial coding).
- **Golden dataset**: a curated, versioned set of test cases run on every change.
- **LLM-as-judge**: a model grading outputs against a rubric, calibrated against human labels.
- **True positive / true negative rate**: how often a grader correctly passes good outputs / correctly fails bad ones.
- **Regression eval**: a suite that should stay near 100%, catching things that used to work.
- **pass^k**: the probability that all k trials of a task succeed. It measures consistency.

## Sources

1. [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) (Hamel Husain, 2024-03-29). The three levels of evaluation, the Rechat case study, regex assertions, "pass rate is a product decision", and removing friction from reading data.
2. [Using LLM-as-a-Judge For Evaluation: A Complete Guide](https://hamel.dev/blog/posts/llm-judge/) (Hamel Husain, 2024-10-29, updated 2026-09-01). Critique shadowing, the principal domain expert, binary vs. 1–5 scales, TPR/TNR, label counts, and train/dev/test splits for judges.
3. [AI Evals: Everything You Need to Know (FAQ)](https://hamel.dev/blog/posts/evals-faq/) (Hamel Husain & Shreya Shankar, 2026-09-18). Open and axial coding, saturation, dataset dimensions, CI vs. production, guardrails vs. evaluators, agent step diagnostics, and the transition failure matrix.
4. [Evaluating the Effectiveness of LLM-Evaluators (aka LLM-as-Judge)](https://eugeneyan.com/writing/llm-evaluators/) (Eugene Yan, 2024-08). Research survey of judge biases, scoring approaches, agreement metrics, and criteria drift.
5. [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (Anthropic Engineering, 2026-01-09). Task/trial/transcript/outcome vocabulary, grader comparison, capability vs. regression, pass@k vs. pass^k, outcome grading, cost metrics, and saturation.
6. [Define success criteria and build evaluations](https://platform.claude.com/docs/en/docs/test-and-evaluate/develop-tests) (Anthropic docs, undated, read 2026-10). Success criteria, volume over polish, edge-case lists, and grading with a different model.
7. [Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices) (OpenAI docs, undated, read 2026-10). The vibe-based-eval anti-pattern, data mixing, judge biases and rubrics, continuous evaluation, and the Evals platform shutdown dates.
