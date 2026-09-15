# Chapter 8: Cost Follows What You Resend, Not What You Ask

**Summary**: *A conversation gets more expensive per exchange as it lengthens, while the questions stay exactly the same size. Nothing about the workload changed. The history went up again, and it will go up again next turn, and the turn after that. Read the bill that way and token budgeting, usage tracking and caching stop being three topics. They become one question asked three times: what is resent, what rate is it charged at, and what could be read back instead of processed again.*

## Turn Forty Costs More Than Turn One Because It Carries Turn One

Ask a **short question** of a **long conversation** and you are **not billed** for a short question.

Every request **carries the whole history**. System prompt, tool definitions, every earlier exchange, every tool result. A task running forty turns has **sent its first turn forty times**, so cost climbs with roughly the **square of the turn count** rather than in a straight line.<sup>[1]</sup> Humans asked one thing per turn. The application **paid again** for everything already said.

Call that **carried cost**: what this request is charged for **material you sent**, and were charged for, on an earlier one. The history is doing a job, so it is not waste. It is the term that grows while the questions do not, and on a long run it **dwarfs anything trimming reclaims** from the current turn.

Two consequences follow. A projection built from a **per-request average misjudges long sessions** unless that average came from real ones. And the **strongest lever** is not removing history, but having the **same bytes charged at a tenth** of the price.

## Input, Output and the Two Cache Rates Are Four Different Prices

A **cost model** needs every quantity the billing depends on, starting with the fact that a **token is not a token**.

| What is charged | Rate, against base input | Quantity |
|---|---|---|
| Input | 1x | The whole request, history included |
| Output | Higher, the pricier side per token | The task, capped by `max_tokens` |
| Cache write, 5 minutes | 1.25x | What is new behind your checkpoint |
| Cache write, 1 hour | 2x | The same, held longer |
| Cache read | 0.1x | What matched a live entry |

The frontier model publishes output at **five times its own input rate**, which is the shape to carry: a model that **talks a lot is expensive**, in a way that trimming the prompt never touches. **Reasoning tokens** land on the output side too.

The direction is not a quirk of the rate card, and the reason is worth holding because it makes the fact impossible to reverse under pressure. A **prompt is processed once**, in a **single pass over everything** you sent. Generation runs **one token at a time**, and every one of those tokens costs **its own pass** through the model, conditioned on all the text ahead of it. You are buying one traversal for the prompt and **one per token produced**. Which is why the two sit on **separate budget lines**. Spend tracks **how much the system says** far more closely than how much it is told, and **one averaged rate** quietly **under-prices** every design that talks.

Three facts fall out of that table. A **read at a tenth** means five-minute caching **repays its write after one hit** and the hour version after two. The **multipliers stack** rather than replace, so asynchronous submission, at half off every token including cached ones, applies on top of a cache read.<sup>[2]</sup><sup>[6]</sup> And **`max_tokens` caps a response invisibly** to the model, so lowering it never makes Claude economise: the turn that needed the room is truncated, discarded and billed anyway. A **task budget the model can watch** changes what it does.<sup>[1]</sup>

## A Projection Needs Five Quantities, and Every One Is Knowable Before You Ship

Cost estimates rarely get argued about over arithmetic. Somebody **left a term out** and put a **comparison in its place**.

| Quantity | Where the number comes from before launch | What losing it does |
|---|---|---|
| Requests per period | The forecast, or the traffic this replaces | A price with nothing to multiply |
| Tokens sent per request | Counted, not estimated: the counting endpoint takes a real request body and returns the count without running inference, free<sup>[5]</sup> | The largest term becomes an opinion |
| Tokens returned per request | A sample run across representative inputs | The dearer side of the bill vanishes |
| Rates for the model and features chosen | Published per model, per operation | You price a design somebody else built |
| Share of input expected from cache | The prefix you can hold stable | A cached prefix overstated tenfold |

Two named traps live here and both look like diligence. The **borrowed baseline** scales last year's **cost per request**, or a neighbouring team's, importing a different token profile and price list with it. The **deferred number** calls **token counts unknowable until production**, removing the quantities the bill is proportional to.

