# The Forward Deployed Engineer: Fitting AI Into How Work Really Gets Done

> **Core idea in one sentence:** Getting value from AI inside a business depends less on the model than on understanding the real process: watch the work, decide which steps belong to software, a model or a person, design for failure, test on known answers, and measure the value; the forward deployed engineer is the role built around doing that on site.

## TL;DR

- A **forward deployed engineer (FDE)** embeds with a customer and builds working software inside its systems. Palantir, which popularised the role, describes it as "one customer, many capabilities", as opposed to a product engineer's "one capability, many customers" [1].
- OpenAI, Anthropic and Cursor now run FDE teams for AI deployments [2][3][4]. They describe the job as engineering, not consulting.
- The case for the role is a **deployment gap**: an MIT NANDA study reported most organisations seeing no measurable return from generative AI, and attributed this to approach rather than model quality [7].
- Recurring practices: **build on existing systems**, **don't change workflows more than necessary**, and **budget more time for trust than for the build** [2][5].
- A five-step method: **audit the real process → assign each step to software, a model or a person → design for failure → test on known-answer cases → put a number on the value** [4][5][6].

## Where the role comes from

Palantir has long split its engineers into "Devs", who build its platforms, and "Deltas", officially Forward Deployed Software Engineers, who deploy those platforms for specific customers [1]. A Delta's mandate is "to achieve technical outcomes for our customers", measured "in terms of impact on the customer's goal". Palantir's example is a manufacturer trying to reduce defective products coming off an assembly line [1].

Palantir is explicit that this is not consulting. One engineer puts the difference this way: consultants "generally create a one-time analysis, recommendation, or solution", while Deltas build "a long-term solution" with the customer [1]. Deltas also feed fixes and features back into the core product, in coordination with the product teams [1]. A recurring judgement call is whether to build a capability into the platform or "create something custom that we can deliver faster" [1].

## The role at AI companies

The same model now appears at model developers and AI tool makers:

- **Anthropic** places FDEs in its Applied AI team. They embed with strategic customers, "work within customer systems to build production applications with Claude models", deliver artefacts such as MCP servers, sub-agents and agent skills, and feed repeatable deployment patterns back to product and engineering. The posting expects 25–50% travel to customer sites [3].
- **OpenAI**'s head of forward deployed engineering, Colin Jarvis, told the Altimeter Capital podcast that the only approach that worked consistently was to "embed directly with clients, learn their workflows, and work alongside their staff" (as reported by Business Insider) [2].
- **Cursor**'s VP of forward deployed engineering, Pauline Brunet, describes FDE as fitting best where a customer is less digitally mature *and* the product is highly customisable. Mature customers with simple products need documentation and self-service instead. She warns against FDE teams drifting into solution architecture, bug logging or product training, and frames engagements as roughly six weeks with a defined outcome that the customer owns, while the FDE stays hands-on in the build [4].

## Why the role exists: the deployment gap

MIT NANDA's July 2025 report *The GenAI Divide* found that "95% of organizations are getting zero return" from generative AI in its research, while "just 5% of integrated AI pilots are extracting millions in value" [7]. It drew on a review of over 300 public AI initiatives, interviews with 52 organisations and a survey of 153 senior leaders [7]. The authors concluded that the divide "does not seem to be driven by model quality or regulation, but seems to be determined by approach", and that most enterprise tools fail because of "brittle workflows, lack of contextual learning, and misalignment with day-to-day operations" [7]. They also found that partnerships with external builders reached deployment about twice as often as internal builds (~67% vs. ~33%), while noting these outcomes were self-reported [7].

Treat the headline figure with care. The report calls itself "preliminary findings", and 95% describes organisations in its sample, not every AI project everywhere.

Practitioners describe the same failure from the inside. Vasuman Moza, CEO of Varick Agents, says AI is often "slapped on top of broken processes" because nobody has mapped how the work is actually done [5]. Documentation usually covers the main path and perhaps an edge case or two, not what people do when things go wrong [5]. Those undocumented recovery paths are exactly what an agent needs to know, and they are why the role puts engineers next to the people doing the work.

## Practices that recur across teams

**Build on the systems the business already runs.** Moza describes a client who had spent five million dollars and five years migrating to NetSuite; a proposal that required leaving it would have been rejected. His team builds "the agents on top of your systems of record" [5]. Varick's guide puts it generally: "Build over the current data and systems rather than beginning with a large replacement project" [6].

