# Chapter 2: Four Parts to Every Call, and Three Places It Breaks

**Summary**: *Every request to Claude carries the same four parts, and every failure worth debugging sits in one of three seams between them. Hold the seams apart and a whole family of scenarios answers itself: which failures deserve another attempt, which need repairing before the call leaves the building, and which arrive wearing a perfectly healthy status code. Muddle them and an afternoon disappears into adding backoff to a malformed request, or into a network ticket about an answer the model gave on purpose.*

## Four Parts Go Out on Every Call, and the Fourth One Is a Contract

**Write the call out by hand** once, in a scratch file nobody keeps. Not because it ships that way. Everything a client library does afterwards is one of these **four parts**, and a failure inside a layer nobody has looked at gets **misattributed to the nearest layer** somebody has.

Claude is reached over a **REST API** at `https://api.anthropic.com`, and a message goes to **`POST /v1/messages`**.<sup>[1]</sup> That is the **address**. Then **three headers, none of them optional**: a **credential**, which is either an **`x-api-key`** value or an **`Authorization` bearer token**; an **`anthropic-version`** value such as `2023-06-01`; and **`content-type: application/json`**.<sup>[1]</sup> Then a JSON body. Then a response, which is where developers stop paying attention and where the interesting failures live.

Call that shape the **four-part call**: an **address**, headers that authenticate and pin a version, a body, and a response you have to **interpret rather than read**.

| Part | What it carries | What its failure looks like |
|---|---|---|
| Address | Endpoint and HTTP method | A `404`, on a path that looks right |
| Headers | Credential, API version, content type | A `401` or a `403`, identical on every attempt |
| Body | The JSON payload of the request | A `400`, or a `413` past 32 MB |
| Response | Status code, headers, JSON body | Nothing at all, until something downstream falls over |

The last row is the one worth staring at. A **status code** answers whether the **exchange completed**. The **body answers** what **came back**. Those are **two separate questions**, and code that checks the first and assumes the second is the **most common defect** on this material.

## The Client Library Is Somebody Else's Maintained Copy of That Boilerplate

Anthropic publishes **official clients for seven languages**, Python, TypeScript, C#, Go, Java, PHP and Ruby, alongside an **`ant` command-line tool** aimed at **shell scripting** and interactive use.<sup>[2]</sup> A client sends the required headers itself, so the version pin and the **credential** stop being something anybody has to remember.<sup>[1]</sup>

The library and a hand-written HTTP call reach the **same endpoint and the same model**. Nothing about the **answer improves**. What changes is **who owns the parts**, and the parts are duller and more numerous than they look from outside:

Errors arrive as **typed exceptions rather than raw JSON**, and the documentation is explicit that **catching those classes** beats matching on the text of a message, **most specific class first**.<sup>[3]</sup> A **retry policy** is already there. List endpoints come with an **iterator that follows the page cursor**.<sup>[1]</sup> Non-streaming message requests are validated against a **ten-minute ceiling** before they are sent, and the socket is configured for **TCP keep-alive** so an idle network path is less likely to drop underneath the call.<sup>[3]</sup> The **`request-id`** on every response is exposed as a property rather than left in a header bag.<sup>[3]</sup>

None of that is hard. All of it is **maintenance**, and it **accrues to whoever wrote** it.

The **hand-rolled client** is the first named anti-pattern here, and it is seductive precisely because the **first version works**. A **general HTTP library**, three headers typed out, a JSON body, done in twenty minutes. What that produces is a **private fork** of a library nobody wanted to own. It now has to **track the API by hand**, forever, at the priority such work actually gets.

The **foreign client plus a translation shim** is the second, and it usually arrives **dressed as portability**. Anthropic does publish libraries that expose Claude through another framework's surface, including a path that speaks the OpenAI SDK's shape, and it says plainly that these are not **general-purpose Messages API clients**.<sup>[2]</sup> A shim buys a second API's semantics and **one more layer to debug** through, in exchange for a provider switch nobody has scheduled.

