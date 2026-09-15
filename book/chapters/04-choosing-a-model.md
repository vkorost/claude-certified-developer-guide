# Chapter 4: A Model Choice Is Three Numbers You Were Not Given

**Summary**: *A ticket arrives naming what the feature must do and nothing about how well, how fast or how cheaply. Under a ticket like that every model is defensible, which is the tell that the ticket is incomplete rather than the decision hard. This chapter covers the three figures missing from it, what separates the tiers, and why a model you pinned still shifts underneath you.*

## The Requirement Arrived With No Quality Bar, No Latency Ceiling and No Cost Per Unit

The **ticket is complete** on its own terms: what the feature does, the acceptance criteria, the edge cases, the owner, the date. At the bottom, **one line** asking you to use **whichever Claude model** gives the **best results**.

That line is a **hole with a sentence** in front of it. Best judged by whom, inside what **wait**, at what **price**?

Call the three missing figures the **unstated triple**: the **quality bar** the output must clear, the **ceiling** on how long somebody will sit waiting, and the **spend** the business carries per unit of work. Nothing can be traded until somebody says what it is **traded against**. While all **three stay blank** every tier is arguable, and **four defensible answers** mean an **underspecified question** rather than a hard one. Humans supply **functional requirements** unprompted; the triple has to be **asked** for.

Three ways to skip that conversation look like progress. The **borrowed leaderboard** scores work that is not yours and **sets no threshold**. The **mid-tier hedge** guesses all three figures and **records none**. The **top of the range by default** spends everything on quality and leaves the **other two unbudgeted**, the standard and most expensive mistake here.

## Four Tiers, Ordered the Same Way by Price and by Capability

Once the triple exists, the ladder is short. From **lowest cost and capability to highest**: Claude Haiku 4.5, Claude Sonnet 5, Claude Opus 5, then Claude Fable 5 at the frontier.<sup>[1]</sup>

| Tier, cheapest first | Per million tokens, in then out | Documented for |
|---|---|---|
| Claude Haiku 4.5 | $1 and $5 | Real-time work, high volume, subagents<sup>[2]</sup> |
| Claude Sonnet 5 | $2 and $10 | Code, analysis, content, visual understanding, tool use<sup>[2]</sup> |
| Claude Opus 5 | $5 and $25 | Multihour coding agents, refactors, vision-heavy work<sup>[2]</sup> |
| Claude Fable 5 | $10 and $50 | Long-horizon agents, deep reasoning, research<sup>[2]</sup> |

**Names and rates move**, so confirm the lineup when you build. The shape outlives the roster: each step up roughly **doubles the per-token price**, and capability climbs with it. Capacity can settle a choice on its own. Claude Opus 5 serves a **1M token context window** by default and up to **128k output tokens**<sup>[2]</sup>, where Claude Haiku 4.5 **caps output at 64k**.<sup>[3]</sup> One very long answer **eliminates a tier** before quality comes up.

## Price a Model on Cost Per Completed Task, and Price the Hardest Tenth

Per token the frontier looks extravagant, and **per token is the wrong unit**. You buy finished work, and a more capable model finishes with **fewer turns**, less searching and less backtracking, which routinely **swallows the premium**.<sup>[1]</sup> The **direction is not fixed** either. On one research benchmark Anthropic measured the frontier model at low effort as more accurate and about **10% cheaper per task** than the mid-tier model; on a coding subset it inverted, with Claude Opus 5 alone matching Claude Fable 5's accuracy at **roughly 60% of the cost**.<sup>[1]</sup>

The **median priced and the tail ignored** sits underneath most cheap-tier decisions. On the typical request every tier looks alike and the cheapest looks best. The bill is set by the requests it fails: a failed task **bills its tokens**, then the retry, then the **downstream damage**. In one twenty-problem run two problems carried 43% of the spend.<sup>[1]</sup> At the floor, Claude Haiku 4.5 answered a graduate-level question set at a **tenth of Claude Opus 5's cost** per question, scoring 63% against 92%.<sup>[1]</sup> Excellent where **output is checkable**. Poor where a **wrong answer travels**.

An **eval promotes a model change**. An argument does not. **Step up** when a measurement shows the tier missing the bar on your hardest cases, **step down** when one shows a **cheaper tier holding** it; chapter 19 owns the eval.

## Reasoning Depth and Raw Speed Are Bought Separately From Capability

Chapter 1 established that **model, thinking and effort** are independent settings. What belongs to a selection decision is that the **tiers do not all offer** the same settings, so a reasoning requirement can decide the model before cost does.

Effort runs on all supported models with no beta header, defaults to `high`, and offers `low`, `medium`, `high`, `xhigh` and `max`, with the **top levels absent on some models**.<sup>[4]</sup> Adaptive thinking varies harder. Claude Fable 5 and Claude Mythos 5 have it always on, and **disabling it errors**.<sup>[2]</sup> Claude Opus 5 runs it by default and accepts `thinking: {"type": "disabled"}` only at **`high` effort or below**.<sup>[5]</sup> Claude Haiku 4.5 has **no interleaved thinking**.<sup>[6]</sup> **Effort is a latency lever** too: Anthropic measured **`low` at 4.5 minutes per problem** against 7.9 at the default.<sup>[1]</sup>

