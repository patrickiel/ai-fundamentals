<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 14: Dynamic context discovery

MAYA: [cheerful] Welcome to AI Fundamentals, the podcast version. I'm Maya, and I'm the one learning here.

BEN: And I'm Ben. Today is chapter fourteen: dynamic context discovery. Letting the agent pull context in, instead of pushing it all in.

MAYA: Last time, in chapter thirteen, we covered prompt injection, and why any text an agent reads can steer it. Today sounds a bit calmer.

BEN: [laughs] It is. We're back to the context window, which we left after chapters four and five. Here's the one sentence for today. Keep the always-loaded context small, by putting bulky material where the agent can find it. Tool output, chat history, skill instructions, tool definitions, terminal logs. Give the agent short names and pointers, and let it load the specific part it needs, when it needs it.

MAYA: Pointers instead of piles.

BEN: That's a nice way to put it. There are two kinds of context here. Static context is loaded every time. Dynamic context discovery gives the agent less up front, and lets it pull in what it needs.

MAYA: [curious] My instinct is the opposite, though. Give the agent everything it might need, so it never has to go looking.

BEN: That's the usual instinct. And remember from chapter four: every token in the window competes for the model's attention. Cursor's post from January 2026 argues the opposite of that instinct. As models got better at acting as agents, giving them fewer details up front worked better, because they can fetch what they need themselves.

MAYA: What do you actually gain?

BEN: Cursor gives two benefits. Only the data the agent actually needs enters the window, which saves tokens. And with less irrelevant material, there's less to confuse the model, or to contradict the relevant parts.

MAYA: Is that just Cursor's view?

BEN: Anthropic's context-engineering guide describes the same shift. Instead of retrieving everything before the model runs, agents keep, quote, "lightweight identifiers (file paths, stored queries, web links, etc.)". And they load the data with tools at runtime. Anthropic calls that just-in-time retrieval.

MAYA: So the agent knows where things are, but doesn't hold them.

BEN: Exactly. And folder structure, file names and timestamps tell the agent what something is before it opens it. Each step informs the next. Anthropic calls this progressive disclosure.

MAYA: Same term as in chapter eleven, with skills.

BEN: Same idea. And the guide is honest about the trade-off. Runtime exploration is slower than reading pre-loaded data. And it needs well-designed tools to avoid wasted steps. So a hybrid often works best. Claude Code loads {CLAUDE.md|CLAUDE dot M D} up front, and uses glob and grep to find everything else.

MAYA: Where does the agent find all this bulky material?

BEN: Cursor chose files as the common interface. It describes them as a simple primitive, safer than a new abstraction that might not hold up as agents improve. But it also says, quote, it's "not clear if files will be the final interface for LLM-based tools".

MAYA: [thoughtful] So the whole idea rests on one trade. Fewer tokens and less confusion, in exchange for a few extra steps.

BEN: That's the trade. And it only works if the names and descriptions are good enough for the agent to find what it needs.

MAYA: Okay. The chapter goes through five places where this applies, right?

BEN: Five, from Cursor's post. The first is long tool output. Shell commands and MCP tools can return huge outputs. Keeping them inline fills the window.

MAYA: Can't you just cut them off? Keep the first part?

BEN: That's the common alternative, truncation. And it can cut exactly the part that mattered.

MAYA: [laughs] Like the error message at the very bottom of a build log.

BEN: [laughs] That's the classic case. So instead, Cursor writes long shell and MCP output to a file. The agent checks the end with tail, and reads more if it needs to. Cursor reports this led to fewer unnecessary summarisations when the context filled up.

MAYA: Does Claude Code do that too?

BEN: As of October 2026, yes. For Bash, output up to roughly thirty thousand characters arrives inline. Beyond that, Claude gets the path of a saved file plus a preview of up to two thousand characters. Then it reads or searches the file for the rest. For MCP, Claude Code warns when a tool's output exceeds ten thousand tokens. And successful text results longer than fifty thousand characters are saved to a file, and replaced in the conversation with the file path.

MAYA: So the conversation holds a pointer to the output, not the output itself.

BEN: Right. And there's a related idea: filter before the model sees anything. Anthropic's code-execution approach lets the agent process a large dataset in code, and see, for example, five rows instead of ten thousand. And Claude Code's docs suggest a hook that greps a log for the word ERROR, and returns only the matching lines. In their words, "reducing context from tens of thousands of tokens to hundreds".

MAYA: Okay. Second place?

BEN: Chat history. When the window fills, agents summarise and continue. And as we know, summaries lose detail. So after summarisation, Cursor gives the agent a reference to the full chat history file. The agent can search it for a detail the summary dropped.

MAYA: Oh, that fixes the "warning that didn't make it into the summary" problem from chapter four.

