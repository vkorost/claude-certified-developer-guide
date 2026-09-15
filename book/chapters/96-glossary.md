# Glossary

**Summary:** *Two kinds of entry, and they fail differently. The names this book gives to recurring shapes exist so you can recognise a pattern in a scenario written about an industry nobody here has mentioned. The platform terms exist because their definitions are exact, items are built on the difference between two of them, and being approximately right about one is the same as being wrong. Chapter references point at where each is argued rather than merely used.*

## Names This Book Gives to Recurring Shapes

The **accreting prompt** (ch. 5). A prompt that has grown by addition for months, where each paragraph was a reasonable response to one bad output and nothing was ever removed. It fails on length: verbose input pulls verbose output, which buys latency and no accuracy.

The **boundary clause** (ch. 9). The sentence in a tool description naming the requests this tool does not serve, and which tool serves them instead. Leave it out and Claude has no way to rule the tool out, so it calls the tool, reads what comes back, and calls the other one too.

The **carried cost** (ch. 8). What today's request pays for material you sent, and were already charged for, on an earlier one. It is the quantity caching attacks, and it grows with conversation length while the question you are asking stays the same size.

The **copy set** (ch. 22). Every place a credential's value has come to rest, together with the access each of those places carries. Three measurements describe the damage a leak does: how many copies exist, what each one reaches, and how long each stays valid.

The **crossing rule** (ch. 16). What makes an edge between components real rather than notional. Three things have to be written down: what may cross, which single component decides, and what happens to material that fails the decision. Written only into a prompt, it is a preference with good adherence.

The **dependency assumed frozen** (ch. 23). A version, a model, an endpoint or a schema treated as permanent because it has not moved yet. It produces one specific failure: a migration run as an incident, on a deadline somebody else set, in the week it turns urgent.

The **enumeration test** (ch. 12). Ask whether you can write down the steps in advance. If you can, a model choosing them at runtime adds cost and variance and buys nothing. It is a test for what the task leaves open, not an argument that simple designs are better.

The **execution question** (ch. 11). Does this capability have to reach a system holding live state and authenticating its callers, or is it knowledge about how work gets done? Asked first, it removes two of four candidate mechanisms before any comparison starts.

The **field of view** (ch. 19). What a single check can observe: one function, one call, one flow, one number. A failure is findable only by a check whose field of view contains it, which is why a suite of green tests and a broken product are perfectly consistent.

The **four-part call** (ch. 2). Every request to Claude, reduced to what it always contains: an address, headers that authenticate and pin a version, a body, and a response you interpret rather than read. Every failure worth debugging sits in a seam between two of those parts.

The **model-shaped dependency** (ch. 17). What sits on the far end of the call, described by the five properties that decide your engineering: remote, variable in what it returns, billed per call, versioned on somebody else's schedule, and able to fail while reporting success.

The **ownership ledger** (ch. 13). Two columns for every framework decision: what leaves your codebase, and what arrives as an inherited assumption. A choice argued from one column alone is not a choice.

The **placement sort** (ch. 5). Sorting the text in a prompt by two questions: how long it applies for, and who could have written it. Placement follows the answers, which makes instruction placement a trust decision before it is a style one.

The **point of refusal** (ch. 21). The moment in a sequence where something declines to proceed. A control without one shapes what Claude tries to do and never changes what the runtime allows, which makes it advice.

The **reuse boundary** (ch. 10). Where a capability stops belonging to one application and becomes a separately released artifact that several callers depend on. Crossing it converts an internal function into a versioned contract with an upgrade schedule.

The **shape question** and the **truth question** (ch. 6). Two questions a response has to answer separately. Shape asks whether it is addressable: fields present, types right, values inside the permitted set. Truth asks whether those fields match the world. Constrained generation settles the first and cannot touch the second.

The **spent context** (ch. 7). Material fetched for a step that has since finished, resent on every later request, drawing on the same budget as the question you actually want answered. It costs tokens and competes for attention, and both halves matter.

The **standing constraint** (ch. 15). A property of the environment that predates the request, appears in nobody's description of the work, and decides the design anyway. It announces itself late: a tenancy rule violated by a cache surfaces months later, once, when one tenant is served another tenant's decision.

The **standing grant** (ch. 14). Authority handed over once, in a file, then inherited by every session afterwards without anyone asking whether the task in front of them needs it. Permission modes, settings files, context files and allow rules are all standing grants.

The **unattributable window** (ch. 18). The span between the last reproducible state and the current one, holding every change nobody recorded. Its width is the real measure of how well a system is configured, and nothing inside it can be attributed after the fact.

The **unstated triple** (ch. 4). Three figures a model choice depends on that arrive in no requirements document: the quality bar the output must clear, the ceiling on how long somebody will wait, and the spend the business carries per unit of work.