One lever isolates latency completely. **Fast mode** runs the same model on a **faster inference configuration**. Intelligence and capabilities do not change. It buys up to **2.5 times the output tokens per second** on Claude Opus 5 and Claude Opus 4.8, at $10 and $50 per million tokens against the standard $5 and $25.<sup>[7]</sup> Speed bought with money, quality held still. So when speed is the problem and quality holds, **lower effort**, streaming and **fast mode** leave the output alone. **Dropping a tier does not**.<sup>[8]</sup>

## A Model ID Is Frozen and the System Around It Is Not

From the 4.6 generation onward, **model IDs are dateless** and each names one **fixed snapshot**. **Weights are never updated** under an existing ID; an update ships as a new one. The common misreading is that a dateless ID routes to whatever is newest.<sup>[9]</sup> On earlier models the **short name is a pointer**, resolving to the **most recent dated snapshot**. Two strings that look alike, **guaranteeing opposite things**.

A **pinned ID is not a frozen system**. The **serving layer** around the weights, meaning the router, the classifiers and the sampling logic, keeps changing, and an infrastructure update is the likely cause when behaviour shifts on an ID nobody moved.<sup>[9]</sup> Nor can you stay put forever: models are deprecated then retired with at least **60 days' notice**, **requests after that date fail**, and partner platforms keep their own schedules.<sup>[10]</sup>

The **drop-in that is only a string change** is what this produces. The request shape survives a generation and several other things do not. On Claude Opus 4.7 and later, a non-default `temperature`, `top_p` or `top_k` **returns a `400`**, and so do manual thinking budgets and assistant prefill. A request without a thinking field ran **without thinking** on Claude Opus 4.8 and thinks on Claude Opus 5, putting **reasoning tokens** under a **`max_tokens` cap** sized without them.<sup>[5]</sup>

The **quietest failure raises nothing**. Instructions accumulate for the model that needed them, and a **newer one obeys them literally**: prompts tuned for Claude Opus 4.8 **cost 36% more per ticket** on Claude Opus 5 for no gain in accuracy, and an **audit took 14% off** while raising it.<sup>[1]</sup> Nothing breaks. The bill moves, and a model choice turns out to carry a **maintenance schedule**; chapter 18 owns pinning it.

## How Wrong Answers Are Built on This Material

Wrong answers here describe things a careful developer really does: benchmarking, choosing a **mid-range model**, **upgrading to the newest release**. What makes one wrong is the **assumption underneath**.

- A **target guessed instead of obtained**. The option names a tier, a model or a benchmark while the **quality bar**, the **wait** and the **spend** are all blank. A **public score shares the defect**: it **measures work that is not yours** and **sets no threshold**.
- **One axis optimised and two left unbudgeted**. **Maximum capability**, or **minimum price**, treats the decision as one-dimensional. Ask what an option silently does to the **other two numbers**.
- A **release treated as a string swap**. The **identifier is assumed to carry** the prompt, the settings and the token arithmetic along. Some of that **breaks loudly with a 400**. Some only **costs more**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Functional requirements only, no figures for speed, spend or quality | Obtain all three targets, then choose against them | Naming a tier or a benchmark before the figures exist |
| Spend is the problem, output quality holds | Cost per completed task, priced on the hardest tenth | A per-token price judged on the typical request |
| Latency is the problem, output quality holds | A lever that leaves the output alone: effort, streaming, paid speed | A smaller tier, which changes what is produced |
| A working integration moves to a newer release | Re-baseline settings, prompts and token counts | Assuming a drop-in because the request shape held |
| A model in production has a retirement date | Migrate before it, because later requests fail | Treating deprecation as advisory, or trusting one platform's dates |

One habit transfers past the table. For every option, say which of the **three numbers it moves** and which two it leaves unstated.

## What to Remember

- The **unstated triple** is the **quality bar**, the **latency ceiling** and the **cost per unit** of work. **No model is defensible** until all three exist, so obtain them before shortlisting.
- **Capability and price** rise together across **Claude Haiku 4.5**, **Claude Sonnet 5**, **Claude Opus 5**, **Claude Fable 5**, each step roughly **doubling per-token price**. Rosters go stale: **confirm the lineup** when you build.
- Compare on **cost per completed task**, not per token, because a stronger model uses fewer turns. **Price the hardest tenth**: a **failed task bills** its **tokens**, the **retry**, and the **downstream damage**.
- An **eval promotes a model change**, both ways: **step up** when a measurement shows the **bar missed** on your **hardest cases**, **step down** when one shows a **cheaper tier holding** it.
- Reasoning modes vary by model: **`xhigh` and `max` are absent** on some, **adaptive thinking** is always on at the frontier and **cannot be disabled** there, and **Claude Haiku 4.5** has **no interleaved thinking**.
- **Fast mode buys latency alone**: the **same model**, **no capability change**, **2.5 times the output tokens** per second, at double the standard rate.
- A **dateless model ID** is a **fixed snapshot**, never a pointer to the newest, while **pre-4.6 short names point** at the latest dated snapshot. **Weights stay fixed** and the **serving layer does not**.
- A **release changes behaviour, not just an identifier**, and **retirement makes requests fail** after **60 days' notice**. Loud breaks are **400s** on **sampling parameters**, **thinking budgets** and **prefill**; the quiet cost is money, when **stale instructions** are obeyed literally.

