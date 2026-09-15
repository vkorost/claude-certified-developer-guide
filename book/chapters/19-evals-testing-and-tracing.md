# Chapter 19: A Test Says It Broke; a Trace Says Where

**Summary**: *Four questions run this chapter and each one asks where. Where an error's cause sits, which decides whether waiting is a remedy. Where in the run a failure happened, which only a step-by-step record answers. Where in the stack it started, your integration layer or the model's output, which is the distinction most scenarios turn on. And where in a population a measurement holds, which decides what an average may license. Get the location wrong and every remedy after it is aimed at the wrong thing.*

## A Green Suite Is a Claim About Parts, Not About the Product

The **retrieval function passes** its unit tests. So does the **prompt builder**, and a check confirms the model call returns the fields the parser wants. Run your flow end to end and it answers a policy question from **general knowledge** instead of from the **retrieved policy**, with **nothing raised anywhere**.

The **join broke**. Retrieval hands back a **list of records**, the builder expects text, and the assembled prompt carries something the model reads as noise. Both components are correct. **Nobody wrote the contract** between them, so nothing exercised it.

Every check observes a **bounded field of view**: one function, one call, one flow, one number. A failure is findable only by a check whose field of view contains it. A **seam belongs to neither component** beside it, so it sits inside nobody's until you build a check whose **field of view is the seam**.

**Coverage counted in components** is the anti-pattern, and the count is what makes it convincing. Every part is covered. The **joins are not parts**.

## An Error's Bucket Is Where Its Cause Lives, Not Which Status Code Carried It

One question sorts every failure your application throws. Could the **clock alone change the outcome**?

If yes, the cause sits in the **state of the world**. Capacity, load, a per-minute limit, a dropped connection. The documented shapes are **`429` rate limits**, **`529` overloads**, **5xx server faults** and **timeouts**.<sup>[1]</sup> Waiting is a **real remedy** for those alone.

If no, the **cause sits in the request**. A malformed body, a revoked key, a permission the credential lacks, a resource that is not there: **`400`**, **`401`**, **`403`**, **404**.<sup>[1]</sup> Every later send returns the **identical error**. **Repair** it or **reject** it, and neither of those is a retry.

That mapping is a hypothesis. Two documented cases break it in opposite directions. A **spend-cap `429` carries no `retry-after`** and keeps failing until access resumes,<sup>[1]</sup> the same status in the other bucket, with the **missing header as the tell**. A request too large for the context runs the other way: Claude Code **lowers `max_tokens`** and sends a different request rather than repeating one that **fails identically**.<sup>[2]</sup>

The **status code read as the diagnosis** is the anti-pattern. The code proposes a cause; the header, the message and the request ID settle it.<sup>[1]</sup>

## Misclassifying Costs Silence in One Direction and Noise in the Other

Call a **terminal failure retriable** and every attempt is **guaranteed to fail**. Your **budget drains**. The real error surfaces late wearing the shape of slowness, and on a rate limit each attempt **deepens the limit** that caused it. Call a **retriable failure terminal** and a condition that would have cleared in seconds reaches a person as a **broken feature**. That one gets **reported the same day**. The first looks like latency and runs for months, so terminal is the **cheaper guess** when the cause is unclear.

A second question sits underneath and it is not the same one. Has **anything already happened**? **Retriability** is a **property of the cause**; **safety** is a property of what a **first attempt committed**. Claude Code will not re-issue a request that failed after Claude had completed a tool call, because the retry would **run those calls twice**.<sup>[2]</sup> Any retry crossing a **boundary that writes** is a second write unless the write is **idempotent**.

Where recovery stops is a routing decision, and retry mechanics themselves belong to chapter 2. **Impact and recoverability predict** whether a person must act now, where **one severity predicts nothing**.

## A Trace Records Each Step, and It Gets Built Before the Failure

