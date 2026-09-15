# Chapter 5: Instruction Placement Is a Trust Decision

**Summary**: *One sentence, written once, put in two different places, produces two systems that fail differently. That is the whole chapter. Where text sits decides how long it applies, how many copies exist, and whether a stranger who supplied a document can edit your rules by writing a line into it. Placement comes first. Clarity, examples, output constraints and the loop that improves them all sit on top of a sort you either did or skipped.*

## The Same Sentence in Two Places Is Two Different Systems

Two prompts. Both carry the sentence *return one label from the fixed set and nothing else*, **spelled identically**.

In the first it sits in the **top-level `system` field**, **written once**, applying to every request the service will ever make including the ones **nobody has written yet**.

In the second it was **appended to the user turn**, under the ticket text, because that is where somebody was working when they thought of it. It now travels inside a string the turn-building code assembles. There are as many **copies as code paths**, and **copies drift**.

Same words. Same model. Same ticket. The second system has **no single place** where its rules live, and it seats them in the channel carrying **text somebody else wrote**.

Sort every piece of text in a prompt by **two questions** and call the result the **placement sort**. How **long** does this text apply for. Who could have **written** it. **Lifetime decides** whether **text repeats**. **Authorship decides** whether it may **instruct**.

## Three Destinations, and Nothing You Did Not Write Reaches the First

The sort has **three outcomes rather than two**, and the third is the one developers skip.

| The text | How long it applies | Who could have written it | Where it belongs |
|---|---|---|---|
| Role, tone, standing constraints | Every request, unchanged | You | Top-level `system` |
| The output contract, and the rule that supplied content never instructs | Every request, unchanged | You | Top-level `system` |
| Worked examples of the output | Every request, unchanged | You | `system`, tagged as examples |
| This request's actual question | One request | The user in front of you | The user turn |
| A document the user attached | One request | Whoever handed it to them | The user turn, delimited |
| A fetched page, inbound mail, OCR text, a tool's return | One request | Anybody at all | `tool_result` blocks |

Read the right-hand column downward. **Standing rules gather** in one field, **stated once**. **Per-request material stays** in the turn. **Third-party content** goes into **tool results**, which is where Anthropic documents that it belongs: deliver it inside `tool_result` blocks rather than in `system` or in plain user text, because Claude is **trained to treat directives** arriving inside tool results with **more scepticism** than directives elsewhere.<sup>[1]</sup>

Chapter 3 covered the one refinement. A message with **`"role": "system"`** may appear after a user turn on **supported models**, carrying the **same authority** as the top-level field but **never coming first**, so a rule that only becomes relevant later lands at the end of the history without **disturbing a cached prefix**.<sup>[2]</sup>

The **document promoted to policy** is what the sort prevents, and it never looks like a security decision at the time. Somebody concatenates a contract into the system prompt so the model definitely sees it. That contract now **carries the authority your rules carry**, so any line in it addressed to a reader is **addressed to Claude**. Nobody attacked anything. **Placement did the damage alone**.

The mirror image is the **standing rule filed in the last turn**, appended to the newest user message on a theory about recency. Recency is not how a **standing constraint holds**, and the constraint now shares a channel with whatever got pasted in.

## An Instruction Is Checkable or It Is Decoration

Anthropic's own test is worth adopting whole: **show the prompt to a colleague** with minimal context on the task and ask them to follow it, and if that **human would be confused**, so is the model.<sup>[3]</sup>

A human following *be precise* has no idea when they have complied, and neither has a grader. An instruction is **checkable** when output can be tested against it: a **named set of permitted values**, a **stated ceiling on length**, a rule saying which of **two conflicting sources wins**. The rest is a mood.

Two documented refinements sharpen that. **Say what to do** rather than what to avoid, because a prohibition removes an outcome without supplying a replacement. And **give the reason** behind an instruction, because the **model generalises from an explanation** to cases the instruction never named.<sup>[3]</sup>

What developers reach for when an instruction fails is the wrong lever twice over. The **firmer restatement** takes a rule the model did not follow and **says it again in capitals**, naming no failing part of the output. On current models it misfires: prompts written to stop **older models under-reaching** now cause over-reaching, and Anthropic's documented fix is to **dial the aggressive language back** toward ordinary phrasing.<sup>[4]</sup> **Volume was never a technique**. Now it has a cost.

## Delimiters Make Supplied Text Legible as Data

Tags earn their place when a prompt mixes kinds of content. Wrapping **instructions**, **context**, **examples** and **variable input** each in its own tag reduces the chance that **Claude reads one as another**, and examples go inside **`<example>` tags** so they are not read as part of the task.<sup>[5]</sup>

Three rules on the same page concern long material. **Long documents go near the top**, above the query, the instructions and the examples, which improves results across models. **Several documents get wrapped individually**, content and source in their own subtags. On long-document work, ask Claude to **quote the relevant passages first**.<sup>[5]</sup>