The **subprocess call** is the third. The command-line tool is documented for **shell scripting and interactive use**.<sup>[2]</sup> A service that shells out to it once per request has traded **typed errors and connection reuse** for a process boundary it now pays for on every call.

## Retries Default to Two Attempts, and Two Is a Setting Rather Than a Law

Ask what happens when a call fails for a reason that has nothing to do with the request, and the documented answer is specific. The official clients **retry transient failures** with **exponential backoff**, **twice by default**, honouring a **`retry-after` header** when the service sends one, and every client exposes a **maximum-retries option** that **adjusts or switches off** that behaviour.<sup>[3]</sup> The retried set is named too: **connection errors of any kind**, `408`, `409`, `429`, and anything at or above 500.<sup>[4]</sup>

Both **halves of that sentence carry weight**, and scenarios on this material almost always turn on one of them.

| Claim about the library's retries | Verdict |
|---|---|
| Nothing happens unless application code does it | Wrong. A default policy ships and runs |
| Something happens and nothing can change it | Wrong. Maximum retries is a client option, and zero disables it |
| Connection faults retry, throttling passes through | Wrong. Throttling is in the retried set, and it is the common case |
| A default exists and a setting adjusts it | Correct, and the only description that fits a client library |

The useful thing to tell a teammate lands between the two obvious answers. A **policy is already running**, whether or not anybody chose it, and a team on a **tight latency budget** can **turn it down** or off in one line.

A **retry loop wrapped around a retry loop** is the anti-pattern, and it is pure arithmetic. Three attempts of your own around a default of two gives **nine trips** to a service that has already asked for a pause. Nobody writes that deliberately. It happens because the **outer loop** was written by somebody who assumed the **inner one did not exist**.

Two more traps sit close by. A **timeout set across a whole request lifecycle** does not **restart** when the library retries,<sup>[4]</sup> so a generous outer deadline can expire in the middle of a policy that was about to succeed. And not every `429` is transient. One raised by a **usage tier's spend cap** carries no **`retry-after` header** and keeps failing until access is restored,<sup>[3]</sup> so an **unbounded retry loop** against it turns a **billing problem into an outage**.

## Persistent and Two-Way Are Separate Properties, and Only One Needs a WebSocket

Teams reach for a **websocket** when somebody says the words "real time", which is the **wrong trigger**. Two **properties decide the shape**, and they are independent: how long the **relationship lasts**, and how many **directions carry messages** that nobody asked for.

| Shape | Who can send unprompted | What it costs | When it fits |
|---|---|---|---|
| Request and response | Neither end | One connection setup per exchange | A question with one answer |
| Repeated polling | Neither end | Request volume, and staleness bounded by the interval | Cheap checks on something that changes slowly |
| Held-open one-way response | Server only | A connection tied up for the length of the response | Output arriving in pieces while a human waits |
| WebSocket | Both ends | A connection to keep alive, reconnect and monitor | A long-lived exchange where both ends push |

Read down the **first column** and the decision falls out. **Updates arrive as they happen**, from the other end, **unasked**. Messages go back over the **same connection**. The relationship is measured in **minutes or hours rather than milliseconds**. That combination is what a websocket was built for, and every substitute reintroduces something the combination was meant to remove.

Polling is the substitute people accept without noticing. **Shortening the interval** buys **freshness with request volume**, and the interval never reaches zero, so the data is always as old as the gap between checks. **Polling relabelled as real time** is the anti-pattern, and the tell in a scenario is a proposal that gets better when a **number gets smaller**.

The other substitute is subtler. An ordinary **HTTP response held open** carries **data one way**, from the server to you. Claude's own streaming works exactly like this, over **server-sent events**,<sup>[5]</sup> and chapter 3 covers what that means for consuming a message. It is the right shape for **output arriving progressively**. It is not a channel, because your only way to say anything back is to **open a fresh request**, which is the cost the persistent connection was supposed to remove. The **held-open response treated as a two-way channel** is worth naming for that reason.

