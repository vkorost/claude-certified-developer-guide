# Chapter 3: Messages, Streaming, Batches: Who Is Waiting Decides

**Summary**: *Three requests can carry the same model, the same prompt and the same document and still belong on three different endpoints. What separates them is who is holding still until the answer lands. Read that wrong and you pay full price for work nobody was watching, or you leave somebody staring at an empty panel while a perfectly good response assembles itself out of sight. This chapter is the API surface arranged around that question, from the shape of one message to the shape of forty thousand.*

## Who Is Waiting Is Established Before Anything Else and Changed Last

Three requests. Identical model, identical instructions, identical attached document.

One was typed by somebody **watching a cursor blink**. One came from a service that **owes an HTTP response** inside **two seconds**. One is number 8,431 in an **overnight run** of forty thousand, and the only **deadline** is a report due at eight in the morning.

Nothing in the **prompt distinguishes** them. Everything about the **delivery** does.

Call the answer the **waiting party**: the **entity whose clock the request sits** on. Sometimes that is a human watching a screen. Sometimes it is a **caller blocked** on a return value. Sometimes it is nobody at all until a **deadline hours out**. **Establish it once**, early, and **four decisions fall out** together. Establish it late and you get a **retrofit**, because the endpoint, the error handling, the cost model and the interface all bend around it.

| Waiting party | Shape | What it buys | What it costs |
|---|---|---|---|
| A human watching output appear | A streamed message | Visible progress within a second | Event handling, and partial state to reassemble |
| A caller blocked on the return value | One ordinary message request | A single complete object to parse | The caller waits out the whole generation |
| A caller doing other work meanwhile | Many ordinary requests, issued concurrently | Throughput without a queue | Your own rate limits, hit at your own pace |
| Nobody, until a deadline hours away | A batch submission | Half the token price, higher throughput | Turnaround measured in hours, not seconds |

Read the third and fourth rows against each other, because they are the pair that scenarios pull apart. Both are described as **asynchronous** and they solve unrelated problems. **Non-blocking calls keep your process useful** while a request is in flight; every response still **arrives at conversational speed**. A **batch gives up that speed** entirely and takes a discount for it.

## One Message Request Is a Model, a Cap, and a List of Turns

The body is smaller than people expect. A **model identifier**, a **ceiling** on how long the response may run, and **`messages`**: an **ordered list of turns**, each one a `role` and a `content` array. **Standing instructions** go in a **separate top-level `system` field** rather than into any turn.<sup>[1]</sup>

**Content is a list of typed blocks** rather than a string. The **block type is the extension point**. That one choice is why a **single endpoint absorbs** text, images, documents, tool calls and reasoning without ever needing a sibling. A response comes back the same way: an `id`, `role: "assistant"`, a `content` array of blocks, a **`stop_reason`**, and a **`usage` object** counting the tokens in and out.<sup>[1]</sup>

```
POST /v1/messages
{
  "model":      "<model id>",
  "max_tokens": <ceiling on the response>,
  "system":     "<standing instructions, optional>",
  "messages":   [ { "role": "user", "content": [ <blocks> ] } ]
}
```

Now the property that governs everything downstream. The **Messages API is stateless**, so the **full conversational history** goes up on **every call**.<sup>[1]</sup> **No handle identifies a conversation**. A **multi-turn exchange** is a list you keep and grow: append the assistant turn you got back, append the next user turn, send the whole thing again.

**Two consequences** arrive with that, and both are **examinable**.

**Earlier turns** do not have to have come from Claude. **Synthetic assistant turns** are a documented technique, and you can **prefill the beginning of the answer** by putting **partial assistant content** in the final position of the list.<sup>[1]</sup> Chapter 5 owns what to put in those turns.

And the **history is the bill**. **Every turn you carry** is **input tokens** again on the next call. A conversation therefore gets **more expensive per exchange** as it lengthens, even though the questions stay the same size. Chapter 7 owns what to do about a window filling up and chapter 8 owns the arithmetic.