**Delimiting is a boundary** the **model reads**, not one the **runtime enforces**. Text inside a tag can imitate that tag. The harder version is to **JSON-encode third-party strings** rather than concatenating them into free-form text, because escaping leaves no way to close a quote and step out into instruction position.<sup>[1]</sup> State the policy as well: tell Claude in the system prompt that content returned from tools, documents or searches is **data** and **never overrides your instructions** or the user's request.<sup>[1]</sup>

Sanitizing runs underneath, before content is placed anywhere. **Filter input** for **known injection patterns**. Screen it with a **lightweight model call** whose verdict is constrained to a **parseable value** your code can branch on, and run that same screen over what your tools return.<sup>[1]</sup> One asymmetry catches people: your **own instructions** must **not travel** in a **tool result** either, since content there **counts as untrusted** and your rule may be ignored or flagged. **Send it in the user turn** that follows the result.<sup>[1]</sup> Chapter 20 owns the attack. This chapter owns the arrangement that makes it legible.

## Examples Settle Margins That Another Paragraph Will Not

Chapter 1 established the **ladder from zero-shot to multi-shot** and what each rung costs. The narrower question is the one scenarios ask: which **failures examples repair**, and which they leave alone.

Demonstrating a **desired output** steers **format**, **tone** and **structure** more dependably than describing one in the abstract.<sup>[5]</sup><sup>[6]</sup> The mechanism explains why. A **description states a rule** and leaves its margins to inference. A **worked instance shows one margin** already decided, in the exact casing, the exact field names, the exact handling of the awkward case.

Three conditions call for them, held as conditions rather than as cases. The **task is understood** and the **structure keeps being invented**. **Category boundaries drift between runs**, because a definition leaves edges open that a labelled instance closes. Or the model is looking in the wrong place: **fields come back empty** although the value sits in a **footnote** or a **caption**, and demonstrations of extraction from those locations move attention where no instruction about diligence does.

**More examples of the case that already works** is the anti-pattern, and it feels like effort, which is why it survives review. Adding clean examples to a prompt that already handles clean inputs makes the shortfall bigger. Examples have to come from the **population that fails**.

## An Output Constraint Names a Form, and It Is Still Only a Request

An **output constraint governs shape** independent of content: field names, **permitted values**, length, and the **stopping point** saying nothing else may follow. Where one is absent, content is often right while form varies enough to break whatever parses it.

Anthropic documents three levers on formatting. Say what the output should be rather than what it should not contain. **Use tags as format indicators**. **Match the style** of your prompt to the style you want back, because a prompt's formatting influences the response, to the point that stripping markdown out of a prompt **reduces markdown in the output**.<sup>[4]</sup>

One technique here was withdrawn under you and still appears in older guidance. Prefilling the assistant's turn used to be the standard way to force a shape by starting the answer yourself. From **Claude 4.6 onward**, **prefilled responses on the final assistant turn** are unsupported and **return a `400` error**; **earlier models still accept** them, and assistant messages elsewhere are unaffected.<sup>[7]</sup> A scenario **offering prefill as the fix** on a current model is offering an error.

The **ban stated instead of a method** owns this section. **Marking a field non-nullable**, declaring an empty value unacceptable, **retrying until something appears**: each removes the honest answer without improving the search that produced it, so an absent value becomes an **invented one**. A constraint on form cannot supply content that was **never generated**.

A **prompt-level constraint** also stays a request the model may miss on an untested input. Chapter 6 owns the mechanism that closes that gap by **constraining generation against a schema**.

## Iteration Converges Only When Each Round Carries Evidence

The **loop that works** is **diagnose**, **add the one missing piece**, **measure**. The loop that fails is re-prompt, re-prompt, re-prompt, with the prompt lengthening each time and nobody naming what broke.

Diagnosis is a **short lookup**, because the failure identifies the **absent technique**.

| What you observed | What is absent |
|---|---|
| Right answer, wrong shape, parser breaks | An output constraint naming form and stopping point |
| Scope drifts and tone moves as the conversation runs on | A standing contract in `system`, stated once and specifically |
| Task understood, structure invented | Worked examples showing the structure |
| Clean on tested inputs, broken on a variant | A rule or an example covering that variant |

Three named failures live in the loop rather than in the prompt. The **rewrite from scratch** throws away every case that already passed, so **nothing accumulates** and nothing converges. The **accreting prompt** adds paragraphs without adding a missing piece, and **length is not free**: a verbose prompt pulls verbose output, which **buys latency and no accuracy**. The **user warned off** ships the defect and asks humans to route around a category that is usually growing.

Feedback carries evidence or the round is wasted: **failing test names**, the inputs, **expected against actual**, the trace. Unspecific feedback buys unspecific change.