Neither survives one question. What here was **measured**?

## You Cannot Cut What You Have Not Attributed

Log three things on every call. **Tokens in, tokens out**, and which **feature made** it.

Without them a **cost spike** supports one question, which is why the **total is high**. With them it supports the useful one: which step, on which kind of request. A flow that looks **uniformly expensive** usually is not, and one step commonly **carries most of the spend**.

**Attribution comes before optimisation**, and not for tidiness. Every remedy available to a team that has not measured is an **across-the-board remedy**. The **uniform cut** applies one reduction to features that were **never driving anything**, and nobody can say which had room to absorb it. The **blanket downgrade** trades quality on all traffic against **spend concentrated** in some of it.

Three mechanics decide whether the numbers are true. **Parallel tool calls** in one turn share an identifier and repeat the same usage figures, so **deduplicate rather than sum**. Once an agent spawns subagents the **plain usage field undercounts**, and whole-tree accounting comes from the **per-model breakdown**. Totals are per call, so a session making several is yours to accumulate.<sup>[7]</sup> Above your own code, Claude Code attributes recent consumption to **skills, subagents, plugins and MCP servers**, and per-user metrics export over **OpenTelemetry**.<sup>[8]</sup>

## Caching Reuses the Prefix Instead of Deleting It

Watch a multi-turn application get expensive and the reflex is to cut. Keep the last two messages, summarise after every turn, stop sending tool results. Each shrinks the bill by **damaging the product**, and summarising adds a **billed call per turn**. **Caching attacks the identical quantity** and leaves the content alone.<sup>[1]</sup>

Prefixes assemble in one order, **tools then system then messages**, and a change at any level **invalidates that level** and everything after it. Matching is **exact, byte for byte**. A **cache checkpoint** marks where a reusable stretch ends, **four per request at most**, and a write happens only at a checkpoint. A later request hashes its own checkpoint and, failing a match, walks backward through at most **20 positions** hunting for an entry an earlier request wrote.<sup>[3]</sup>

That explains the commonest failure here. Put the checkpoint on a **block that changes every request** and the entry is keyed to content that never recurs, so you pay each write and **read nothing back**. The **moving prefix** is the same defect one step earlier: a timestamp or a request identifier ahead of the stable content. Anthropic measured a **25-token status line** at the front of a system prompt taking one run from **$0.59 to $4.24**, **worse than caching off**.<sup>[1]</sup> Per-request text belongs in the **newest user turn**.

**Three conditions** hold together or the workload gets nothing.

| Condition | Why it bites |
|---|---|
| The prefix is identical | One added word ahead of the checkpoint is a full reprocess |
| It recurs inside the lifetime | Five minutes by default, refreshed free on each hit, timed from the start of the request rather than the end of the response |
| It clears the minimum length | Varies by model, and a short prompt is processed with no caching and no error<sup>[3]</sup> |

The silence in that last row is the trap. **Both usage counters read zero**, and that is the entire signal. Which is also the difference between the two diagnostics: usage answers whether the **cache hit**, and cache **diagnostics answers what changed**, naming the **first divergence** among model, system, tools and messages.<sup>[4]</sup>

One shape defeats the default lifetime. A loop **waiting on humans between turns** comes back in minutes, not seconds, so the **entry expires unused**. That is what the **hour-long write** buys, at **twice the price**, repaying itself on the first miss it prevents.<sup>[1]</sup>

## How Wrong Answers Are Built on This Material

Wrong answers on cost are rarely reckless. They are a **real economy applied to the wrong term**, which is why each reads as reasonable. **Four assumptions** produce most of them.