**Don't change the workflow more than you must.** Moza's example: if operators are used to an eleven-step workflow and it suddenly becomes one step, "adoption rates might suffer" [5]. The new system should be close enough to the old one that people still understand how to operate and check it.

**Budget more time for trust than for the build.** At Morgan Stanley, according to Jarvis, the technical work took six to eight weeks; the team then spent another four months "running pilots, collecting evaluations, and iterating with wealth advisors", after which "about 98%" of advisors adopted the tool [2].

**Pick work where improvement adds up.** A common heuristic is to start with tasks that happen often and cost real time, since automating a rare task saves little. Brunet's advice to define a concrete outcome before the engagement starts serves the same purpose [4].

## A five-step method

### Step 1: Audit the real process

Varick describes starting each engagement with an audit [5][6]. Its guide describes collecting context from interviews, emails, spreadsheets, SOPs and approvals, then mapping tools, people and **exceptions**, and producing an "operating map": current and future workflow, the selected use case, its boundaries, and the expected value [6]. In practice: watch the job being done, write every step in the order it actually happens, and ask why each step exists. A step nobody can justify is often a workaround that matters more than it looks.

### Step 2: Assign each step to software, a model or a person

Varick's guide uses three filters [6]:

| Who does the step | When |
|---|---|
| **Deterministic software** | Rules and inputs are predictable |
| **An agent (model)** | The objective is clear but inputs, paths or actions vary |
| **A human in control** | There is material ambiguity, accountability, or irreversible consequences |

The result is rarely "AI does everything". In Moza's eight-step example, four steps ran autonomously, three ran with a human in the loop, and one stayed fully human, either because the risk was too high or because automating it would not produce measurable value [5]. This mirrors the responsibility model in topic 28.

### Step 3: Design for failure

A model returns an answer whether or not it is sure, so design for the cases where it shouldn't act alone. Varick's guide lists failure modes such as missing data, malformed responses, dead APIs, timeouts, duplicates and partial completion, and recommends checkpointing, retries with backoff, schema validation of outputs, confidence thresholds that trigger escalation, and routing high-risk actions to a person [6]. It also advises starting "with the smallest useful action" and granting "more authority only after the system proves reliable" [6].

### Step 4: Test on cases with known answers

Collect real cases from the audited process where the correct outcome is known. Include normal, edge, incomplete, ambiguous and high-risk inputs [6]. Run the system on all of them, count the correct results, then read every miss and group the failures [6]. Because outputs vary between runs, run each case more than once. This is an evaluation set (topic 18), and it turns "seems to work" into a number you can track through pilots.

### Step 5: Put a number on the value

Brunet sorts business value into three kinds: **increase revenue, decrease cost, or mitigate risk** [4]. An agent's running cost only means something next to the outcome it changes. Her example: a customer complained that an agent was "costing $2,000 per day". The agent helped send the right person to repair equipment. Framed as reduced dispatch costs, the customer agreed it could be worth it, but had "never measured it this way" [4]. Record a baseline during the audit so this comparison is possible later.

## What the role demands

- **Engineering depth.** Palantir lists "technical decomp", taking a business problem "down to the lines of code you need to solve it", alongside autonomy and the judgement to choose between platform work and faster custom builds [1]. Cursor currently hires FDEs with at least five years of software engineering and extensive customer-facing experience [4]. Anthropic asks for production experience with LLMs, agents and evaluation frameworks [3].
- **Business understanding and empathy.** Palantir mentions "high user empathy" and a desire to understand business strategy [1]. Anthropic asks for the ability "to conduct discovery with customers" [3].
- **Comfort with ambiguity.** Both Anthropic and Palantir stress working autonomously in unclear situations [1][3].

Coding agents lower the cost of building a first version, which shifts more of the scarce work towards discovery, process mapping and evaluation. The companies hiring for the role still expect strong engineers.

## How to start

Pick the smallest real process you can observe, at your workplace or a friend's business. Sit next to the person who does the most repetitive task for an hour while they do it, then run the five steps on what you saw. The output is an audit document: the process as actually performed, the proposed split between software, model and person, the evaluation cases you would use, and the value in money, time or risk. It is concrete evidence of the skill, whether you apply for an FDE role or propose the work to a business.