Anthropic's own products use the protocol where the properties call for it. An **MCP tunnel** connects Claude to servers inside a **private network** over an **outbound-only connection**, so no **inbound firewall port** has to be opened and no origin gets exposed publicly.<sup>[6]</sup> The encrypted handshake to the customer's proxy is then carried inside that **tunnel's WebSocket stream**.<sup>[7]</sup> Which **end opened the connection** and which end sends over it are **two different facts**, and confusing them is how a security review ends up asking for a **firewall change nobody needs**.

## Three Seams, and Only the Middle One Deserves Another Attempt

Every failure in the **four-part call** lands in one of **three seams**. **Naming the seam first** is the whole **diagnostic skill**, because each one responds to a different repair and each one gets worse under the repair that belongs to a neighbour.

| Seam | How it announces itself | What repairs it | What makes it worse |
|---|---|---|---|
| Integration | `400`, `401`, `403`, `404`, 413. Identical on every attempt<sup>[3]</sup> | Fix the request, the credential or the size | Backoff, which pays repeatedly for the same answer |
| Transport | Connection faults, `429`, `500`, 529. Intermittent, clustering under load<sup>[3]</sup> | Bounded retries with backoff, honouring `retry-after` | A tighter loop, and no ceiling on attempts |
| Output | HTTP `200`, and the content is unusable | Validation, a schema, a fallback model | Reading the status code and stopping there |

The **integration seam** belongs **entirely to the caller**. Its signature is **determinism**. The same bytes fail the same way at three in the morning as they did in the test. A `413` past the **32 MB request ceiling** will never succeed on the fourth attempt.<sup>[3]</sup>

The **transport seam** is what the library's retry policy exists for, and it is the only seam where another attempt is a **reasoned act rather than a hope**. A **`529`** reports the service **temporarily overloaded**. A `500` carries the documented instruction to retry with **exponential backoff**.<sup>[3]</sup>

The **output seam** is where developers get hurt, because **everything above it worked**. A **refusal arrives as a successful `200`** response carrying a **`stop_reason` of `refusal`**. The usual repair is sending the same request to a **different model** rather than treating the response as an error.<sup>[8]</sup> With a streamed response the situation is sharper still: an error can occur after the API has **already returned a `200`**, so it does not surface through the ordinary error path at all.<sup>[3]</sup> Humans reading a log line **cannot tell a refusal from an answer** unless somebody wrote code to record which one it was.

One habit closes this out, and it costs nothing. Every **response carries** a **`request-id`**, and support asks for it because it is the only **handle that identifies a single call**.<sup>[1]</sup> **Log it on failure** and a report stops being a description of a bad afternoon.

## Eleven Documented Codes, and the Seam Each One Belongs To

The **seam** is the thing to carry into a question. The **codes are a lookup**, and a lookup you **half remember** is worse than none at all, because it **sorts confidently** into the **wrong seam** and then applies that seam's repair.<sup>[3]</sup>

Seven of them say the **request, the credential or the account** is wrong, and another attempt changes nothing about any of them.

| Code | Error type | What it means |
|:----|:------------------|:--------------------------|
| `400` | `invalid_request_error` | The format or content of the request, and the catch-all for 4XX codes with no entry of their own |
| `401` | `authentication_error` | The key is malformed, revoked or expired |
| `402` | `billing_error` | Billing or payment details need attention |
| `403` | `permission_error` | The key is genuine and is not entitled to this resource |
| `404` | `not_found_error` | The endpoint path, or a resource id inside the URL |
| `409` | `conflict_error` | A concurrent modification, or a unique value already taken |
| `413` | `request_too_large` | Past the byte ceiling for that endpoint |

Four say the **service**, or the **pace you are calling** it at, and **waiting** is part of every repair.

| Code | Error type | What it means |
|:----|:------------------|:--------------------------|
| `429` | `rate_limit_error` | A rate limit, a monthly spend cap, or a workspace spend limit |
| `500` | `api_error` | A fault inside Anthropic's own systems |
| `504` | `timeout_error` | The request timed out while being processed |
| `529` | `overloaded_error` | The service is temporarily overloaded |

Three of the eleven repay a **second reading**.