- **Deletion offered where reuse was available**. **Truncating history**, **summarising every turn**, **dropping tool results**. Right when material is finished, which chapter 7 covers. Wrong when a **stable prefix is resent** and the conversation still needs it.
- A **term missing from the model**. **Output omitted**, **token counts postponed to production**, a **per-request figure imported from elsewhere**. Each needs its missing quantity not to vary, and no scenario asking for a projection says that.
- An **untargeted remedy for a problem nobody located**. A **uniform reduction**, a move to the **cheapest model**, a declaration that **growth is unavoidable**. All three assume **spend is spread evenly**, which nobody measured.
- A **control the model cannot see**, **treated as a budget**. **Lowering the reply cap** changes what survives, not what is spent.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Cost per exchange climbs as conversations lengthen, requests unchanged | Cache the stable prefix, so resent history bills at the read rate | Truncating to recent turns, or summarising every turn |
| Nobody can say which feature drives the bill | Instrument token usage per feature, then decide | A uniform cut, a downgrade, or accepting the growth |
| A projection is wanted before launch | Volume, tokens up, tokens down, the rates, expected cache reads | Scaling a per-request cost from somewhere else |
| Caching is on and the read counter sits at zero | Find the byte that moves ahead of the checkpoint | More checkpoints, or a longer lifetime |
| Volume is high and nobody waits on any single result | Asynchronous submission, discounting every token | Treating it as interactive, or buying the saving with quality |
| Spend must be capped without wrecking hard turns | A budget the model can see, with a hard limit behind it | Lowering the response cap, which discards work and bills it |

One habit carries past the table. For any option, ask whether it changes how **many tokens move**, what **rate they are charged** at, or **only where the bill becomes visible**. Reporting shrinks nothing by itself, and deletion is one route among several to a smaller number.

## What to Remember

- **Carried cost** is what a request pays **again** for material **sent on an earlier one**. Because every turn **resends the whole history**, a long task's cost grows with roughly the **square of its turn count** while the **questions stay the same size**.
- **Four rates, not one price**: **input at 1x**, **output higher per token**, cache writes at **1.25x for five minutes** and **2x for an hour**, **cache reads at 0.1x**. A read at a tenth repays a five-minute write after **one hit** and an hour-long write after two, and the **multipliers stack** with the asynchronous discount.
- A **projection carries five quantities**: **request volume**, **tokens sent**, **tokens returned**, **rates** for the model chosen, and the **share expected from cache**. All are **knowable pre-launch**, because the **counting endpoint** returns input tokens free, without running inference.
- **Attribution precedes optimisation**: **log tokens in, tokens out** and the **calling feature** per call, because every remedy available without it is **across the board** and damages features that **never drove spend**. **Deduplicate parallel tool calls** sharing one identifier, read subagent spend from the **per-model breakdown**, and **accumulate per-call totals** yourself.
- **Caching reuses the prefix** rather than deleting it, beating **truncation and per-turn summarising** while the conversation still needs its history. Prefixes build **tools, then system, then messages**, so a **change at one level invalidates** everything after it.
- A **cache checkpoint** marks where a reusable stretch ends, **four per request at most**; a **write happens only at a checkpoint**, and a **read walks back 20 blocks** at most hunting one. Put a checkpoint on **content that changes each request** and you **pay every write**, reading nothing back.
- **Caching needs three conditions** together: **identical bytes**, **recurrence inside the lifetime** (**five minutes**, refreshed free on each hit), and a prefix over the **model's minimum**. Falling short is **silent**: no error, **both usage counters at zero**, which is when diagnostics names the **first divergence** among **model, system, tools and messages**.
- A **task budget** is **visible to the model**, so it changes what Claude does, while **`max_tokens` caps a response invisibly**: lowering it **discards the turn** that needed the room and **bills it anyway**.

## Endnotes for this chapter

1. Every turn of an agentic task resends the entire growing conversation, so a 40-turn task sends its first turn 40 times and task cost grows with roughly the square of turn count; caching does not stop the resending, but each resend is billed at the cache-read rate, a tenth of the input price, with the 1.25x cache-write rate paid only on what is new. Content that changes per request, such as a timestamp or a queue position, placed ahead of the stable prefix turns every request into a full cache write: a 25-token status line at the front of the system prompt cost $4.24 per run instead of $0.59, more than running with caching off. For a loop that waits on humans between turns, the 1-hour cache duration costs 2x the input price to write instead of 1.25x and pays for itself on the first prevented miss. A task budget saves money because the model sees a live token countdown and self-regulates; `max_tokens` is a safety cap that saves nothing, because it caps a single response invisibly to the model, and the turns that needed the room are discarded and still billed. Anthropic, *Optimizing for cost and intelligence* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)