## What lasts vs. what will date

- **Durable principles:**
  - The documented process is not the real process. Observe the work, especially the exceptions.
  - Assign each step by predictability, judgement and cost of error.
  - Integrate with existing systems and keep workflows recognisable; adoption is the constraint.
  - Expand an agent's authority only as evidence of reliability accumulates.
  - Value is revenue gained, cost removed or risk reduced, compared with the current way.
- **Time-sensitive details (as of 2026-10):** team sizes and hiring requirements at specific companies; job-posting details such as travel share; the MIT NANDA figures, which come from a preliminary 2025 study; specific integration mechanisms such as MCP servers and agent skills.

## Try it yourself

1. **Shadow a process for one hour.** Write each step in order. Mark every step nobody could justify.
2. **Run the three filters.** Label each step *software*, *agent*, *agent + human check* or *human*, with one sentence of reasoning.
3. **Build a known-answer set.** Collect 20–50 past cases with known outcomes, covering normal, edge and high-risk inputs. Run a prototype twice on each, count correct results and read every miss.
4. **Price the alternative.** For one step, estimate the agent's running cost and the current cost, including errors. Classify the value as revenue, cost or risk.
5. **Write a one-page audit** combining the above, plus how you would pilot it and earn users' trust.

## Key terms

- **Forward deployed engineer (FDE):** an engineer embedded with a customer who builds and deploys working software inside the customer's real systems and processes.
- **Delta / Dev:** Palantir's internal names for forward deployed and product software engineers.
- **Audit:** a written map of how a process actually runs, what to automate, and what it is worth.
- **System of record:** the authoritative business system (ERP, CRM, finance) that agents should build on rather than replace.
- **Human-in-the-loop:** a design where a person reviews or approves model output before it takes effect.
- **Evaluation set:** real cases with known correct outcomes, used to measure how often the system is right.
- **Deployment gap:** the difference between trying AI tools and getting measurable value from them in production.

## Sources

1. [Dev versus Delta: Demystifying engineering roles at Palantir](https://blog.palantir.com/dev-versus-delta-demystifying-engineering-roles-at-palantir-ad44c2a6e87) (Palantir Blog, 2019-04-08). Palantir's own definition of the Forward Deployed Software Engineer, how it differs from consulting, and the skills it needs.
2. [An OpenAI exec explains how his growing team helps companies move from AI hype to adoption](https://www.aol.com/articles/openai-exec-explains-growing-team-080035434.html) (Lee Chong Ming, Business Insider, syndicated on AOL, 2025-11-28). Colin Jarvis on OpenAI's FDE approach and the Morgan Stanley timeline and adoption figure, from the Altimeter Capital podcast.
3. [Forward Deployed Engineer (London)](https://job-boards.greenhouse.io/anthropic/jobs/5423029008) (Anthropic Applied AI team job description, read 2026-10). How Anthropic defines the FDE's responsibilities, deliverables and required skills.
4. [Forward Deployed Engineering at Cursor](https://ai.engineer/talks/forward-deployed-engineering-at-cursor-pauline-brunet) (Pauline Brunet, Cursor, AI Engineer World's Fair, 2026). When FDE is justified, engagement scoping, the revenue/cost/risk value framing, and the $2,000-a-day example.
5. [AI tools for Forward Deployed Engineering](https://ai.engineer/talks/l0FLhNqBOic-ai-tools-forward-deployed-engineering) (Vasuman Moza & JD Pruitt, Varick Agents, AI Engineer World's Fair, 2026). Broken processes, undocumented exceptions, building on systems of record, workflow change and adoption, the eight-step split, and audit-first engagements.
6. [Varick's Guide to AI Forward Deployed Engineering](https://learn.varickagents.com/fde-in-30-days) (Varick Agents, undated, read 2026-10). The audit/evals/deployment structure, the three filters, failure handling and value tracking.
7. [The GenAI Divide: State of AI in Business 2025](https://mlq.ai/media/quarterly_decks/v0.1_State_of_AI_in_Business_2025_Report.pdf) (Challapally, Pease, Raskar & Chari, MIT NANDA, 2025-07). The 95% figure, its methodology and caveats, failure causes, and external vs. internal build outcomes.