One refinement to the `system` field is worth knowing because it exists for a **caching reason**. On the newer models a message with `"role": "system"` may **appear after a user turn**, carrying the same authority as the top-level field. It **cannot come first**. Because it lands at the **end of the history** rather than the front, it adds an instruction without **invalidating a cached prefix** that was already there.<sup>[1]</sup>

## A Tool Result Is Paired by Identifier, Not by Position or by Narration

The strictest case is the **tool round trip**. Two independent systems have to agree about which **answer belongs to which question**, and only one of them can read English.

Claude returns **`stop_reason: "tool_use"`** and one or more **`tool_use` blocks**. Each carries an `id`, a `name`, and an `input` object matching the tool's schema. You run the tool. You continue by sending a **new user turn of `tool_result` blocks**, each **echoing the `tool_use_id`** of the call it answers, with the result in `content` and `is_error` set if the tool failed.<sup>[2]</sup>

Three rules ride along with that and each one has teeth. That user message may carry the **result blocks and nothing else**. Adding a sentence of your own after them **ends the assistant turn**, and can fail the request outright. The **`tools` array has to stay the same** across the continuation. And when the response carried a **thinking block**, the assistant content **goes back exactly as it arrived**, signature intact, so the reasoning stays attached to the call it produced.<sup>[3]</sup> **Extra input for Claude** goes in a separate user message once the turn has closed.<sup>[2]</sup>

**Two named anti-patterns** live here and both look **reasonable** in a code review.

The **result written as prose** takes what the tool returned, phrases it as an English sentence, and sends it as an ordinary user message. It reads fine to a human. It has **discarded the identifier**, so **nothing structurally connects** the answer to the call, and with **two tools in flight** nothing says which is which.

The **result filed in the assistant turn** puts the tool output into an assistant message, on the reasoning that the model asked for it so the model's side owns it. The **assistant turn is the record** of what Claude produced. Externally computed content placed there is an **assertion that Claude produced** it, which is false, and it leaves the original **`tool_use` block still waiting**.

Chapter 9 owns how tools are **described and chosen**. This is only the **wiring**.

## `stop_reason` Says What You Owe the Caller Next

Every successful response carries a **`stop_reason`**, and it is the difference between a message you can hand on and a **message that is a fragment**.<sup>[4]</sup> Group the values by the obligation they create rather than by what they are called, because the **obligation is what a scenario** is really asking about.

| What the response obliges you to do | Values that put you there | The move |
|---|---|---|
| Nothing. The turn is complete | `end_turn`, `stop_sequence` | Use the content. Read `stop_sequence` to see which sequence fired |
| Construct a continuation and send it | `tool_use`, `pause_turn` | Return `tool_result` blocks for `tool_use`; for `pause_turn`, send the response back as it stands so the server-side loop resumes |
| Tell the reader the answer is cut short | `max_tokens`, `model_context_window_exceeded` | Mark the output incomplete, or continue generation deliberately |
| Route the request somewhere else | `refusal` | Retry on another Claude model; `stop_details` names the policy category |

**Four obligations, seven values**. Two of them repay a closer look.

`pause_turn` and `tool_use` both mean the **turn is unfinished** and they are continued differently, which is the trap. `pause_turn` comes back when the **server-side loop hits its iteration limit** while running platform tools, and the continuation is the **response itself, unchanged**. `tool_use` means **Claude is waiting** on you, and the continuation is your **results**.<sup>[4]</sup>

**`refusal`** is the one that **never announces itself as trouble**. It arrives as an **ordinary HTTP `200`** from a **safety classifier**. The documented handling is to retry the same request on a **different Claude model**.<sup>[4]</sup> Chapter 2 made the general version of this point about the seam between a **healthy status code** and a usable body. This is the field that closes it.