A **test reports** that a failure exists. Nothing in a pass or a fail says which step produced it. A **trace is that missing record**: every step of the run with its **prompt**, its **tool calls**, its **intermediate output** and its **timing**, so the **failing step** is visible instead of reconstructed.

The Agent SDK shows the shape worth copying. Each **turn of the loop** is a span, each API call a child span carrying **model name, latency and token counts**, each tool invocation a child span with the **permission wait separated** from the execution, and a **subagent's spans nest** under the parent's tool span so a delegation chain reads as one trace.<sup>[3]</sup>

Two properties decide what that record is good for. **Telemetry is structural by default**, carrying **durations, model names and tool names** but not the **content** the agent reads and writes, which is **opt-in**,<sup>[3]</sup> so the default trace says which step and how long and never what was said. And **export fails quietly**. An **unreachable collector** leaves the agent running and the **telemetry dropped**, with no error raised in your application.<sup>[3]</sup> The instrument has the same failure mode as the thing it watches.

## The Model's Output Is the Line, and You Read It Before Blaming Either Side

Go to the trace and **read what the model returned**, before anything downstream touched it. Three readings exhaust the cases.

| What the trace shows | Where the fault is | What changes |
|---|---|---|
| Returned content is what was wanted, and the system did the wrong thing with it | Below the model | The parser, the handoff, the tool wrapper, the dispatcher |
| Returned content is wrong, and the input was truncated, malformed or missing its retrieved context | Still below the model | The step that built the input |
| Returned content is wrong, and the input arrived complete and well formed | The model's output | Examples, constraint, prompt, model choice |

The **middle row costs teams weeks**. It presents as a model failure, and every remedy aimed at the model leaves it standing. Hence the rule: a **model failure** is a model failure only after you have **read the input the model received**. **Read what was sent**, never the template meant to produce it. Configuration debugging runs the same two steps: confirm the **instruction reached the context window**, because once it has, the problem is how it is written.<sup>[4]</sup>

A failure crossing a boundary **carries its identity or loses** it. Chapter 6 covers the tool-result half, where **`is_error`** lets the model react to a failure it would otherwise read as data.<sup>[5]</sup> The generalisation belongs here: one message standing for an **invalid identifier**, an **expired token**, a **timeout** and a **policy refusal** has discarded the distinctions recovery runs on. **Recovery can be no more specific** than the error reaching it.

## An Average Is Only a Fact About the Population It Averaged

An **eval** is the check whose field of view is the **whole run's output**. It cannot say where **anything broke**. What it answers is narrower and nothing else answers it: did your **change help**, on a **fixed set of cases**.

Build for **coverage before rubric**. Anthropic's guidance is that **20 to 50 tasks** drawn from **real failures** make a good start, and that cases where a behaviour should occur and cases where it should not both belong.<sup>[6]</sup> A **judge** returns a confident-looking number. It means nothing until its **agreement with expert human judgement** has been measured and recalibrated as things move.<sup>[6]</sup> Ambiguity in a task specification becomes **noise in the metric**, so the standard for a case is that **two domain experts** reach the same verdict independently.<sup>[6]</sup>

Then granularity. An **overall figure licenses nothing** about any segment inside it. Slicing by category requires the **record to carry the category**, because nothing can be measured along an **axis the data does not hold**, which makes adding that field your first move whenever performance differs by group. A flat average is also consistent with a change that **repaired one group and broke another**, which is why you read the per-case results rather than the mean alone.

## How Wrong Answers Are Built on This Material

Wrong answers on diagnosis are **competent engineering** aimed one step **too early** or one level **too coarse**. Four assumptions produce most of them.

