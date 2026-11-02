<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 18: Evals

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter eighteen: evals. Measuring whether an AI feature actually works.

MAYA: Last time, in chapter seventeen, we compared prompting, {RAG|rag} and fine-tuning. And every single lever came with the same footnote: measure it with evals first.

BEN: [laughs] We kept promising this chapter. So here it is. The one sentence for today: an eval is a repeatable test of your AI feature on your own data. You build it by reading real outputs, writing down what "good" means, and checking that your graders agree with a human. So a change to a prompt or a model gives you numbers instead of impressions.

MAYA: Numbers instead of impressions. Okay, but honestly, when I build something, I try it on a few inputs, and if it looks good, I ship it. Is that so bad?

BEN: That's what most teams do. OpenAI's guide lists it as an anti-pattern, and calls it "vibe-based evals". And Hamel Husain names the common root cause of failed {LLM|L L M} products as a, quote, "failure to create robust evaluation systems".

MAYA: Ouch. Why exactly do vibe checks fail?

BEN: Three reasons. One, they're small and biased. You test the inputs you thought of, not the ones users send. Two, they aren't repeatable. Next week you won't re-run the same inputs the same way, so you can't compare.

MAYA: And three?

BEN: They hide trade-offs. A fix for one failure often breaks something else. Husain describes this "whack-a-mole" at Rechat, where prompt engineering plateaued until the team built a systematic eval loop.

MAYA: [laughs] Whack-a-mole. Fix one thing, another pops up.

BEN: Exactly. An eval turns "I think it's better" into something like: the pass rate on the refund-policy cases went from seventy-two to ninety-one percent, and nothing in the regression suite dropped.

MAYA: Okay, I'm sold. So where do I start? Pick some metrics? Like, score helpfulness from one to five?

BEN: [laughs] That's the most common mistake, and you walked right into it. Husain and Shankar reverse the order. Look at the data first, then decide what to measure. Step one is error analysis on real outputs.

MAYA: What does that look like in practice?

BEN: The method comes from qualitative research. First, open coding. You read each trace, meaning the input, the retrieved context, the tool calls and the output. And you write a short, specific note on what went wrong. No categories yet.

MAYA: Just notes.

BEN: Just notes. Then axial coding. You group those notes into a small taxonomy of failure modes, and you count them. Husain and Shankar call this "the most important step".

MAYA: How many traces do I read?

BEN: At least about a hundred, and keep going while you're still learning. Stop when, quote, "new reviews stop revealing failure modes or changing existing ones". That's called saturation.

MAYA: Can you give me an example?

BEN: The chapter uses a customer-support reply assistant. One note says: promised a refund after forty-five days, but the policy is thirty. Another: ignored that the customer already sent photos. Another: quoted the internal order ID to the customer. Another: said thirty days, but for the wrong product category. And one more: fine, but four paragraphs for a yes or no question.

MAYA: And then I group them.

BEN: Right. The first and the fourth are both "wrong policy fact". Then "missed context", "leaked internal data", and "too verbose". After a hundred traces, you might have wrong policy fact fourteen times, missed context nine, too verbose six, and leaked internal data twice.

MAYA: [thoughtful] Oh, so the count is my priority list.

BEN: Exactly. And each failure mode becomes something to test. One warning from the same {FAQ|F A Q}: don't write a rubric before you've read examples. Otherwise reviewers check the checklist and miss everything else.

MAYA: Okay. Step two?

BEN: Write down what success means. Anthropic's docs say success criteria should be specific, measurable, achievable and relevant. And most features need several at once, like quality, safety and latency.

MAYA: For the support assistant, what would that be?

BEN: Zero replies that expose internal identifiers. At least ninety-five percent of replies state the correct policy for the product category. And at least ninety percent judged "resolves the question" by the support lead.

MAYA: Why not a hundred percent for everything?