The **status code read as the verdict** stays the anti-pattern, and **`stop_reason`** is the specific repair. A pipeline that checks for `200` and parses the content will **forward a truncated document**, an **unresolved tool call**, and a refusal. All three look like ordinary short answers in a log. Nothing in the log says otherwise.

## Streaming Changes When the Words Appear, Not What They Cost

Set **`"stream": true`** and the response comes back incrementally over **server-sent events**.<sup>[5]</sup> Every event carries a named type and a JSON payload. The order is regular. You can write a **state machine** against it. A **`message_start` arrives first**, holding the message shell with empty content. Each content block then opens with a **`content_block_start`**, fills through a run of **`content_block_delta`** events, and closes with a **`content_block_stop`**. Top-level changes arrive in **`message_delta`**. Then **`message_stop`**. Keep-alive **`ping` events** appear anywhere.<sup>[5]</sup>

**Three facts** about that stream matter more than the **event names**.

**`stop_reason` is `null` in `message_start`** and arrives later, in a **`message_delta`**.<sup>[4]</sup> Code that reads the opening event and moves on has read the field before it was populated.

**Errors travel inside the stream**. During heavy load an **`overloaded_error`** can arrive as an `error` event on a connection that **already returned HTTP `200`**, where a **non-streamed request would have failed** with a 529.<sup>[5]</sup> A consumer that only handles text deltas will treat that as the end of a short answer.

New event types may be added under the **versioning policy**, so **unknown types are skipped** rather than fatal.<sup>[5]</sup>

There is also a use for streaming that has **nothing to do with anybody watching**. The **SDKs can stream internally** and hand back the same complete message object an ordinary call returns. They require it for requests with a **large `max_tokens` value**. The reason is mundane. A long generation on one non-streamed call **runs into HTTP timeouts**.<sup>[5]</sup> That is transport, wearing the same parameter.

**Streaming adopted as a throughput fix** is the named anti-pattern. The words start appearing sooner. The total **generation time does not shrink**. The token cost is unchanged. A backend job that assembles the whole message before acting has **gained event-handling code** and nothing else. Streaming answers one requirement, and the requirement is that a human is watching.

## Images and Documents Arrive as Blocks, With Three Ways to Name the Bytes

Vision is not a separate endpoint. An **`image` content block** sits in a user turn beside the text. The bytes are named one of three ways: inline **`base64`**, a **`url`** pointing at a hosted image, or a **`file_id`** returned by the Files API. **JPEG, PNG, GIF and WebP** are accepted. No image may **exceed 8000 by 8000 pixels**.<sup>[6]</sup>

Several images in one request are **analysed together**, which is what makes **page-by-page document work** possible. **Label each one** in the accompanying text so you can refer to it later. In a **multi-turn conversation** Claude keeps access to images from earlier turns, so a follow-up question about the first two does not **require resending** them.<sup>[6]</sup>

One limit is easy to trip and hard to diagnose. **Past 20 image blocks** in a single request, a **tighter per-image dimension rule** applies to every image in that request, counting images you **resent from earlier turns** and images nested inside tool results. Blocks over the tighter limit are **rejected with a validation error** rather than being scaled down.<sup>[6]</sup>

**PDFs enter as `document` blocks** and take the same three routes. The cost model is what to carry away. Each page is rendered as an image as well as read as text, so a **PDF is billed both ways**.<sup>[7]</sup> **Page count drives the total**, not file size. And both the size ceiling and the page ceiling apply to the **whole request payload** rather than to the document alone.<sup>[7]</sup>

Which is where the **Files API** earns its place. **Upload once**, get a **`file_id`**, reference that identifier in as many later requests as you like instead of **re-encoding the content** each time.<sup>[8]</sup> **Base64 resent every turn** is the anti-pattern it fixes, and it is invisible until a conversation about one **scanned contract** has carried that contract up the wire on **every single turn**.

