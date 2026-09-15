# Chapter 6: Schema-Valid Is Not the Same as Correct

**Summary**: *A schema tells you the response arrived with the right fields carrying the right types. It says nothing about whether the numbers in those fields are true. Well formed, confidently worded and wrong is a common state, and no validator distinguishes it from well formed and right. This chapter covers the four things belonging between the model and your database: fixing the shape at generation, checking the response on receipt, parsing it so it survives surprises, and verifying content against something the model did not produce.*

## A Response Can Satisfy Every Rule You Wrote and Still Be Wrong

Suppose a reporting pipeline lifts figures out of filings into a ledger. Responses come back as JSON, **every field present**, **every type matching**, the wording around each number calm and specific. Nothing raises anything, for weeks. Then a **reconciliation** turns up several figures that were **plausible, well formed and wrong**.

Go looking for the check that should have caught it. Every check there was **reading form**: the validator asked whether `revenue` was a number, and it was. **Nothing asked** whether it was the **right one**.

That gap is worth naming. Any check you can run answers one of two questions. The **shape question** asks whether the response is addressable: **fields present**, **types right**, values inside the **permitted set**. The **truth question** asks whether those fields **match the world**. Constrained generation, schema validation and defensive parsing answer the first, completely and cheaply. **Not one touches the second**.

**Validity read as verification** is the anti-pattern, and it survives review because the **green check is real**. Something was genuinely verified. It was the **shape**.

## A Prompt Asks for a Shape; a Grammar Removes the Alternatives

Chapter 5 covered asking for a format in the prompt, and that works most of the time. An **instruction holds** on the **inputs you tested** and slips on the one you did not.

Structured outputs close the gap in the API rather than in the wording. You supply a **JSON Schema**, a grammar compiled from it constrains generation, and a response **violating the schema cannot be produced** at all.<sup>[1]</sup> `output_config.format` with `type: "json_schema"` **binds the final response**, `strict: true` on a tool definition **binds tool names** and inputs, and either or both can run in one request.<sup>[1]</sup> Chapter 9 owns the tool half; this chapter owns what comes back.

The difference from a prompt-level request is **not degree**. **Constraining generation removes the failure class**; **hardening the parser** changes only how **loudly the consumer fails**.

Every cost of that guarantee follows from the grammar. Compiling one adds **latency** on the first call with a new schema, so a schema held still is **cheaper** than one **rebuilt per tenant**, and every request then carries an **added system prompt** describing the format. A **schema too complex to compile** is refused outright, and optional fields are most of what makes one complex.<sup>[1]</sup>

**Prefilling the assistant turn** was the old way to force a shape, and **structured outputs** are its documented replacement on **Claude 4.6 and later**.<sup>[2]</sup>

Agents get the same mechanism one level up. The **Agent SDK validates** the result against your schema, **re-prompts on a mismatch**, and gives up with the subtype `error_max_structured_output_retries`.<sup>[3]</sup> A quieter failure sits beside it: a run can **end with subtype `success`** and **no `structured_output`** at all.<sup>[3]</sup>

## Three Documented Ways a Guaranteed Schema Hands You Something Else

**Constrained decoding** guarantees generation, not every response object that reaches the parser. Three documented exceptions arrive as an **ordinary 200**.

| The exception | What the receiving code does |
|---|---|
| A safety refusal, `stop_reason` of `refusal`, arriving as a `200` rather than an error<sup>[4]</sup> | Branch before parsing; on some models, retry the request elsewhere<sup>[4]</sup> |
| The output ceiling reached, `stop_reason` of `max_tokens`, structure cut mid-object<sup>[1]</sup> | Raise the ceiling and retry rather than salvage a fragment<sup>[1]</sup> |
| An enum in different capitalisation, completing with no error and no special `stop_reason`<sup>[1]</sup> | Compare enums case-insensitively; never define two differing only in case<sup>[1]</sup> |

The last teaches the chapter's point twice: **nothing reports anything**, and a value your **code switches on matches no branch**.