2. Prompt caching uses pricing multipliers relative to base input token rates: a 5-minute cache write is 1.25x base input price, a 1-hour cache write is 2x, and a cache read is 0.1x, so caching pays off after one cache read for the 5-minute duration or after two for the 1-hour duration. These multipliers stack with other pricing modifiers, including the Batch API discount and data residency. The published rate card lists Claude Fable 5 at $10 per million base input tokens and $50 per million output tokens. Anthropic, *Pricing* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/pricing](https://platform.claude.com/docs/en/about-claude/pricing)

3. Cache prefixes are created in the order `tools`, `system`, then `messages`, and changes at each level invalidate that level and all subsequent levels; cache hits require 100% identical prompt segments up to and including the block marked with cache control. Up to 4 cache breakpoints may be defined; cache writes happen only at a breakpoint, and on each request the system computes the prefix hash at the breakpoint and walks backward one block at a time through a lookback window of 20 blocks checking for entries prior requests wrote. Placing the breakpoint on content that changes every request means the lookback finds nothing, so the correct placement is the last block whose prefix is identical across the requests that should share a cache. The default cache lifetime is 5 minutes, refreshed for no additional cost each time the cached content is used, and measured from the start of the request that writes or reads the entry rather than from the end of its response. There is a per-model minimum cacheable prompt length; requests to cache fewer tokens than the minimum are processed without caching and no error is returned, and the way to verify is that both `cache_creation_input_tokens` and `cache_read_input_tokens` are 0. Anthropic, *Prompt caching* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)

4. Without cache diagnostics the only signal of a miss is `usage.cache_read_input_tokens` dropping to zero, with no indication of what changed. Passing the previous response `id` as `diagnostics.previous_message_id` makes the API compare the two requests and report the first point of divergence as `model_changed`, `system_changed`, `tools_changed` or `messages_changed`; the response reports the earliest divergence only. Diagnostics answers whether the request changed, while `usage.cache_read_input_tokens` answers whether the cache hit. Anthropic, *Cache diagnostics* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/cache-diagnostics](https://platform.claude.com/docs/en/build-with-claude/cache-diagnostics)

5. The token counting endpoint accepts the same structured list of inputs used for creating a message, including system prompts, tools, images and PDFs, and returns the total number of input tokens without running inference; it is free to use and supported on all active models, and is documented as a way to manage costs proactively and make model routing decisions. Anthropic, *Token counting* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/token-counting](https://platform.claude.com/docs/en/build-with-claude/token-counting)

6. The Message Batches API processes large volumes of requests asynchronously, reducing costs by 50% and increasing throughput, with most batches finishing in less than 1 hour and results available within 24 hours; the discount applies to every token of a request, including cached ones. Anthropic, *Batch processing* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/batch-processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing)

7. When Claude uses multiple tools in one turn, all messages in that turn share the same ID with identical usage data, so deduplicate by ID to avoid double-counting. The result message carries `total_cost_usd` and cumulative usage for that call; where the agent spawns subagents, the per-model usage map is what gives whole-tree token accounting, because the plain `usage` field undercounts as soon as nesting occurs. Each call returns its own total and the SDK provides no session-level total, so an application making several calls accumulates them itself. Anthropic, *Track cost and usage* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)

8. The `/usage` command shows detailed token usage for the current session, and on a plan it also breaks recent usage down by attribution to skills, subagents, plugins and individual MCP servers, each as a percentage of the total, alongside behavior flags raised when a behaviour such as long context or cache misses accounts for 10% or more of recent usage. OpenTelemetry export works on every setup and is the only option that streams per-user token and cost metrics into an organisation's own observability stack in near real time. Anthropic, *Manage costs effectively* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/costs](https://code.claude.com/docs/en/costs)