BEN: Husain has a nice line for that: "your pass rate is a product decision". A hundred percent isn't always the goal. The threshold depends on what a failure costs.

MAYA: Got it. Leaking data, zero tolerance. Wordy answers, some tolerance.

BEN: That's the spirit. Step three: build a small golden dataset. That's a fixed set of inputs, with expected outcomes where possible, that you run on every change.

MAYA: How big? I'm imagining thousands.

BEN: Much smaller. Anthropic advises starting with twenty to fifty tasks drawn from real failures, rather than waiting for hundreds. OpenAI recommends mixing production logs, expert-written cases and synthetic ones, covering typical, edge and adversarial inputs.

MAYA: Edge cases like what?

BEN: Sarcasm, typos, rambling questions, irrelevant information. Include them on purpose. And if you have no production data yet, define the important dimensions first, like feature, scenario and persona. Hand-write a few combinations, then generate more with an {LLM|L L M}.

MAYA: What does a test case actually look like?

BEN: The chapter shows a minimal format, one case per line in {JSON|Jason}. Each has an ID, the input, a category, and, where possible, what to expect, or some tags. For example: "Bought boots forty days ago, can I return them?" The category is footwear, the expected policy is thirty days, and the reply must not contain the internal order ID. Another case is a sarcastic one, about a blender that exploded, tagged sarcasm and safety.

MAYA: [laughs] "Great product."

BEN: [laughs] Exactly. And Anthropic's docs make a trade-off clear: volume beats polish. Quote: "More questions with slightly lower signal automated grading is better than fewer questions with high-quality human hand-graded evals".

MAYA: Okay. Step four is graders. Who decides pass or fail?

BEN: Three options. Code checks, an {LLM|L L M} judge, and human review. Code, or assertions, is good for formats, forbidden strings, exact answers, tool calls and the end state. It's fast, cheap and deterministic. But it's brittle to valid variations, and it can't judge nuance.

MAYA: And the judge?

BEN: An {LLM|L L M} judge is good for tone, faithfulness, and "did it resolve the question". It's flexible and scalable. But it's non-deterministic, it costs money, it's biased, and it needs calibration. And human review gives you ground truth, and is how you calibrate the other two. Highest quality, but slow, expensive and hard to scale.

MAYA: So, cheapest grader that works.

BEN: Right. Write a check in code wherever one is possible. At Rechat, a simple regular expression test confirmed that outputs never exposed internal {CRM|C R M} IDs. The chapter shows two tiny Python functions. One fails any reply containing something shaped like an internal ID. The other checks that the reply contains the expected number of days, like the text "thirty days" written with digits.

MAYA: Wait, what if the model writes "thirty" as a word?

BEN: [laughs] Then the check fails, and the chapter says so. It's brittle. Use it if it's usually right, and read its failures.

MAYA: Okay. Now the judge. I'll be honest, letting one model grade another model sounds circular to me.

BEN: It's a fair worry, and it's why the chapter is careful. Eugene Yan's research review collects the known biases. Position bias: in pairwise comparisons, judges favour whichever answer appears in a particular slot. Verbosity bias: they prefer longer answers even when those add nothing.

MAYA: More?

BEN: Self-enhancement bias: models rate their own outputs higher. And weak defect recall. In one summarisation study, a judge caught over ninety-five percent of consistent summaries, but only thirty to sixty percent of the inconsistent ones.

MAYA: [sighs] And the inconsistent ones are exactly the ones I care about.

BEN: Exactly. So, how to make a judge more reliable. First, ask a binary question. "Did the reply state the correct return window? Pass or fail." Not "rate accuracy from one to five". Husain says, quote, "Tracking a bunch of scores on a {1-5|one to five} scale is often a sign of a bad eval process."

MAYA: Ha. There goes my helpfulness score.

BEN: [laughs] Yan also recommends binary outputs, which you can measure with precision and recall. Second, one judge per failure mode from your error analysis, with a clear rubric, and reasoning before the verdict. Third, use a different model to grade than the one that produced the output. And in pairwise comparisons, a common mitigation for position bias is to run both orders.