A `400` is a **catch-all rather than a diagnosis**. It covers **4XX codes** that have no entry of their own, and a **spend limit** you set on your own organisation or workspace returns one, except on a **Claude Code workspace**, which can answer `429` instead.<sup>[3]</sup> A `400` is therefore no **proof that a body is malformed**, and a scenario pairing one with a spend limit is not describing a broken request.

`401`, `402` and `403` all say the **caller may not proceed**, for **three unrelated reasons**. The **key is bad**, the account owes money, or the key is perfectly good and **lacks this one permission**. Three repairs, and not one of them is another attempt.

`409` is the **exception inside the first table**. The client **retries connection errors**, `408`, `409`, `429` and `500` or above by default,<sup>[4]</sup> which reads as a contradiction until you read what a **conflict** is: a state some other **writer may already have let go** of. Retrying it is the documented repair.<sup>[3]</sup> Every other code in that table fails the same way forever.

## How Wrong Answers Are Built on This Material

Wrong answers on **integration mechanics** almost never describe something impossible. They describe a **real technique**, correctly, **applied one seam over** from where it belongs. Three assumptions produce most of them.

- A **library is either absent or immovable**. **Options split** into "your **code must do this itself**" and "the **library does this** and you cannot change it", and both miss the **shape a client library** actually has, which is a **documented default with a knob** on it. Whenever a scenario asks what happens to **transient failures**, the answer that survives says a **default runs** and **names it as configurable**.
- **Freshness bought with request volume**, **sold as a connection**. An option that keeps asking is offered against a **requirement to be told**. It is a genuinely fine design for **slow-moving data** and it is the **wrong shape** when the **other end knows first**, because **latency stays pinned** to whatever **interval somebody picked**.
- A **status code** read as a **verdict on the content**. **Options** that **treat `200` as success** and **everything else as failure** ignore a **whole seam**, and the scenarios that punish this are the ones where the **pipeline broke downstream** of a call that **never errored**.
- A **heavier integration preferred to the maintained one**. Building against a **general HTTP client**, wrapping a **different provider's library**, or **shelling out to a process** all look like **engineering judgement**. Ask what each one now has to **track by hand** as the API moves.

**Match** on what the **scenario** is doing rather than on the **domain** it is set in.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| A service pushes updates continuously and the app also sends messages back | A protocol built for a persistent two-way connection | Any option whose freshness depends on how often you ask |
| A teammate asks whether application code needs its own retry logic | A default policy exists and the client exposes it as a setting | Calling it absent, or correct but unchangeable |
| The language has an official Claude client and the team is starting an integration | Use the client and its documented pattern | Rebuilding headers, retries and parsing against a general HTTP library |
| A call fails the same way on every attempt | The request, the credential or the size is wrong, and it is yours | Adding backoff to a deterministic failure |
| Failures are intermittent and get worse under load | Bounded retries with backoff, honouring the interval the service names | A tighter loop, or retrying past a spend cap forever |
| Nothing errored and a downstream step broke anyway | Inspect the response body and its stop reason, then validate | Treating a healthy status code as a healthy answer |
| A status code arrives and the option sorts it by the number alone | Ask whether a later attempt could change the outcome, then repair the seam that owns it | Treating `400` as proof of a malformed body, or `409` as something retrying cannot clear |
| Portability across model providers is offered as the reason | Cost the extra layer against a switch nobody has scheduled | A foreign client plus a translation shim |

The transferable habit is to **name the seam before naming the fix**. Say out loud whether the failure is in what you **sent**, in getting it there, or in what **came back**, and roughly half the options in front of you **stop being plausible**.

## What to Remember