The **waiting party** (ch. 3). The entity whose clock the request sits on. Naming it decides the request shape on its own: a person watching output appear, a caller blocked on a return value, and a job nobody is waiting for want three different mechanisms.

The **writable channel** (ch. 20). Everything an injected instruction can reach: the assembled prompt, the model's reading of it, and the model's output. Defences are ranked by how far outside that channel they sit.

## Platform Terms Whose Exact Meaning Decides Items

**Adaptive thinking**. The model judging how much visible reasoning a request warrants, rather than a caller setting a token budget for it. On the newest models an explicit `budget_tokens` value is not supported and returns a `400`, and effort is a separate output-level setting (ch. 1, ch. 17).

**Agent SDK**. Anthropic's implementation of the agent loop, with the tool-calling cycle, context handling, permissions and session management already built. Adopting it deletes roughly sixty lines and installs a set of assumptions in their place (ch. 13).

**Batch API**. Asynchronous processing for work nobody is waiting on, priced at a discount against synchronous calls. The discount stacks with other pricing modifiers (ch. 3, ch. 8).

**Context window**. The capacity a single request has for everything it carries: system prompt, tools, conversation history, retrieved material, and the output for the turn. It is working memory rather than storage, and caching changes what those tokens cost rather than whether they occupy room (ch. 1, ch. 7).

**Effort**. A setting governing how much work goes into producing the response. It is independent of model choice and of thinking, which makes model, thinking and effort three settings rather than one quality dial (ch. 1).

**Fast mode**. A speed setting, buying output tokens per second at premium pricing. It does not buy capability (ch. 1).

**Hook**. A deterministic action the runtime executes at a defined point in the sequence, regardless of what the model decided. It is where an absolute prohibition belongs, because a context file cannot enforce one (ch. 14, ch. 21).

**Idempotent**. Safe to repeat. Any retry crossing a boundary that writes performs a second write unless the operation is idempotent, which is why retry policy and write design are one decision (ch. 19).

**MCP configuration scope**. Who loads a server, and nothing else. `local` is private to one project, `project` lives in a committed `.mcp.json` and reaches everyone who clones the repository, and `user` is private to you across every project (ch. 10).

**MCP transport**. How the client reaches the server, decided by process locality. A server documented with a command to run uses stdio, one documented with a URL uses HTTP or SSE, and tools written in the application's own code use an in-process server with no transport at all (ch. 10, ch. 11).

**Messages API**. The synchronous request path, and stateless. The full conversational history goes up on every call, no handle identifies a conversation, and anything absent from the request is absent from the model's view (ch. 3, ch. 16).

**Plugin**. Skills, agents, hooks and MCP servers bundled into one versioned installable unit, which is how a chosen set travels to a team. Plugin skills are namespaced with the plugin's name so two plugins can ship a skill of the same name (ch. 11).

**Prompt caching**. Charging a reduced rate for a prefix seen before. Prefixes assemble in one order, tools then system then messages, and a change at any level invalidates that level and everything after it. The five-minute duration writes at 1.25x input price and the hour at 2x (ch. 8).

**Permission mode**. The standing setting deciding what Claude Code may do without asking. A resumed session restores it with named exceptions: plan mode and bypass mode are never restored, so a resumed session starts where a new one would (ch. 14, ch. 16).

**`stop_reason`**. The field saying why generation ended, and the one place several silent failures are visible. A refusal arrives as a `200` response carrying `stop_reason: "refusal"`, and a `max_tokens` truncation arrives as a `200` as well (ch. 2, ch. 6).

**`tool_use` and `tool_result`**. The two halves of a tool call. Claude returns `stop_reason: "tool_use"` with one or more `tool_use` blocks; your code executes and replies with a new user turn of `tool_result` blocks, each echoing the `tool_use_id` of the call it answers, results first and one per call (ch. 3, ch. 9).

**Prompt injection**. Instructions planted in content the model reads on somebody's behalf, weighed alongside yours because from inside the sequence they are the same kind of thing. The defence is placement and gating rather than a stronger instruction (ch. 20).

**`Retry-After`**. The header saying when to try again. Its absence carries information: a `429` raised by a spend cap carries no such header and keeps failing until access is restored, so retrying it turns a billing problem into an outage (ch. 2).

**Skill**. A packaged procedure Claude loads and follows, as against a tool it calls. It carries knowledge about how work gets done and cannot reach a system holding live state on its own (ch. 11).

**Subagent**. A unit running in its own context-isolated thread, with its own model, system prompt, tools, connected servers and skills. It starts empty and inherits no Skills from its parent, so whatever the next stage needs has to be written into the launching prompt (ch. 7, ch. 11, ch. 12).

**Zero-shot, one-shot and few-shot**. Three prompting modes separated by how many worked examples the prompt carries. Zero-shot describes the task and supplies none. Examples cost input tokens on every call, which makes the choice a priced one (ch. 1).