BEN: That's the kind of gap it covers. Claude Code takes a similar approach when you summarise part of a session from the {/rewind|slash rewind} menu. Summarising, quote, "doesn't change files on disk, and the original messages stay in the session transcript, so Claude can still reference the details".

MAYA: So the summary isn't the only copy.

BEN: Right. Whatever the tool, the principle is the same. Treat the summary as an index, and keep the raw record as a backstop.

MAYA: Third is skills, which we did in chapter eleven.

BEN: So I'll keep it short. Agent Skills package instructions, scripts and reference material for a type of task, in a folder with a {SKILL.md|SKILL dot M D} file. Anthropic released the format as an open standard in December 2025. And the standard's site now lists dozens of adopting tools, including Claude Code, Cursor, OpenAI Codex, {Gemini CLI|Gemini C L I}, GitHub Copilot and VS Code.

MAYA: And they load in stages.

BEN: Three stages. Discovery: at startup, only each skill's name and description go into the system prompt. Quote, "just enough information for Claude to know when each skill should be used". Activation: when a task matches, the agent reads the full {SKILL.md|SKILL dot M D}. And execution: bundled files are opened, or scripts run, only as needed. A script can run, quote, "without loading either the script or the PDF into context."

MAYA: Which is why a skill can be huge without costing much.

BEN: Anthropic says "the amount of context that can be bundled into a skill is effectively unbounded". Cursor finds the right skill files with grep and its own semantic search. And in Claude Code, a skill marked {disable-model-invocation: true|disable model invocation, true} isn't even listed at startup. It costs no context until you run it yourself, with a slash and its name. Chapter twelve covers the supply-chain risks of skills.

MAYA: Okay, number four. MCP tools. I have a feeling this is the big one.

BEN: [laughs] It has the most numbers. Here's the problem. Most MCP clients load every connected tool's name, description and schema into context at the start, whether or not it's used. Anthropic's example: five servers with fifty-eight tools took about fifty-five thousand tokens before any work began. And internally, it has seen tool definitions reach a hundred and thirty-four thousand tokens.

MAYA: [sighs] Before the agent has done anything.

BEN: Before anything. And large tool sets also make the model more likely to pick the wrong tool. Chapter four called that context confusion.

MAYA: So what's the fix?

BEN: The chapter compares three implementations of the same idea. First, Cursor's tool folders. Tool descriptions are synced to one folder per server. Only the tool names are static context, and the agent looks up the details when it needs them. In an {A/B test|A B test}, that cut total agent tokens by forty-six point nine percent, in runs that called an MCP tool.

MAYA: Almost half. Is that reliable?

BEN: Cursor says it was statistically significant. But with high variance, depending on the number of servers. Second, Anthropic's Tool Search Tool, on the API. Tools marked with {defer_loading|defer loading} set to true stay out of context. Claude searches for tools, using regex, {BM25|B M twenty-five} or a custom search, and the matching definitions are loaded.

MAYA: And the result?

BEN: About eighty-five percent fewer tokens in their example. And, interestingly, better accuracy on their MCP evaluations. {Opus 4|Opus four} went from forty-nine to seventy-four percent. {Opus 4.5|Opus four point five} went from seventy-nine point five to eighty-eight point one.

MAYA: [curious] Wait, fewer tools in view, and it chooses better?

BEN: That's the point about confusion. Less in the window, better focus. The third approach is code execution with MCP. The MCP servers are presented as a file tree, with a folder per server and a TypeScript file per tool. The agent explores the tree, or calls a {search_tools|search tools} function with a detail level. Then it writes code that calls the tools. In one workflow, that went from a hundred and fifty thousand tokens to two thousand. That's ninety-eight point seven percent less.

MAYA: That's a huge drop. Where does it come from?

BEN: The agent only opens the tool files it actually needs. And because it works in code, it can filter data before anything reaches the model, like the five rows instead of ten thousand we mentioned earlier.

MAYA: Why did Cursor pick folders, rather than a search?

BEN: It considered a search-based approach. But it chose folders so each server's tools stay grouped, rather than scattered across a flat index. And there's a side benefit. Because the tool list lives on disk, the agent can tell the user when a server needs re-authentication, instead of silently losing its tools.

MAYA: So should I always turn on tool search?

BEN: Not always. Anthropic suggests it pays off when definitions exceed about ten thousand tokens, when tool selection is a problem, or with ten or more tools. It helps less with a few small tools that are used every session. And code execution adds its own costs. Running agent-written code needs a secure sandbox, with resource limits and monitoring.

MAYA: And what does Claude Code do by default?

BEN: Claude Code uses tool search by default. Only tool names and server instructions load at session start. So, quote, "adding more MCP servers has minimal impact on your context window".

MAYA: [curious] But if the full definitions aren't loaded, how does Claude know which tool to use?

BEN: The names and server instructions are there from the start. When a task needs a tool, Claude searches by description and loads the matching definitions.

MAYA: Can I change that default?