- The **four-part call** is **address, headers, body, response**: an endpoint and method, a **credential plus `anthropic-version` plus content type**, a JSON payload, and a reply that carries a **status code, headers and a body** you have to interpret separately.
- **Official clients exist for seven languages** and send the **required headers** for you. They reach the **same endpoint and the same model** as a hand-written call; what differs is who **maintains the parts** as the API moves.
- **Clients retry transient failures twice by default**, with **exponential backoff**, honouring **`retry-after`** when present, and **maximum retries is an option** you can raise, lower or set to zero. A default that is **present and adjustable** is the description that fits.
- **Retried by default**: **connection errors, `408`, `409`, `429`**, and **`500` or above**. A **retry loop around the client's own** multiplies attempts, and an **outer deadline does not restart** when the library retries.
- A **`429` from a spend cap** carries no **`retry-after`** and keeps failing until access resumes, so **retrying it indefinitely** converts a **billing problem into an outage**.
- **Eleven documented codes**: `400` invalid request, `401` authentication, `402` billing, `403` permission, `404` not found, `409` conflict, `413` too large, `429` rate limit, `500` internal, `504` timeout, `529` overloaded. **`409`** is the only one **shaped like an integration failure** that the client **retries by default**, because the **conflicting state can clear**.
- **Persistent and two-way are separate properties**. **Polling** leaves staleness **pinned to the interval**; a **held-open response pushes one way only**, server to client; a **websocket keeps one connection** on which **both ends send unprompted**.
- **Three seams break a call**: **integration** (4xx, identical every attempt, yours to fix), **transport** (connection faults and 5xx, intermittent, retry with backoff), and **output** (**HTTP `200` with unusable content**, repaired by validation or a different model).
- A **refusal is a successful `200`** carrying **`stop_reason: refusal`**, and a **streamed error can arrive after the `200`**, which is why **status-code-only error handling misses an entire seam**.
- **Every response carries** a **`request-id`**. **Log it on failure**, because it is the **only identifier that names a single call** when you report one.

## Endnotes for this chapter

1. The Claude API is a RESTful API at `https://api.anthropic.com`; all requests must include a credential header (`x-api-key` or `Authorization`), an `anthropic-version` header such as `2023-06-01`, and `content-type: application/json`, which the client SDKs send automatically. The page also documents the `request-id` response header and the auto-paginating iterators the SDKs provide. Anthropic, *API overview* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/overview](https://platform.claude.com/docs/en/api/overview)

2. Anthropic publishes an `ant` command-line tool for shell scripting and interactive use, client SDKs in seven languages, and framework-specific libraries which expose Claude through another framework's API surface and are not general-purpose Messages API clients. Anthropic, *CLI, SDKs, and libraries* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/cli-sdks-libraries/overview](https://platform.claude.com/docs/en/cli-sdks-libraries/overview)

3. HTTP status codes and their error types, the 32 MB request size limit, the instruction to retry a `500` with exponential backoff, the note that a spend-cap `429` carries no `retry-after` header, the default of two automatic retries with exponential backoff and a configurable maximum-retries option, typed SDK exceptions, the ten-minute validation and TCP keep-alive behaviour, the `request-id` header, and the fact that an error can occur after a streaming response has already returned a 200. Anthropic, *Claude API errors* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/errors](https://platform.claude.com/docs/en/api/errors)

4. The set of errors retried by default is all connection errors, `408` Request Timeout, `409` Conflict, `429` Rate Limit and `500` or above, with a default of two retries configurable through the client's maximum-retries option; a retried request does not restart a context timeout set for the request lifecycle. Anthropic, *Go SDK* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/go](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/go)

5. Setting `"stream": true` on a message returns the response incrementally using server-sent events. Anthropic, *Streaming messages* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/streaming](https://platform.claude.com/docs/en/build-with-claude/streaming)

6. MCP tunnels connect Claude to MCP servers running inside a private network over an outbound-only connection, removing the need to open inbound firewall ports or expose services to the public internet. Anthropic, *MCP tunnels* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/overview](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/overview)

7. The inner TLS handshake between Anthropic's backend and the customer's proxy is carried inside the tunnel's plaintext WebSocket stream, and the term "outbound-only" describes the connection rather than the requests carried over it. Anthropic, *Architecture and components* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/concepts](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/concepts)

8. A refusal is a successful HTTP `200` response carrying `stop_reason: "refusal"` rather than an error, and the same request can usually be sent to another Claude model to get an answer. Anthropic, *Refusals and fallback* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)