- A **remedy applied before the cause is located**. **Lowering a sampling setting**, **swapping the model**, **raising a threshold**, **adding a validator**. Each is a real fix offered while the **failing step is unknown**, and correct only once the scenario has isolated the break.
- A **distinction collapsed rather than restored**. **One severity for every error**, **one message for every backend fault**, **one confidence cut** for every outcome, an **empty result** standing in for a failure. The scenario usually names the collapse as the symptom, and the option offering more of it reads as **tidying**.
- An **aggregate offered where a segment was named**. A **global threshold**, one **overall accuracy figure**, **review spread evenly across categories**. All are right when the **population is uniform**, which the scenario has usually just denied.
- **Repetition where the request itself has to change**. **Retrying a rejection**, **duplicating requests** against a rate limit, **retrying unbounded**, adding a **fresh credential** to a throttle. Correct only where **time was the variable** that could move.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| A status names a temporary pacing or capacity condition | Back off with a cap, honouring the wait the service supplies | A fresh credential, a duplicated request, or dropping the failing check |
| The same request sometimes succeeds and sometimes fails | Capture full input and output on both sides, then compare | Changing a parameter, counting the frequency, or filing it as expected |
| Every component passes its tests and the flow is wrong | Exercise the handoff with real output from the producing step | More component tests, or the model blamed for the result |
| One overall figure is quoted and errors are known to cluster by category | Record the category on each item, measure per category, sample inside the clusters | A global threshold, acting on the aggregate, or reviewing only low-confidence items |
| Part of a job failed and the rest succeeded | Report the failed part: what was attempted, the failure type, any partial result | Returning empty, aborting everything, or retrying unbounded |

One **habit transfers** past the table. For any option offered as a fix, ask what it would have to **observe** to be right, then check whether **anybody in the scenario is observing** it.

## What to Remember

- Every check has a **field of view**, the **bounded span of the run** it can see: one function, one call, one flow, one number. A **seam sits inside neither component**, so a suite where **every part passes** claims something about **parts, not about the product**.
- One **question sorts** an error: could the **clock alone change the outcome**. Yes is a **transient cause** where **waiting** is the remedy, as in **`429`, `529`, 5xx and timeouts**; no puts the cause in the **request**, leaving **repair or rejection**, as in **`400`, `401`, `403` and `404`**. The status only **proposes a cause**: a **spend-cap `429` carries no `retry-after`** and **never clears** on its own.
- The two misclassifications **cost differently**. **Terminal treated as retriable drains the budget**, **delays** the real error and **deepens a rate limit**; **retriable treated as terminal** shows a **broken feature** and gets **fixed the same day**, so raise it as **terminal when unsure**. Separately, **retriability** is a property of the **cause** and **safety** is a property of what **already happened**: a retry across a **boundary that writes** is a **second write** unless it is **idempotent**.
- A **trace records each step** with its **prompt, tool calls, intermediate output and timing**, turning "a **case failed**" into "**step four failed**, and here is what it received". Default telemetry is **structural**, holding **durations, model and tool names** but **no content** until content is **opted** in.
- **Isolate origin** by reading what the **model returned** before anything downstream touched it. **Right output, wrong behaviour** is the **integration layer**, and so is **wrong output** from a **truncated or malformed input**. Only wrong output from a **complete, well-formed input** answers to **prompts, examples or model choice**.
- A **failure crossing a boundary carries its identity** or loses it, and **recovery is no more specific** than the error reaching it: one message covering an **invalid identifier**, an **expired token**, a **timeout** and a **refusal** has **discarded what recovery runs** on.
- An **eval measures** whether a change helped, never where it **broke**. **Coverage beats rubric polish**: **20 to 50 cases** drawn from **real failures**, including cases where the **behaviour should not occur**, and a **judge means nothing** until its agreement with **human labels is measured**.
- An **average** is only a fact about the **population it averaged**. Nothing can be **sliced along an axis** the **record does not carry**, so **record the category** first and **read the per-case results**, because a **steady mean** hides a **repair to one group** and a break in another.

## Endnotes for this chapter