**Availability** is worth a glance before you design around it. The **Files API** is offered on the **first-party API** and, in **beta**, on some of the **cloud paths**, and not on all of them.<sup>[9]</sup>

## Batches Take Half the Price and Hand Back the Correlation Problem

The **Message Batches API** is where the fourth row of the opening table lives. Requests are submitted together, processed asynchronously, and **charged at 50% of standard prices**. Most batches finish inside an hour. Any batch **unfinished at 24 hours expires**.<sup>[10]</sup>

The pattern is **four steps** and the last two are **yours**.

```
create   POST a "requests" list; each entry is a custom_id plus a params
         object holding an ordinary Messages request
poll     the batch carries processing_status: in_progress, then ended
collect  fetch results_url; JSONL, one result object per line
match    join each line back to your work by its custom_id
```

**Results come back out of order** and are matched by **`custom_id`, never by position**.<sup>[10]</sup> That identifier is yours to choose, up to 64 characters of letters, digits, hyphens and underscores. Choosing one that already means something in your own system is the difference between a **join and a guess**.

Each result is one of **four types**, and the distinction is billing as much as control flow. **`succeeded` carries the message**. **`errored`, `canceled` and `expired`** are **not billed** at all.<sup>[10]</sup> The batch's **`request_counts`** totals them. **One request failing** leaves the rest of the batch alone.<sup>[10]</sup> Results stay **retrievable for 29 days** from creation, and a batch can be **cancelled mid-flight**, in which case it still ends and may carry partial results.<sup>[10]</sup>

A short list of **Messages parameters** is **rejected in a batch**, and each refusal says something true about the shape. **`stream` is refused** because results are collected as a file. Fast mode is refused because it **tunes synchronous latency**, which no longer exists here. **Stateful thread parameters** are refused because a batch request is not part of a thread.<sup>[10]</sup>

Prompt caching still applies and **stacks with the batch discount**, but because batch requests run concurrently the **cache hits are best effort** rather than guaranteed.<sup>[10]</sup> Design the saving as a bonus and not as a floor.

**Two anti-patterns**, and they are mirror images.

**Batch chosen for latency** treats "asynchronous" as a synonym for "faster". Nothing about a batch **returns any individual answer sooner**. If a caller is blocked on one result, batching it has made the wait worse and saved half of a rounding error. The **discount needs volume** to mean anything.

A **synchronous loop run at batch volume** is the commoner failure and it is what an overnight job usually starts life as. **Tens of thousands of sequential calls** will meet your rate limits, occupy the process for the duration, and cost twice what the same work costs submitted together. The requirement that selects a batch is **high volume** plus **indifference** to when any single answer lands, and **both halves have to hold**.

## Third-Party Providers Keep the Message Shape and Change the Frame Around It

Enterprises route Claude through **Amazon Bedrock**, **Google Cloud**, **Microsoft Foundry** or **Claude Platform on AWS**. The reasons are rarely technical. An existing commercial agreement, a security boundary a compliance team already signed off, a billing relationship somebody wants to keep singular. The constraint arrives as a fact about the organisation, and it **binds from the first call**.

What stays the same is the part you spend your time on. **Content blocks**, **`stop_reason`**, **tool use** and **streaming** carry across, and Anthropic's own client libraries support these platforms directly rather than through a translation layer.<sup>[11]</sup> **Bedrock exposes the same Messages API shape** while running on **AWS-managed infrastructure** inside the AWS security boundary.<sup>[12]</sup>

What changes sits around the message. **Authentication is the platform's**, not an **Anthropic key**. **Endpoints and model identifiers** are the platform's. Billing runs through the platform's marketplace, which is usually why the requirement exists at all.<sup>[13]</sup> On **Google Cloud** the request itself differs in two named ways: the **model is named in the endpoint URL** rather than in the body, and **`anthropic_version` moves into the body** with a platform-specific value.<sup>[11]</sup> **Feature availability is not uniform** either. A capability present on the first-party API may be beta or absent on a given platform, and on Foundry it differs between hosting options of the same platform.<sup>[9]</sup>