## Endnotes for this chapter

1. Cost and intelligence levers, including the ordering of the current models from lowest to highest cost and capability; the instruction to compare models on cost per completed task because a more capable model finishes with fewer turns and less backtracking; the DeepResearch Bench II result in which the frontier model at low effort was more accurate and about 10% cheaper per task than the mid-tier model; the SWE-bench Pro subset result in which Claude Opus 5 alone matched Claude Fable 5 within run-to-run noise at about 60% of the cost; the instruction to price the hardest tenth of a workload rather than the median, and the WideSearch run in which two of twenty problems carried 43% of the spend; the GPQA Diamond comparison in which Claude Haiku 4.5 answered at about a tenth of Claude Opus 5's cost per question at 63% against 92%; the effort latency figures of 4.5 minutes per problem at low effort against 7.9 at the default; and the support-desk audit in which prompts written for Claude Opus 4.8 cost 36% more per ticket on Claude Opus 5 for no change in accuracy, while running the audit made the same work 14% cheaper and raised accuracy from 92% to 97% of tickets. Anthropic, *Optimizing for cost and intelligence* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)

2. The documented starting points and example use cases for the four current tiers (Fable 5, Opus 5, Sonnet 5, Haiku 4.5); Claude Opus 5's 1M token context window by default and 128k maximum output tokens; the always-on adaptive thinking of Claude Fable 5 and Claude Mythos 5, on which no thinking configuration is required and a disabled setting returns an error; and per-tier list prices of $10 and $50, $5 and $25, $2 and $10, and $1 and $5 per million input and output tokens. Anthropic, *Choosing the right model* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/choosing-a-model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model)

3. Output limits by model: the current frontier and Opus-class models, together with Sonnet 5, Opus 4.6 and Sonnet 4.6, support up to 128k output tokens per request, while Haiku 4.5, Sonnet 4.5 and Opus 4.5 support up to 64k. Anthropic, *Thinking* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/thinking](https://platform.claude.com/docs/en/build-with-claude/thinking)

4. The effort parameter is available on all supported models with no beta header, defaults to high, and offers the levels low, medium, high, xhigh and max, with xhigh a newer level that some models supporting max do not offer. Anthropic, *Effort* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/effort](https://platform.claude.com/docs/en/build-with-claude/effort)

5. Migration behaviour across recent releases: thinking runs by default on Claude Opus 5 where a request without a thinking field ran without thinking on Claude Opus 4.8, with `max_tokens` remaining a hard limit on thinking plus response text; disabling thinking is accepted only at effort high or below and is validated on each request; setting `temperature`, `top_p` or `top_k` to a non-default value returns a `400` error on Claude Opus 4.7 and later; manual extended thinking with `budget_tokens` returns a `400` error; and prefilling the assistant message returns a `400` error. Anthropic, *Migration guide* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/migration-guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide)

6. Interleaved thinking is automatic on every model that supports adaptive thinking and requires no beta header, and Claude Haiku 4.5 does not support interleaved thinking. Anthropic, *Thinking* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/thinking](https://platform.claude.com/docs/en/build-with-claude/thinking)

7. Fast mode delivers up to 2.5 times higher output tokens per second from Claude Opus 5 and Claude Opus 4.8 at premium pricing, runs the same model on a faster inference configuration with no change to intelligence or capabilities, and is priced at $10 per million input tokens and $50 per million output tokens on the supported models. Anthropic, *Fast mode* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/fast-mode](https://platform.claude.com/docs/en/build-with-claude/fast-mode)

8. Selecting an appropriate model is one of the most direct ways to reduce latency, with Claude Haiku 4.5 offering the fastest response times, alongside minimising input and output tokens and streaming to improve perceived responsiveness. Anthropic, *Reducing latency* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-latency](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-latency)

9. Model IDs identify a pinned version whose underlying model stays constant for the lifetime of that ID; from the 4.6 generation onward IDs are dateless and each is the canonical fixed snapshot rather than an evergreen pointer to the latest or best-performing version, with updates shipping under a new ID; models before that generation have short aliases on the Claude API that resolve to the most recent dated snapshot; and the serving infrastructure around a model, including the request router, safety classifiers and sampling logic, can change and occasionally produce minor differences in observable behaviour on an unchanged model ID. Anthropic, *Model IDs and versioning* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions](https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions)

10. Anthropic provides at least 60 days' notice before retiring a publicly released model, requests to models past the retirement date will fail, and partner-operated platforms including Amazon Bedrock and Google Cloud set their own retirement schedules so a model's lifecycle status and dates can differ. Anthropic, *Model deprecations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/model-deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)