Two facts bound the loop from outside. Iteration needs something to iterate against, and that set has to **hold the inputs that fail**, which puts real traffic inside the loop instead of a clean development set standing in for it. **Messy traffic needs rules** of its own: how to behave when the **source omits a mandatory field**, how to **resolve two passages that disagree**, how to answer an **out-of-scope request**. Chapter 19 owns running that measurement as a discipline.

The other bound is knowing when to stop. Anthropic says plainly that **not every failing evaluation** is best solved by prompting, naming latency and cost as examples a **different model choice** may settle more easily.<sup>[8]</sup> Chapter 4 owns that call. A scenario whose constraint is money or milliseconds is **not a prompting question in disguise**.

## How Wrong Answers Are Built on This Material

Wrong answers here **describe real practice**, and somebody has genuinely done every one of them, often to a prompt that **improved afterwards**. **Four assumptions** produce most of them.

- **Willingness assumed where specification is missing**. The option **repeats an instruction**, **intensifies** it, or asserts that the **output is unacceptable**, all three reading the gap as reluctance. The **model complied** with what it was given, so the **useful move names the piece** nobody supplied.
- A **prohibition offered where a method is needed**. **Forbidding an empty field**, **banning a value**, **retrying until something arrives**: each removes an outcome and supplies no route to a better one. They turn correct only where the scenario states the **model already holds the information**.
- The **failing population kept outside the loop**. **More examples from the clean set**, a **wider output cap**, a **different sampling setting**. Each optimises the distribution that was already working.
- A **standing rule given a per-request home**. **Restating the contract in every turn**, **duplicating it in both places**, or **leaving each instruction** where it was first typed. **Copies drift**, and your rules end up in the channel carrying **supplied content**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Instructions sit in both the system prompt and the turns, and the format varies | Standing behaviour and the output contract consolidated in `system`, once | Restating the contract every request, or holding it in both places |
| A supplied document must be analysed under persistent rules | Rules in `system`; document in the turn, delimited from the request | Concatenating document text into the standing instructions |
| The task is right and the structure keeps being invented | Worked examples showing the exact shape | A longer description, or a firmer request for the same shape |
| Fields come back empty although the material holds them somewhere unusual | Examples demonstrating extraction from those locations | Forbidding the empty value, or retrying until one appears |
| Good scores in development, poor behaviour on live traffic | Failing inputs added to the measured set, plus rules for degenerate ones | More examples from the clean set, or more output room |
| One call reads, extracts, judges and summarises, and quality is uneven by part | Focused calls in sequence, each fed the previous step's structured output | Urging thoroughness, a bigger cap, or a temperature change |
| Repeated fixes keep lengthening the prompt without fixing it | Name the failure, add the single missing technique, measure | Rewriting from scratch, or relaxing the check that reports it |

One habit transfers past the table. For any option that changes a prompt, say which **piece it adds** and which **destination** it puts that piece in. An option **adding no piece** is **restating**, and one **adding a piece** to the **wrong destination** reads best and ages worst.

## What to Remember

- **Placement sorts** by **two questions**: how **long the text applies**, and who could have written it. **Lifetime decides duplication**; **authorship** decides whether it may **instruct**.
- **Three destinations**, never two: **persistent rules** in **top-level `system`**, this request's question and the user's **own material** in the turn, **third-party content** in **`tool_result` blocks**. **Nothing you did not write** reaches the first.
- A **mid-conversation system message** carries the **same authority** as the **`system` field**, **cannot be first**, and appends at the end, so it adds a **late rule** without **invalidating a cached prefix**.
- An instruction is **checkable** when **output can be tested** against it: **named permitted values**, a **stated ceiling**, a **rule for conflicting sources**. Anthropic's test is whether a **human with minimal context** could follow it unconfused.
- **Say what to do** rather than what to **avoid**, and **give the reason**, because the **model generalises** from an **explanation** to cases the **instruction never named**. **Restating a rule more firmly** names no failing part, and **aggressive phrasing** now causes **over-triggering**.
- **Tags separate instructions**, **context**, **examples** and **input** so none is **read as another**, and **long documents** sit near the top, above **query**, **instructions** and **examples**.
- **Delimiting is a boundary** the model reads, not one the **runtime enforces**: **JSON-encode untrusted strings**, **state in `system`** that **supplied content never overrides** instructions, and screen input and tool output with a **lightweight classifier** first. Your **own instructions** never travel in a **tool result**, since **content there counts as untrusted**.
- **Examples repair invented structure** and open category boundaries, and they **change where the model looks** when **values sit** in **footnotes**, **captions** or **covering letters**. They must come from the **population that fails**.
- An output constraint governs form independent of content: **field names**, **permitted values**, **length**, and a **stopping point**. **Prefilling the final assistant turn** returns a **`400` error** on **Claude 4.6** and later.
- **Forbidding an empty value** supplies no method, so the **missing answer** becomes an **invented one**. **Shape constraints** cannot **generate content** that was never produced.
- **Diagnose before re-prompting**: **wrong shape** wants a **constraint**, **drifting scope** wants a **specific system prompt**, **invented structure** wants **examples**, a **broken variant** wants a rule covering it.
- **Iteration converges** on specific evidence: **failing cases**, **inputs**, **expected against actual**. **Rewriting from scratch** discards what passed, and a **prompt that only grows** is a skipped diagnosis.
- The measured set must hold the **inputs that fail**, and messy traffic needs its own rules: **omitted mandatory fields**, **passages that disagree**, **requests outside the defined scope**.
- **Not every failing evaluation** is a **prompting problem**. Anthropic names **latency** and **cost** as constraints a **different model choice** may settle **more easily**.