## Validation on Receipt Needs a Destination for the Failure

Validating means running the response against the schema in your own process, on arrival, before anything downstream sees it. That sounds redundant, and it is not. Several **SDKs strip constraints** such as `minimum` and `maxLength` before sending, so Claude sees a simplified schema while your **code still enforces the originals**.<sup>[1]</sup> The range rule is enforced on receipt.

Where the failure goes matters more than the check, because one with **no destination** is a crash with extra steps. Retry the call, **reject the record** into a queue somebody works, or **route it to a person**.

Choosing between those needs **categories** rather than a number. A pipeline routing on one **global threshold** has flattened three situations onto one axis: an **optional field genuinely absent**, a **required field too poor to read**, a **document never in scope**. **No cut separates them**. Name the three, exemplify each, and route on the category.

## Defensive Parsing Survives the Unexpected and Still Reports It

Two failures bracket this. **Crashing on anything unfamiliar** is the outage you were asked to prevent. **Dropping whatever fails to conform** is worse: the **pipeline reports success** and the **records stay invisible**. The **silent drop** is the anti-pattern, attractive because it makes the alert stop.

Graceful means both halves. The **malformed case survives**, and every rejection leaves a **log line**, a metric and the payload, so the rate is a number somebody can see.

| The surprise | What survives it |
|---|---|
| A field you did not expect | Ignore it, record that it arrived. A new field is how a producer announces a change |
| A value that is missing | Keep absent, null and empty as three states. Collapsing them invents information |
| A type that does not match | Convert where the conversion is exact, reject where it is a guess. A string that fails to become a number is a rejection, never a zero |

The same discipline runs the other way, where your code feeds the model. A tool that **swallows its failure** and returns an **empty result** gets read as valid data, and the model **reasons confidently on nothing**. Mark it so the turn carries the failure, and catch the SDK's **typed exception classes** rather than matching error text.<sup>[5]</sup>

## A Check Only Counts If Its Truth Comes From Outside the Model

Now the **truth question**. The tempting answers to it share one defect.

Ask for a **self-reported confidence score** and one **generation rates another**, with no new information arriving in between. That rating runs highest where the estimate is most **confidently mistaken**, which is the case you wanted it for. Sample the prompt several times and take the majority, and a **stable error converges** as smoothly as a right answer does. Anthropic documents **best-of-N** as a way to notice **inconsistency** across outputs, which is a signal that something may be wrong.<sup>[6]</sup> **Agreement** is **not the reverse signal**, though it gets read as one constantly. Move to a **stronger model** and the error rate falls while the **class of failure stays**, **undetected** by anything downstream.

**Correlated verification** is the name for all four. The rule underneath is short enough to carry. A **check catches a confident wrong answer** only when its own **correctness** rests on something **established outside the model**.

That points at **unglamorous work**, and the work is the answer.

- **Recompute derived values** in code and compare. **Arithmetic** is the cheapest **independent authority** you own.
- **Cross-check totals and identifiers** against the source records, which is where a figure lifted from a filing meets the filing.
- **Enforce range and consistency rules** drawn from the **domain** rather than from the response.
- Route every violation to **human review** instead of **retrying** until the check passes.

Two documented techniques narrow the problem beforehand. **Restricting Claude to the supplied documents** rather than its general knowledge cuts the class of claims that have **no source** at all,<sup>[6]</sup> and **citations return the exact passages** supporting each claim so a person or a program can check them.<sup>[7]</sup>

**Provenance belongs in the schema** for the same reason the figures do. Somebody acting on a claim needs its support as addressable fields answering four questions. What kind of **source** was it. Which **passage or returned value** does it rest on. When was that **observed**. Has anybody **checked** it. Compressing that into a tidier narrative removes what the reviewer came for. **Attaching the raw transcript** instead hands every reviewer the same reconstruction job.