That combination has one **sound engineering answer** and two seductive wrong ones.

The answer is to **call the mandated path** from the first commit. Put a **thin seam** between your application logic and the provider, so the parts that differ are named in one place. The **deferred migration** is the first wrong answer: **build against the direct API** now, move later. Every request between now and later **violates the agreement**, and the **rework is guaranteed** rather than merely likely. The **parallel implementation** is the second, and it is worse arithmetic. **Two code paths maintained forever**, and the one that runs in production is still the one the contract prohibits.

## How Wrong Answers Are Built on This Material

Wrong answers on API mechanics are almost never inventions. They name a **real endpoint**, a real parameter and a real technique, and they **aim it at a requirement** it does not serve. **Four assumptions** produce most of them.

- **Asynchronous read as fast**. The option treats **batch submission** as a **performance improvement**. Not blocking reads as a kind of speed. **Batch throughput is high** and the **turnaround on any single answer is worse**. The option becomes correct only when the scenario says the **volume is large** and **names no clock on individual results**.
- The **delivery mechanism swapped for a different mechanism**. A requirement about *when* **output reaches somebody** is answered with **caching**, a **larger token cap**, a **schema**, or a **bigger model**. Each of those is a **real lever on cost, length, shape or capability**. None of them makes a **complete response arrive in pieces**, which is the only thing the requirement asked for.
- A **structural pairing replaced by a description**. Options that **hand a tool result back as prose**, or **file it under the wrong role**, or **leave out the identifier**, all **lose the correlation** the protocol runs on. It reads correctly to a person and the API cannot resolve it. Ask what breaks when **two calls are outstanding at once**.
- A **constraint deferred because it is not technical**. **Contractual routing**, **data residency** and **billing** arrive as requirements from outside engineering and are **treated as schedulable**. They **bind from the first call**. An option that satisfies the constraint later has left **every request in between** unsatisfying it.

Match on what the **scenario** is doing rather than on the **industry** it is dressed in.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Thousands of items, no clock on any single result, cost is named | Asynchronous batch submission, results collected and joined afterwards | A synchronous loop, or a bigger token cap offered as throughput |
| Users should see output as it is produced rather than after a pause | Streaming, so partial content is delivered while generation continues | Caching, a larger cap, or a schema, each answering a different requirement |
| A tool ran and its output has to reach the model | A user turn of result blocks carrying the identifier of the call | Prose in a user message, a result placed in the assistant turn, or standing instructions |
| An answer stopped short and nobody can say why | Read `stop_reason` and act on what it obliges | Treating HTTP `200` as proof the content is complete |
| A contract or a compliance boundary names a cloud provider | Call the mandated path now, behind one provider-specific seam | Building direct and migrating later, or maintaining both paths |
| The same large document is sent on every turn of a conversation | Upload once and reference it by identifier | Re-encoding the bytes into each request |
| Results came back and the ordering does not match what was sent | Join on the identifier you assigned each request | Assuming responses arrive in submission order |

One habit does **more work** than the table. **Name the waiting party out loud** before evaluating any option, then ask whether the option acts on **delivery**, on **cost**, on **shape**, or on **capability**. Most wrong answers on this material are **correct answers** to one of the other three.

## What to Remember

