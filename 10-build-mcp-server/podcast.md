<!-- Podcast script, rendered by scripts/podcast.mjs. MAYA is the learner, BEN the expert.
     A turn starts with "NAME:". "PAUSE: <seconds>" inserts silence. Write everything as it should be spoken;
     {written|spoken} shows the first form in the transcript and speaks the second. -->

# AI Fundamentals, chapter 10: How MCP servers work

MAYA: [cheerful] Welcome back to AI Fundamentals, the podcast version. I'm Maya, and I'm still the one learning here.

BEN: And I'm Ben. Today is chapter ten: how {MCP|M C P} servers work. Tools as described functions.

MAYA: Last time, in chapter nine, we saw that {MCP|M C P} is an agreement, not a new model ability. Hosts, clients and servers, and servers offer tools, resources and prompts.

BEN: Right. And today we open up a server and look inside. Here's the one sentence. An {MCP|M C P} server is a small program that exposes ordinary functions as tools, each with a name, a description and a typed input schema. That way a model can decide when to call it, supply the right arguments, and use what comes back.

MAYA: [curious] Ordinary functions? So there's no magic in there?

BEN: No magic. A tool is a function. What {MCP|M C P} adds is a standard way for a client to find out which functions exist, and how to call them.

MAYA: Okay. So what actually happens when I ask a question and a tool gets used?

BEN: The quickstart describes five steps. One: the client sends your question to the model. Two: the model looks at the available tools and decides which to use. Three: the client runs the chosen tool through the {MCP|M C P} server. Four: the result goes back to the model. And five: the model writes a natural-language answer.

MAYA: And how does the model know which tools are available?

BEN: On the wire, step two depends on a request called {tools/list|tools slash list}. It returns each tool's definition. Step three is a {tools/call|tools slash call} request, with a tool name and arguments. The server's job is to wrap your function so it answers those two requests.

MAYA: So the model decides, but the client does the calling.

BEN: Exactly. The specification calls tools model-controlled: the model picks them. But it also says there should always be a human able to deny a tool call.

MAYA: Okay. If I want to build one, where do I start?