BEN: With a setting called {ENABLE_TOOL_SEARCH|enable tool search}. Set to auto, it loads tools up front only while their definitions fit within ten percent of the window. Set to false, it loads everything. And a server marked {alwaysLoad|always load} is exempt.

MAYA: Anything for people who write MCP servers?

BEN: Yes. Because Claude finds tools by searching their descriptions, MCP server authors should write clear server instructions, and put key terms early in tool descriptions. And the docs note that command-line tools, like {gh|G H} or {aws|A W S}, are still more context-efficient than MCP servers, since they add no per-tool listing.

MAYA: Huh. So sometimes the plain old command line wins.

BEN: For context cost, it does. The fifth place is terminal sessions. Getting terminal output to an agent used to mean pasting it into the chat. Or having the agent's own commands inject their full output into context.

MAYA: [laughs] I paste terminal output constantly.

BEN: Most people do. Cursor now syncs its integrated terminal sessions to the local filesystem. So the agent can grep for the relevant lines. For example, in the logs of a long-running dev server, to answer "why did my command fail?". And in Claude Code, a backgrounded command's output goes to a file, which Claude reads with its file tools.

MAYA: Can I do that in any agent?

BEN: A common pattern works everywhere. Run long-lived processes with their output captured to a log file, and let the agent search it rather than paste it. The chapter's example runs the dev server with both standard output and errors piped through tee into a log file with a timestamp in its name. Later, you grep those logs for "error", ignoring case, and take only the last twenty matches.

MAYA: Let me try to pull the five together. Tool output: save it to a file, preview it, then read or search, instead of keeping it inline or truncating it. Chat history: keep the transcript searchable, instead of relying only on the summary.

BEN: Good. Keep going.

MAYA: Skills: name and description up front, the body only when used. MCP tool definitions: names up front, then search or browse, then load. And terminal output: log to files, then grep and tail, instead of pasting full logs.

BEN: [warm] That's exactly the chapter's summary table. The window holds pointers and the current slice. Everything else stays on disk, or behind a search.

MAYA: What part of this will still be true in a few years, and what will date?

BEN: The durable part first. Less in the window means better focus. Irrelevant context costs quality as well as tokens. Save in full and search, rather than truncate or keep everything inline. Use progressive disclosure: short names and descriptions first, details on demand. Keep raw records, like transcripts and logs, as a backstop for summaries. And discovery trades latency and extra steps for a smaller context. It only works with good names and descriptions.

MAYA: And what will date?

BEN: Cursor's specific features and the forty-six point nine percent figure, from January 2026. The Tool Search Tool, {defer_loading|defer loading}, and their benchmark numbers, from November 2025, when it was in beta. Claude Code's defaults: tool search being on, the values of {ENABLE_TOOL_SEARCH|enable tool search}, and the thirty-thousand-character Bash and fifty-thousand-character MCP thresholds. And which products support Agent Skills. Those last two are as of October 2026.

MAYA: Let me sum up the whole thing, and you correct me. Static context loads every time. Dynamic context discovery keeps it small, and lets the agent pull in what it needs, through pointers like file paths and short descriptions. That saves tokens, and it can improve answers, because there's less to confuse the model.

BEN: Good so far.

MAYA: Most of it works through files. Long output, chat history, skills, tool definitions and terminal logs go somewhere the agent can search. And the cost is extra steps and some slowness, so a hybrid often works best. Like {CLAUDE.md|CLAUDE dot M D} up front, and everything else found with glob and grep.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. A build produces a huge log. Why not just truncate it to fit, and what's the better approach?

PAUSE: 5

MAYA: Because truncation can cut exactly the part that mattered, like the error at the end. Better to save the full output to a file, give the agent the path and a preview, and let it read the end with tail, or grep for what it needs.

BEN: Perfect. Question two. You're building on the API with a handful of MCP servers. When does Anthropic say tool search pays off, and when does it help less?

PAUSE: 5

MAYA: It pays off when the tool definitions are over about ten thousand tokens, when the model picks the wrong tools, or with ten or more tools. It helps less with a few small tools that get used every session.

BEN: Exactly. And last one. What's the cost of dynamic context discovery, and what does it depend on to work?

PAUSE: 5

MAYA: It's slower, because the agent takes extra steps to find things, instead of having them loaded. And it only works with good names and descriptions, because that's what the agent uses to find the right thing.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the log one. Run my dev server with its output captured to a log, cause an error, and ask the agent to diagnose it from the log alone, without re-running anything.

BEN: And try the others too. Run {/context|slash context} before your first prompt and see what loads up front. Start Claude Code once with tool search switched off and once with the default, and compare. Trigger a file offload with a very long command output. And write a skill with a sharp description, then check that its body only loads when it's used. Next time, in chapter fifteen, we look at the A2A protocol. How agents discover and delegate to other agents, and how A2A complements MCP.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter fifteen.