- The **Messages API is stateless**, so **every call carries the whole history**: a **model**, a **`max_tokens` ceiling**, a **`messages` list** of role-and-content turns, and **standing instructions** in a **top-level `system` field** rather than in any turn.
- **Content is a list of typed blocks**, which is why **text, images, documents**, tool calls and **reasoning** all travel through **one endpoint**. A response returns blocks plus **`stop_reason`** plus a **`usage` count**.
- The **waiting party** is whose **clock the request sits** on: a **person watching**, a **caller blocked** on the reply, a **caller doing other work**, or **nobody until a deadline**. It **selects the endpoint** before the prompt is written.
- **Seven `stop_reason` values create four obligations**: **nothing** (`end_turn`, `stop_sequence`), **send a continuation** (`tool_use`, `pause_turn`), **declare** the answer **incomplete** (`max_tokens`, `model_context_window_exceeded`), **retry elsewhere** (`refusal`, which arrives as a **normal HTTP `200`**).
- A **tool result is a user turn** of **`tool_result` blocks**, each **echoing the `tool_use_id`** of the call it answers, **carrying nothing else** in that message, with the **`tools` array unchanged** and any **thinking block echoed back** verbatim.
- **Streaming sends the same message earlier**, over **server-sent events**: **`message_start`**, then block start, deltas and stop, then **`message_delta`**, then **`message_stop`**. **`stop_reason` is null** at the start and **arrives in `message_delta`**, and an **overload error can appear** as an event after HTTP 200.
- **Images and PDFs are content blocks** named three ways: **inline base64**, a **URL**, or a **`file_id`** from the **Files API**. **Past 20 image blocks** a **tighter dimension limit** applies to **every image** in the request, resent ones included.
- A **PDF page is billed** as **text** and as an **image**, because **each page is rendered** as well as read, so **page count drives the cost** more than file size does.
- **Batches run at 50%** of standard prices, mostly finish **within an hour**, and **expire at 24 hours**. Results **arrive out of order** and are **matched by the `custom_id`** you assigned; **`errored`, `canceled` and `expired`** results are **not billed**.
- **`stream` and fast mode are rejected** inside a batch, because a **batch returns a file** and there is no **synchronous latency** left to tune.
- **Non-blocking concurrency is not batching**. **Async clients keep your process busy** while responses still arrive at **conversational speed**; a **batch surrenders** that speed to buy the **discount** and the **throughput**.
- **Third-party providers** keep the **message shape** and change the frame: **authentication, endpoints, model identifiers and billing** all become the platform's, **feature availability varies**, and on **Google Cloud** the **model moves into the URL** while **`anthropic_version`** moves into the body.

## Endnotes for this chapter

1. The Messages API request shape, the stateless requirement to resend full conversational history, synthetic assistant turns, prefilling the last assistant position, the top-level `system` field and the placement and caching behaviour of a mid-conversation system message, the response object with its `content` array, `stop_reason` and `usage`, and the three image source types. Anthropic, *Using the Messages API* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/working-with-messages](https://platform.claude.com/docs/en/build-with-claude/working-with-messages)

2. A `tool_use` block carries `id`, `name` and `input`; the continuation is a user-role message containing `tool_result` blocks that each carry the `tool_use_id` of the call they answer, optional `content`, and an optional `is_error` flag; further input for Claude is sent as a separate user message after the turn completes. Anthropic, *Handle tool calls* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls)

3. The assistant content array is sent back verbatim on the follow-up request so the thinking block, including its signature, stays unchanged alongside the `tool_use` block. Anthropic, *Thinking in tool and multi-turn workflows* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/thinking-tool-workflows](https://platform.claude.com/docs/en/build-with-claude/thinking-tool-workflows)

4. The `stop_reason` values `end_turn`, `max_tokens`, `stop_sequence`, `tool_use`, `pause_turn`, `refusal` and `model_context_window_exceeded`, their meanings and handling; the requirement that the continuation of a `tool_use` response contain only `tool_result` blocks and keep the same `tools` array; the instruction to continue a `pause_turn` by sending the response back as-is; refusals returning as HTTP `200` with a `stop_details` policy category and being retried on another Claude model; and `stop_reason` being null in `message_start` and delivered in `message_delta` when streaming. Anthropic, *Stop reasons and fallback* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons)