One mirror case is worth holding. When two **credible sources disagree**, a single confident answer gets **manufactured rather than found**. **Averaging invents** a third figure nobody reported, **keeping the larger** one calls a preference a finding, and **searching until something backs** one of them is a hunt whose verdict was fixed before it began. Carry both, attributed, with the **conflict marked**. An output that **represents the disagreement** is the accurate one.

## How Wrong Answers Are Built on This Material

Wrong answers here describe **real engineering**, and somebody has shipped every one. **Four assumptions** produce most of them.

- **Shape offered against a truth problem**. The **output validated** and was **wrong anyway**, and the option adds a **schema**, a **stricter validator** or a **better prompt**. All three answer a question about **parse failures nobody asked**.
- A **check drawn from the process** it is checking. **Self-reported confidence**, **majority vote**, **self-grading**, a **larger model**. Each fails where the **answer fails**, the only place it was needed.
- **Cooperation requested where enforcement is available**. Asking for **JSON**, asking **more firmly**, **retrying** until something parses. **None fixes the boundary**, none detects being ignored.
- A **failure removed from view** rather than from the **system**. **Dropping non-conforming output** quietly, **coercing an absent value** into a present one, **deleting the source** that disagrees. The symptom stops and **nothing improved**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Output validates and is wrong anyway | Recompute or cross-check outside the model, route violations to a person | A stronger model, a stricter schema, a model-filled confidence field |
| The format varies between calls and a program consumes it | Constrain generation to a declared schema, then validate on receipt | A regular expression over today's formatting, or a firmer prompt |
| Code crashes on an unexpected field, a missing value or a wrong type | Parse defensively: survivable, and still visible | Dropping non-conforming output silently, or leaving the crash |
| Credible sources disagree, or a claim must support somebody else's decision | Carry conflict and provenance as fields: source, excerpt, timestamp, verification state | Averaging, keeping the larger, or a narrative the reviewer reassembles |
| Extraction routes everything on one confidence threshold | Name and exemplify the outcomes: absent-optional, unreadable-required, out of scope | One global cut, or retrying until every field fills |
| The response is refused, truncated, or oddly cased | Branch on `stop_reason` before parsing; compare enums case-insensitively | Treating any `200` as a successful generation |

One habit transfers past the table. For any option offered as a **reliability fix**, ask what it would take for the output to be **wrong and still pass** it. If **nothing outside the model** was consulted, it answers the **shape question**.

## What to Remember

- Every check answers the **shape question**, meaning **fields, types and permitted values**, or the **truth question**, meaning whether **content matches the world**. **Schema validation**, **constrained generation** and **defensive parsing** answer only the first.
- **Structured outputs** compile your **JSON Schema into a grammar** that constrains generation, so **violating output cannot be produced**. `output_config.format` **binds the final response**, `strict: true` **binds tool names** and inputs.
- **Constraining generation removes the failure class**; **parsing harder** changes only how **loudly the consumer fails**. A **prompt-level format request** holds on the inputs you tested and **slips** on the one you did not. The grammar costs **first-call latency**, an **added system prompt**, and **refusal** of a **schema too complex to compile**.
- **Three documented escapes** return `200` without usable structure: a **refusal**, a **`max_tokens` truncation**, and an **enum in different capitalisation** carrying **no error** and no special `stop_reason`. **Read `stop_reason`** before content, and **compare enums** case-insensitively.
- **SDKs strip range and length constraints** from the **schema** they send, so your own **validation on receipt** enforces them. A validation failure **needs a destination**: **retry**, **reject** into a queue, or **route** to a person. Choose by **named category**, because **absent-but-optional, unreadable-but-required and out-of-scope** are outcomes **no confidence number separates**.
- **Defensive parsing survives** the surprise and still reports it: **tolerate unknown fields**, keep **absent, null and empty** distinct, **reject a type mismatch** rather than coercing a guess, **log every rejection**. The **silent drop** is the worse failure, since the **pipeline reports success** while **records disappear**.
- A check **catches a confident wrong answer** only when its own correctness rests on something **established outside the model**. **Self-reported confidence**, **majority vote** and a **bigger model** all fail where the **answer itself fails**.
- Independent verification is **arithmetic and lookups**: **recompute derived values**, **cross-check totals** against source records, enforce **domain range rules**, send violations to **human review**. **Provenance** is a field: **source type, excerpt, timestamp, verification state**. Conflicting credible sources are carried with the **conflict marked**, because **averaging invents a number** nobody reported.