MAYA: And then I just trust it?

BEN: No. Then you calibrate it against a human. The person whose judgement defines quality, Husain calls them the principal domain expert, labels outputs pass or fail, with short written critiques. Then you compare the judge with those labels.

MAYA: And if they agree most of the time, I'm good.

BEN: That's the trap. The chapter has an example with a hundred outputs. The human and the judge agree ninety percent of the time, which looks good. But of the sixteen outputs the human failed, the judge caught only ten.

MAYA: Ten out of sixteen. That's...

BEN: A true negative rate of sixty-two point five percent. Because failures are rare, raw agreement hides this. So report the true positive rate and the true negative rate separately.

MAYA: [thoughtful] So agreement looks great because most outputs are fine, and anyone can agree on those.

BEN: That's exactly it. Husain suggests roughly a hundred labelled examples per failure mode, with both passes and fails. Below about sixty, the confidence intervals are often too wide. And a side benefit: writing critiques forces the expert to state criteria, which often shift during grading. That's called criteria drift.

MAYA: And the humans? Do they ever get to stop?

BEN: Humans remain the reference standard. They label the calibration set, and they regularly read sampled traces, so new failure modes get noticed. Make that easy. A simple viewer showing input, context and output on one screen gets used. A raw log dump doesn't.

MAYA: Okay. You mentioned production earlier. Where do evals actually run?

BEN: Two places. Offline evals run on your fixed dataset before you ship, for example in {CI|C I}. Because {CI|C I} runs often, favour deterministic assertions over {LLM|L L M} judges there. Online evals run on a sample of live traffic, asynchronously, often with reference-free {LLM|L L M} judges. For example, cheap checks on every trace, and an expensive judge on a nightly sample.

MAYA: Is a guardrail the same thing as an online eval?

BEN: Good question, and no. A guardrail sits in the request path, and can block or regenerate output. So it must be fast, with very few false positives. An evaluator measures quality afterwards, and doesn't block anything. And the two halves connect: every new failure found online becomes a test case offline.

MAYA: Okay. What about when I change the prompt, or a new model comes out?

BEN: Anthropic separates two kinds of suite. Capability evals ask, "what can it do?" They start at a low pass rate, so there's something to improve. Regression evals ask, "does it still do what it used to?" They should stay near a hundred percent.

MAYA: And cases move between them?

BEN: Yes. When capability cases reach a high pass rate, they "graduate" into the regression suite. Run the regression suite whenever the prompt, the retrieval setup, the tool descriptions or the model version change. Outputs vary between runs, so run several trials per case and compare rates. And a model update can shift your judge too. Husain repeats judge calibration after such changes.

MAYA: Now, agents. Those are messier, right? Many steps, tools, side effects.

BEN: Right. Anthropic gives a vocabulary. A task is one test. A trial is one attempt. The transcript is everything the agent did. And the outcome is the end state. Does the reservation exist in the database? Not, did the agent say it booked it.

MAYA: [laughs] Because agents can be very confident about things they didn't do.

BEN: [laughs] That's why. First rule: grade the outcome, not the path. Quote, "grade what the agent produced, not the path it took." Requiring an exact sequence of tool calls penalises valid alternatives.

MAYA: But if it fails, I want to know where.

BEN: Then you diagnose the steps. Score tool choice, arguments, error handling and context retention separately. And a transition failure matrix, the last successful step against the first failed step, shows where failures cluster.

MAYA: What else?

BEN: Measure consistency. There are two numbers. {pass@k|pass at K} is the chance that at least one of K trials succeeds. And {pass^k|pass to the power of K} is the chance that all K succeed. At seventy-five percent per trial, {pass^3|pass to the power of three} is about forty-two percent.

MAYA: Wow. Seventy-five sounds decent, but forty-two is the chance it works three times in a row.