1. The Claude API returns a predictable set of HTTP error codes: `400` `invalid_request_error`, `401` `authentication_error`, `402` `billing_error`, `403` `permission_error`, `404` `not_found_error`, `409` `conflict_error`, `413` `request_too_large`, `429` `rate_limit_error`, `500` `api_error`, `504` `timeout_error` and `529` `overloaded_error`; a `429` raised by reaching a usage tier's spend cap carries no `retry-after` header and keeps failing until access resumes; the official SDKs automatically retry transient failures such as connection errors, rate limits and 5xx server errors with exponential backoff, twice by default, honouring the `retry-after` header when present, and each SDK client accepts a maximum-retries option to configure or disable that behaviour; every API response carries a unique `request-id` header, and the same identifier appears as the `request_id` field in error response bodies for tracking and debugging. Anthropic, *Claude API errors* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/errors](https://platform.claude.com/docs/en/api/errors)

2. Claude Code retries transient failures up to 10 times with exponential backoff before showing an error, covering server errors, overloaded responses, request timeouts, dropped connections and temporary `429` throttles; a request rejected because the input plus `max_tokens` exceeds the context limit would fail the same way if re-sent unchanged, so Claude Code retries with a reduced `max_tokens` instead; it does not retry a TLS certificate validation failure, nor a server error, dropped connection or stalled stream arriving after Claude has completed a block of text or a tool call, because re-running the request could execute the same tool calls twice, so it keeps what Claude completed, shows an incomplete-response notice and continues the turn from the results of any tool calls Claude completed. Anthropic, *Error reference* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/errors](https://code.claude.com/docs/en/errors)

3. With enhanced telemetry enabled, each step of the agent loop becomes a span: `claude_code.interaction` wraps a single turn from prompt to response, `claude_code.llm_request` wraps each call to the Claude API with model name, latency and token counts as attributes, `claude_code.tool` wraps each tool invocation with child spans for the permission wait and the execution itself, and `claude_code.hook` wraps each hook execution; when the agent spawns a subagent, the subagent's `llm_request` and `tool` spans nest under the parent agent's `claude_code.tool` span so the full delegation chain appears as one trace; telemetry is structural by default, recording durations, model names and tool names on every span, with token counts recorded when the underlying API request returns usage data, while the content the agent reads and writes is not recorded unless opt-in variables are set, which the documentation advises leaving unset unless the observability pipeline is approved to store that data; the CLI fails silently on export errors by default, so an unreachable or rejecting endpoint leaves the agent running normally while the telemetry is dropped with no error surfaced in the application. Anthropic, *Observability with OpenTelemetry* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/observability](https://code.claude.com/docs/en/agent-sdk/observability)

4. The `/context` command shows everything occupying the context window for the current session, including memory files and skills, and is the first check on whether a `CLAUDE.md` file, rule or skill description is present at all; if `/context` confirms the file loaded but a particular instruction is still not followed, the issue is likely how the instruction is written rather than whether it loaded; settings merge across managed, user, project and local scopes with command-line flags and environment variables acting as a further override layer, so when a setting does not seem to apply the value set is usually being overridden by another scope or an environment variable. Anthropic, *Debug your configuration* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/debug-your-config](https://code.claude.com/docs/en/debug-your-config)

5. A `tool_result` block carries an optional `is_error` field, set to true when the tool execution resulted in an error, and Claude uses the returned tool result to continue generating its response to the original prompt. Anthropic, *Handle tool calls* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls)

6. Anthropic's guidance on building evaluations for agents states that 20 to 50 simple tasks drawn from real failures is a good start, that both the cases where a behaviour should occur and the cases where it should not should be tested, that a good task is one where two domain experts would independently reach the same pass or fail verdict, that ambiguity in task specifications becomes noise in metrics, that model-based rubrics should be frequently calibrated against expert human judgement, and that graders cannot be known to work well without reading the transcripts and grades from many trials; it distinguishes measuring whether at least one of several attempts produces a correct result from measuring whether all of them do. Anthropic, *Demystifying evals for AI agents* (Anthropic Engineering, accessed 2026-09-09), [https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