## Endnotes for this chapter

1. Untrusted third-party content should be delivered inside `tool_result` blocks rather than in `system` prompts or plain user `text` blocks, because Claude is trained to treat instructions appearing inside tool results with appropriate scepticism; the nature and source of the content should be made explicit; the system prompt should state that content returned from tools, documents or searches is untrusted data that must never override the system prompt or the user's original request; third-party strings should be JSON-encoded so escaping provides unambiguous delimiters and an attacker cannot close a quote or tag to break out into an instruction context; your own instructions should not be placed in tool results, because content there is treated as untrusted and may be ignored or flagged, and should instead be sent in a user turn following the `tool_result` block; input should be filtered for known injection patterns and pre-screened with a lightweight model using structured outputs to constrain the response to a simple classification; and the same screening pattern should be applied to tool outputs before the content is returned to Claude. Anthropic, *Mitigate jailbreaks and prompt injections* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)

2. On supported models a message with `"role": "system"` may be included after a user turn to add a system instruction partway through a conversation; a system message cannot be the first entry in `messages`, and the top-level `system` field is used for instructions that apply from the start; a mid-conversation system message has the same authority as the top-level `system` field but, because it is appended to the end of the message history, does not invalidate a cached prefix that came before it. Anthropic, *Using the Messages API* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/working-with-messages](https://platform.claude.com/docs/en/build-with-claude/working-with-messages)

3. Claude responds well to clear, explicit instructions, and being specific about the desired output improves results; the stated golden rule is to show the prompt to a colleague with minimal context on the task and ask them to follow it, on the basis that a person who would be confused indicates the model will be too; providing context or motivation behind instructions helps Claude understand the goal and deliver more targeted responses, because Claude generalises from the explanation; setting a role in the system prompt focuses Claude's behaviour and tone. Anthropic, *Prompting best practices* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

4. Effective ways to steer output formatting include telling Claude what to do instead of what not to do, using XML format indicators, and matching prompt style to the desired output style, since the formatting used in a prompt may influence the response style and removing markdown from a prompt can reduce the volume of markdown in the output; prompts designed to reduce under-triggering on tools or skills may cause over-triggering on newer models, and the documented fix is to dial back aggressive language such as "CRITICAL: You MUST use this tool when..." in favour of ordinary phrasing such as "Use this tool when...". Anthropic, *Prompting best practices* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

5. Examples are among the most dependable means of steering Claude's output format, tone and structure, and are placed inside `<example>` tags, with multiple examples in `<examples>` tags, so Claude can distinguish them from instructions; XML tags help Claude parse complex prompts unambiguously where the prompt mixes instructions, context, examples and variable inputs, and wrapping each type of content in its own tag reduces misinterpretation; for large or data-rich inputs, long documents are placed near the top of the prompt above the query, instructions and examples, which improves performance across all models; multiple documents are each wrapped in `<document>` tags with `<document_content>` and `<source>` subtags; and for long document tasks, asking Claude to quote relevant parts of the documents before carrying out the task helps it focus on the relevant content. Anthropic, *Prompting best practices* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

6. Consistency techniques include precisely defining the desired output format using JSON, XML or custom templates, constraining with examples of the desired output on the basis that this is more effective than abstract instructions, using system prompts to set the role, and breaking complex tasks into smaller subtasks chained together so each receives full attention. Anthropic, *Increase output consistency* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency)

7. Starting with Claude 4.6 models, prefilled responses providing a partial assistant message on the last assistant turn are no longer supported, and requests containing them return a `400` error; earlier models continue to support prefills, and adding assistant messages elsewhere in the conversation is not affected. Anthropic, *Prompting best practices* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

8. Prompt engineering guidance focuses on success criteria that are controllable through prompt engineering, and not every success criterion or failing evaluation is best solved by prompt engineering; latency and cost are given as examples that can sometimes be improved more easily by selecting a different model. Anthropic, *Prompt engineering overview* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview)