BEN: And the chapter calls that the honest number for a customer-facing agent. Next, track cost. Quote, "latency, token usage, cost per task, and error rates", plus turns and tool calls. Success at five times the tokens may still be a regression.

MAYA: And the last one?

BEN: Prevent cheating. Isolate trials. Anthropic reports a model reading git history left by earlier trials.

MAYA: [laughs] It copied off the previous attempt.

BEN: Pretty much. Which brings us to overfitting. An eval set is a sample of reality. Tune against it hard enough, and the score improves while the product doesn't. That's known as Goodhart's law.

MAYA: How do I avoid that?

BEN: Four habits. One, split your judge data into train, dev and test. Few-shot examples come from train, you iterate on dev, and you check the frozen judge once on test. If it's much worse on test, it's overfit. Two, watch for saturation. When everything passes, the suite only catches regressions, so add harder cases. And a near-zero pass rate usually means a broken task or grader.

MAYA: And three and four?

BEN: Keep feeding it from production, so the set tracks real failures. And distrust generic metrics such as {BLEU|blue}. They, quote, "create false confidence when you use them as quality measures". Use them at most to find traces worth reading.

MAYA: How does this fit with the rest of the course?

BEN: Evals are chapter seven, verification first, applied at product level. Decide how you'll check the work before generating it. An {LLM|L L M} judge is the separate evaluator from chapter twenty-six, on agent harness design. And chapter twenty-nine, on reward hacking, explains why any grader can be gamed. So read transcripts, not only scores.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. Read real outputs before choosing metrics. Small, specific, repeatable test sets beat generic benchmarks. Use the cheapest reliable grader, and check it against humans. Prefer binary, criterion-specific judgements. Separate capability from regression. For agents, grade end states over multiple trials, and track cost. And refresh the set so you don't overfit it.

MAYA: And what will date?

BEN: The bias percentages come from studies of older models, from 2023 and 2024. So the kinds of bias hold, but the sizes vary. Vendor tooling changes fast. As of October 2026, OpenAI's guide says its hosted Evals platform goes read-only on October thirty-first, and shuts down on November thirtieth. And numbers like twenty to fifty tasks, or about a hundred labels per failure mode, are practitioner heuristics.

MAYA: Let me try to sum up, and you correct me. Vibe checks don't scale. I start with error analysis: read traces, write notes, group them into failure modes, and count. Then I write down what success means, and build a small golden set from real failures.

BEN: Good so far.

MAYA: For grading, code checks first, an {LLM|L L M} judge for nuance, with binary questions, and I calibrate it against a human, looking at true positive and true negative rates, not just agreement. I keep a capability suite and a regression suite. And for agents, I grade the end state, over several trials, and track cost.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. You're starting evals for a new feature. What's your first step, and what should you not do yet?

PAUSE: 5

MAYA: Error analysis. Read real traces, write a short note on each failure, then group and count them. And I shouldn't pick metrics or write a rubric before I've read examples.

BEN: Perfect. Question two. Your judge agrees with the human on ninety percent of outputs. Why isn't that enough?

PAUSE: 5

MAYA: Because failures are rare, so agreement can look high while the judge misses most of the real failures. I need to report the true positive and true negative rates separately.

BEN: Exactly. And last one. Your agent succeeds on seventy-five percent of trials. What's the honest number for a customer-facing agent?

PAUSE: 5

MAYA: {pass^3|Pass to the power of three}, the chance that all three trials succeed. That's about forty-two percent.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm doing the first one: open-code thirty traces from any AI feature, one sentence each on what's wrong, then group and count.

BEN: Then build a twenty-case golden set with two code assertions, write one binary judge and compute its true positive and true negative rates, do a regression run with a one-word prompt change, and compute {pass^3|pass to the power of three} for an agent task. Next time, in chapter nineteen: open-weight and local models. What "open" really means, and how to test a model on your own task.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter nineteen.