5. Setting `"stream": true` returns the response incrementally over server-sent events; the event flow of `message_start`, `content_block_start`, `content_block_delta`, `content_block_stop`, `message_delta` and `message_stop`, with `ping` events interspersed; errors such as `overloaded_error` arriving inside the event stream where a non-streaming request would return HTTP `529`; the instruction to handle unknown event types gracefully; and the SDK facility that streams internally while returning a complete `Message`, which the SDKs require for requests with large `max_tokens` values to avoid HTTP timeouts. Anthropic, *Streaming messages* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/streaming](https://platform.claude.com/docs/en/build-with-claude/streaming)

6. Images are supplied as `image` content blocks using base64, url or file_id source types; supported media types are JPEG, PNG, GIF and WebP; maximum dimensions are 8000 by 8000 pixels; multiple images in one request are analysed jointly and earlier-turn images remain available in later turns; and a request containing more than 20 image blocks applies a stricter per-image dimension limit to every image block in the request, including resent images and images nested inside tool results, with oversized blocks rejected as an `invalid_request_error`. Anthropic, *Vision* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/vision](https://platform.claude.com/docs/en/build-with-claude/vision)

7. PDFs may be provided as a URL, as base64 in document content blocks, or by reference to a file uploaded through the Files API; token cost combines text extracted per page with image token costs because each page is converted into an image; and the size and page limits apply to the entire request payload rather than to the document alone. Anthropic, *PDF support* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/pdf-support](https://platform.claude.com/docs/en/build-with-claude/pdf-support)

8. The Files API provides a create-once, use-many-times pattern: upload a file, receive a `file_id`, and reference that identifier in later Messages requests instead of re-uploading the content. Anthropic, *Files API* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/files](https://platform.claude.com/docs/en/build-with-claude/files)

9. Feature availability is listed per platform across the Claude API, Amazon Bedrock, Claude Platform on AWS, Google Cloud and Microsoft Foundry, with the Files API shown as available on the Claude API and in beta on some platforms; on Microsoft Foundry availability differs between the Hosted on Anthropic and Hosted on Azure options. Anthropic, *Features overview* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/overview](https://platform.claude.com/docs/en/build-with-claude/overview)

10. The Message Batches API processes large volumes of Messages requests asynchronously at 50% of standard prices, with most batches completing in under an hour and a 24-hour expiry; each request carries a `custom_id` matching `^[a-zA-Z0-9_-]{1,64}$` and a `params` object; `processing_status` moves from `in_progress` to `ended`; results are downloaded from `results_url` in JSONL form with order not guaranteed, so `custom_id` is used to match results to requests; the four result types are `succeeded`, `errored`, `canceled` and `expired`, and the last three are not billed; the failure of one request does not affect others; results remain available for 29 days after creation and a processing batch can be cancelled; `stream`, `speed`, thread parameters and routing hints are rejected in batch requests; and prompt caching stacks with the batch discount but cache hits are best-effort. Anthropic, *Batch processing* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/batch-processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing)

11. The API for Claude on Google Cloud's Agent Platform is nearly identical to the Messages API with two differences in request format: `model` is specified in the endpoint URL rather than the request body, and `anthropic_version` is passed in the body with a platform-specific value. Anthropic's official client SDKs support the platform. Anthropic, *Claude on Google Cloud* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/claude-on-vertex-ai](https://platform.claude.com/docs/en/build-with-claude/claude-on-vertex-ai)

12. Claude in Amazon Bedrock runs on AWS-managed infrastructure inside the AWS security boundary while using the same Messages API shape as Anthropic's first-party API, with AWS-native authentication paths. Anthropic, *Claude in Amazon Bedrock* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock)

13. Accessing Claude in Microsoft Foundry bills Claude usage through the Azure Marketplace and manages cost through the Azure subscription, with resources holding security and billing configuration and deployments being the model instances called through the API. Anthropic, *Claude in Microsoft Foundry* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)