BEN: Pick an SDK. As of October 2026, the {MCP|M C P} docs list official SDKs in three tiers. Tier one is TypeScript, Python, {C#|C sharp}, Go, Rust and Ruby. Tier two is Java. And tier three is Swift, PHP and Kotlin. The chapter uses Python.

MAYA: And the setup?

BEN: The quickstart uses {uv|U V}. You create a project called currency, set up a virtual environment, and add the {mcp|M C P} package with its CLI extra, which gives you an {mcp|M C P} command. A plain pip install works too.

MAYA: Anything to watch out for?

BEN: Yes, a version note. Python SDK version two renamed the main high-level class, {FastMCP|Fast M C P}, to {MCPServer|M C P server}. You now import it from {mcp.server|M C P dot server}. And the old import path, quote, "is gone rather than deprecated."

MAYA: So old tutorials just break.

BEN: Most tutorials written before mid-2026 use the old name. So yes, expect that.

MAYA: Okay, let's write a server. What's the example?

BEN: A dollars-to-pounds converter with a fixed rate. It's tiny, but it shows every part. You create an {MCPServer|M C P server} called currency. You set a fixed rate of zero point seven nine, which is for illustration only. Then you write one function, {convert_usd_to_gbp|convert U S D to G B P}. It takes an amount, as a float, and returns a float: the amount times the rate, rounded to two decimals.

MAYA: And how does that become a tool?

BEN: You put a decorator on it that marks it as a tool. And you give it a docstring. "Convert an amount in US dollars to British pounds." Then a note that it uses a fixed illustrative rate, not live market data. And an explanation of the argument: amount, in US dollars. Finally, the script runs the server over the {stdio|standard I O} transport.

MAYA: Where's the part that describes the tool to the model? I didn't hear any schema.

BEN: That's the nice part. The quickstart explains that the class "uses Python type hints and docstrings to automatically generate tool definitions." So when a client sends {tools/list|tools slash list}, it gets back roughly this. The name, {convert_usd_to_gbp|convert U S D to G B P}. The description, from the docstring. And an input schema saying it's an object with one property, amount, which is a number, and amount is required.

MAYA: [thoughtful] Oh, so the function name becomes the name, the docstring becomes the description, and the type hints become the schema.

BEN: Exactly that mapping. And there's an optional fourth part. The return type can become an output schema, which describes the shape of structured results.

MAYA: Let me check what the model gets from each. The name is a handle that should say what it does. The description says what it does and when to use it. The schema says which arguments to give, and their types.

BEN: [warm] Perfect.

MAYA: Are there rules for names?

BEN: The specification has some. Between one and a hundred and twenty-eight characters. Case-sensitive. Only letters, digits, underscores, hyphens and dots. And unique within a server. It also warns that two servers can both expose a tool called search. Clients that combine servers should tell them apart, for example by prefixing the server name.

MAYA: Okay. You said this runs over {stdio|standard I O}. What does that mean in practice?

BEN: A local server is a command the client launches. It exchanges {JSON-RPC|JSON R P C} messages over standard input and standard output. And that leads to the classic trap. The quickstart is blunt. Quote: "Never write to {stdout|standard out}. Writing to {stdout|standard out} will corrupt the {JSON-RPC|JSON R P C} messages and break your server."

MAYA: [laughs] So no debug prints.

BEN: [laughs] No debug prints. Use the logging module, which writes to standard error. A stray {print()|print} is the most common reason a first server "connects but does nothing".

MAYA: I would absolutely have done that. Okay, step three. How do I test it?

BEN: Before you connect a real client, use the {MCP|M C P} Inspector. It's the reference tool for testing servers. It runs through {npx|N P X}, and as of October 2026 it needs Node twenty-two point nineteen or newer. You give it the command that starts your server, with an absolute path to the project folder.

MAYA: And then?

BEN: A web UI lets you list the tools, call one with arguments, and see the raw {JSON-RPC|JSON R P C} exchange. There's also a CLI mode that makes it scriptable, for example to run {tools/list|tools slash list}. And the Python SDK has a shortcut: {uv run mcp dev currency.py|U V run M C P dev currency dot P Y} opens the Inspector for you.

MAYA: Step four, connecting it to a real app.

BEN: For a local server, you tell the client which command starts it. In Claude Desktop, that goes in {claude_desktop_config.json|claude desktop config dot JSON}, under an {mcpServers|M C P servers} key. You add an entry called currency, with the command {uv|U V}, and arguments that point at the project folder and run the script.

MAYA: Any gotchas?

BEN: The quickstart stresses absolute paths. The command may need the full path to {uv|U V}. And restart the app after editing. Other clients store the same information elsewhere, which we compared in chapter nine.

MAYA: And then I just ask it?

BEN: Ask "convert twenty dollars into pounds", and watch the model choose the tool, pass amount equals twenty, and use the result.

MAYA: Okay, that sounds almost too easy.

BEN: [laughs] Getting a server running is the easy part. The real work is designing tools the model can use well. Anthropic's engineering post, "Writing effective tools for agents", treats tools as "a new kind of software which reflects a contract between deterministic systems and non-deterministic agents."

MAYA: So what does it recommend?

BEN: First: describe tools for a newcomer. Quote: "Think of how you would describe your tool to a new hire on your team." Spell out formats, terms and relationships the model can't infer. And: "Even small refinements to tool descriptions can yield dramatic improvements."

MAYA: [curious] Really? Small wording changes make a dramatic difference?

BEN: That's what the post says. The description is the model's main guide for choosing a tool. So it's a prompt, really, and you should test changes to it. Second: name parameters unambiguously. Quote: "Instead of a parameter named user, try a parameter named {user_id|user I D}."

MAYA: Because "user" on its own is ambiguous.

BEN: Right. Third: build fewer, more capable tools. Don't wrap every API endpoint. A {schedule_event|schedule event} tool that finds availability and books a meeting beats three separate tools for listing users, listing events and creating an event. And a tool that searches contacts beats one that lists them all. Quote: "More tools don't always lead to better outcomes."

MAYA: [thoughtful] Hmm. That goes against my instinct. I'd have mapped every endpoint one to one.

BEN: That's the instinct the post argues against. Fourth: namespace related tools. For example {asana_search|asana search} and {jira_search|jira search}, so the boundaries are clear when many servers are connected.

MAYA: Like the search clash from the name rules.

BEN: Exactly. Fifth: return meaningful context. Prefer readable names over opaque IDs. And consider a {response_format|response format} parameter, so the model can ask for concise or detailed output.

MAYA: Sixth?

BEN: Be token-efficient. Paginate, filter and truncate, with sensible defaults. At the time of the post, in September 2025, Claude Code limited tool responses to twenty-five thousand tokens by default.

MAYA: And the last one?

BEN: Evaluate, don't guess. Build realistic multi-step tasks, run them through an agent, read the transcripts, and refine the tools. And keep some tasks held out, to avoid overfitting. Chapter eighteen is all about evals.

MAYA: Let me summarise. Describe tools like for a new hire, clear parameter names, fewer and bigger tools, namespaces, useful output, not too many tokens, and test with real tasks.

BEN: That's all seven. Now, errors. The specification separates two kinds. Protocol errors come from an unknown tool, a malformed request, or a server fault. They're returned as a {JSON-RPC|JSON R P C} error, and they may be shown to the model.

MAYA: And the other kind?

BEN: Tool execution errors. An API failure, bad input, or a business-rule violation. Those come back as a normal result, marked with {isError: true|is error, true}. And they should be shown to the model, quote, "to enable self-correction".

MAYA: Why does the difference matter?

BEN: Because a good error message lets the model fix its own call. The specification's example: "Invalid departure date: must be in the future. Current date is {08/08/2025|August eighth, 2025}." With that, the model can retry with a fixed argument.

MAYA: Instead of just "something went wrong".

BEN: Right. And here's a detail. In Python SDK version two, only the message of a raised {ToolError|tool error} reaches the model. Any other exception becomes a generic "Error executing tool" message, with the details in the server log. So write your error messages for the model.

MAYA: What about state? Say I'm building a shopping basket.

BEN: Remember from chapter nine, the July 2026 spec has no protocol-level session. So a server that needs state across calls, like a shopping basket or a browser session, should return an explicit handle from one tool, and accept it as an argument on the next. And it should say in the description how long that handle lives.

MAYA: So the state is passed around in the open, not hidden.

BEN: Exactly. And security. Servers must validate all inputs, implement access controls, rate-limit calls, and sanitise outputs. Clients should confirm sensitive operations, and show tool inputs to the user before calling, to avoid data being exfiltrated. A tool runs with whatever access you give it. Chapter thirteen goes into that.

MAYA: Okay, last section. Can't I just have an agent write the server for me?

BEN: You can. You don't need to memorise the SDK syntax. But the agent needs current, focused documentation. Because SDK APIs move. An agent trained on 2025 code will happily write the old import of {FastMCP|Fast M C P}, which fails on Python SDK version two.

MAYA: [laughs] The tutorial problem again, just inside the model.

BEN: [laughs] Right. So, a few options. Point the agent at the specific pages it needs: the build-server quickstart, the SDK docs, and the tools section of the spec. Each {MCP|M C P} docs page points to an {llms.txt|L L M S dot text} index, quote, "to discover all available pages", which an agent can use to find the right one.

MAYA: Any other options?

BEN: The {MCP|M C P} docs describe a set of agent skills, the {mcp-server-dev|M C P server dev} plugin, that guide a coding assistant through server design. They ask what the server connects to, who will use it, how many operations it needs, and what auth applies. Then they recommend a deployment path. A remote server over Streamable {HTTP|H T T P}, a packaged local bundle, or local {stdio|standard I O} for prototyping.

MAYA: And the simplest option?

BEN: A common pattern is to give the agent the specific doc page or README, not a whole repository. The relevant material stays in focus, and the context stays small.

MAYA: Does this pattern work beyond a toy converter?

BEN: It covers most real tools. Pure logic, like the converter. A local library. Or a remote HTTP API, like the quickstart's weather server, which calls the US National Weather Service. They all become inputs in, result out, with a description. And a sensible habit when you wrap a library or an API: run its own example on its own first, then wrap one function as a tool.

MAYA: Okay. What lasts, and what will date?

BEN: The durable part. A tool is a function plus a name, a description and an input schema, and the model chooses by reading them. Descriptions and parameter names are prompts. Write them for a reader who knows nothing else, and test changes. Fewer, workflow-shaped tools beat thin wrappers around every endpoint. Return errors the model can act on, and keep {stdout|standard out} clean on {stdio|standard I O} servers. Test with an inspector before a real client. And treat every tool as code with real permissions.

MAYA: And what will date?

BEN: Python SDK version two's names, the {mcp dev|M C P dev} command, and the SDK tiers. The details of the July 2026 spec revision, like statelessness, handles and the tool-name rules. The Inspector's requirements and flags, and Claude Desktop's config paths. And the twenty-five-thousand-token default in Claude Code, from September 2025.

MAYA: Let me try to sum up, and you correct me. An {MCP|M C P} server wraps ordinary functions as tools. Each has a name, a description and an input schema, and in Python the SDK builds those from the function name, the docstring and the type hints. The model reads them and decides what to call.

BEN: Good so far.

MAYA: A local server runs over {stdio|standard I O}, so never print to {stdout|standard out}. Test it in the Inspector, then register the command in the client with absolute paths. Write descriptions for a new hire, build fewer and bigger tools, return errors the model can act on, and pass state as explicit handles. And if an agent writes the server, give it current docs.

BEN: [warm] That's it. You've got it.

MAYA: Okay, quiz time. You ask, I answer, and everyone listening answers too.

BEN: Question one. Your new server connects, but nothing works, and you added a print statement for debugging. What's going on?

PAUSE: 5

MAYA: The print writes to {stdout|standard out}, which corrupts the {JSON-RPC|JSON R P C} messages. Use the logging module instead, which writes to standard error.

BEN: Perfect. Question two. You rename your tool to {tool1|tool one}, with the description "does stuff". What happens, and why?

PAUSE: 5

MAYA: The model has much less to go on, because the name and description are its main guide for choosing a tool. So it may not pick the tool when it should.

BEN: Exactly. And last one. Someone asks to convert minus fifty dollars. How should your tool respond?

PAUSE: 5

MAYA: With a tool execution error, not a generic exception. Raise a {ToolError|tool error} that says what's wrong and what's allowed, so it comes back marked {isError: true|is error, true}, and the model can correct itself.

BEN: Three out of three.

MAYA: [cheerful] And for homework, the chapter has five exercises. I'm starting with the first: build the one-tool server, open it in the Inspector, call it with valid and invalid input, and read the raw {tools/list|tools slash list} output.

BEN: Then break the description on purpose, add a near-duplicate euro converter and see if the model picks correctly, make a good error for negative amounts, and ask an agent to add a tool with and without the docs. Next time, in chapter eleven, we look at Agent Skills. Packaging know-how as folders an agent loads in stages, and how skills differ from {MCP|M C P}.

MAYA: Thanks for listening to AI Fundamentals. See you in chapter eleven.