## Endnotes for this chapter

1. Structured outputs constrain Claude's responses to follow a specific schema by compiling the JSON schema into a grammar used for constrained sampling, so that responses violating the schema cannot be generated; `output_config.format` with `type: "json_schema"` controls the response format and `strict: true` guarantees schema validation on tool names and inputs, and the two may be used independently or together in one request; the compiled schema is temporarily cached, so a first call on a new schema carries the compilation; when structured outputs are in use Claude automatically receives an additional system prompt explaining the expected output format; explicit limits apply to the number of strict tools, optional parameters and parameters using union types, with a `400` error reading "Schema is too complex for compilation" beyond them, and each optional parameter roughly doubles a portion of the grammar's state space, so the documented mitigations include reducing optional parameters and simplifying nested structures; schema compliance is not guaranteed on a refusal (`stop_reason: "refusal"`) or on reaching the token limit (`stop_reason: "max_tokens"`, for which the documented response is to retry with a higher `max_tokens`); enum and const capitalisation is not guaranteed, the response completes normally with no error and no special `stop_reason`, and the guidance is to compare enum values case-insensitively and avoid enum values differing only in capitalisation; several SDKs transform schemas by removing constraints such as `minimum`, `maximum`, `minLength` and `maxLength` and adding `additionalProperties: false`, so that Claude receives a simplified schema while your code still enforces all constraints through validation. Anthropic, *Structured outputs* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/structured-outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)

2. Claude 4.6 and later models do not support prefilling assistant messages, and a request whose last assistant message is a prefill returns a `400` `invalid_request_error`; the documented alternatives are structured outputs on models that support them, system prompt instructions, or `output_config.format`. Anthropic, *Claude API errors* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/errors](https://platform.claude.com/docs/en/api/errors)

3. The Agent SDK accepts an `outputFormat` or `output_format` option of `type: "json_schema"` with a JSON Schema, validates the agent's output against that schema and re-prompts on a mismatch, and returns a result subtype of `error_max_structured_output_retries` when no valid output remains after the retry limit; a result can also end with subtype `success` and no `structured_output` value, for example when the run completes without the agent producing a structured output, and that case should also be treated as a failure. Anthropic, *Get structured output from agents* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/structured-outputs](https://code.claude.com/docs/en/agent-sdk/structured-outputs)

4. Every Messages API response includes a `stop_reason` field indicating why Claude stopped generating, and it should be checked in response handling to decide whether to use the response as-is, continue, retry, or fall back; a refusal is returned by safety classifiers as a normal HTTP `200` response with `stop_reason` of `refusal` rather than as an error, and a refused request on some models can usually be served by retrying on another Claude model; `stop_reason` values are distinct from errors, which indicate failures in processing the request. Anthropic, *Stop reasons and fallback* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons)

5. The official SDKs raise typed exceptions rather than returning raw JSON, and the documented guidance is to catch the SDK's typed classes rather than string-matching error messages, handling the most specific classes first. Anthropic, *Claude API errors* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/errors](https://platform.claude.com/docs/en/api/errors)

6. Documented techniques for minimising hallucinations include best-of-N verification, running Claude through the same prompt multiple times and comparing outputs on the basis that inconsistencies across outputs could indicate hallucinations; chain-of-thought verification; iterative refinement; and external knowledge restriction, explicitly instructing Claude to use only information from provided documents rather than its general knowledge. Anthropic, *Reduce hallucinations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)

7. Citations ground responses in supplied source documents and return the exact passages that support each claim, so the sources behind a response can be tracked and verified. Anthropic, *Citations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/citations](https://platform.claude.com/docs/en/build-with-claude/citations)
