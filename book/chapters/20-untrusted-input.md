# Chapter 20: The Attack Arrives as Content You Asked For

**Summary**: *Nothing about the dangerous input is malformed. It is a ticket, a document, a fetched page: exactly the material the system was built to read, carrying one sentence addressed to the model instead of to a reader. Any defence you write as a sentence is read by the same reader, on the same footing, and a persuaded reader follows the other one. This chapter sorts security controls by one property, whether the attacker's text can reach the place a control lives, and that sort settles most of what this skill asks.*

## A Ticket That Passes Validation and Still Contains an Order

Your agent handles **support tickets**, and tickets **come from the public**. One arrives. The subject line is a subject line, the body is prose, every field **holds the type it declares**. Somewhere in that body sits a sentence **written for the model** rather than for the human who will read the reply.

Nothing rejected it, because there was **nothing to reject**. Text in a text field is **not an anomaly**.

A model **receives one sequence**. Your standing instruction, the turn, a retrieved document and a tool result arrive as **tokens in the same stream**, and nothing in the stream **marks which span you wrote**. **Prompt injection** falls out of that: **instructions planted in content** the model reads for somebody, weighed alongside yours, because from inside the sequence they are the same kind of thing. Anthropic's agent threat model names **injection and model error** as the two routes to an action nobody intended.<sup>[1]</sup>

The vector is **not a file type**. It is every span of text the system reads that a **stranger composed**: inbound mail, a page pulled in to be summarised, characters recovered from an upload, whatever a tool returns after fetching somewhere else.<sup>[2]</sup> Planting can happen **long before the request** that trips it, in a region of the page **no human reads**.

## Every Control Sits Inside the Attacker's Channel or Outside It

One **question sorts** the subject. Can the **attacker's text reach** the place the control lives?

Call everything it reaches the **writable channel**: the **assembled prompt**, the **model's reading** of it, the **model's output**. Inside that channel a **control is a request**, because the injected sentence sits beside it and argues. Outside it a **control is a rule**, because the injected sentence never gets to speak to it.

| Where the control sits | Can injected text argue with it | What it buys |
|---|---|---|
| A sentence telling the model to disregard embedded instructions | Yes. Both are text the same reader weighs | Fewer landings, no floor under the failure rate |
| Untrusted content delimited, encoded and declared as data | Partly. The attacker writes inside the delimiter, not around it | A boundary that must be broken rather than ignored |
| A classifier screening content before the conversation sees it | No. It runs before anything is in context | A verdict the content cannot appeal, on shapes it knows |
| A tool accepting catalogue identifiers rather than arbitrary targets | No. The capability is absent | The disallowed action becomes impossible, not unwanted |
| Server-side authorisation on the operation, against the caller | No. It never reads the content at all | The same verdict on a persuaded model as a clean one |
| Data never sent; reach never granted | No. Nothing is left to argue with | Nothing downstream can undo it |

One property changes down that middle column: whether the control can be **talked out of its verdict**. A probabilistic control has **no floor**. Claude is trained to resist these attacks and Anthropic still recommends **defence in depth**,<sup>[1]</sup> which is the honest reading of every row above the last two.

The **defence written as a sentence** is the anti-pattern. It costs nothing and **reads like policy**. It is not what stands between a hostile ticket and a refund.

The same sort says what a model adds to a threat model. Flooding, physical access and a vulnerable client library are **conventional risks** a review already carries. The additions exist because untrusted text gets interpreted and the interpretation reaches privileged actions: **injection**, **leakage of prompt or context content**, **jailbreak attempts**, and **output that reaches an action path** without passing an application control.

## A Jailbreak Attacks the Model's Limits; an Injection Attacks Your Application's Actions

**Two threat models**, not two names for one thing, and the difference is who is **hostile**.<sup>[2]</sup>

In a jailbreak the **human using your application** is the **attacker**, shaping input until it produces content or takes an action you would refuse. In an injection that **person is the victim**, and the **attacker wrote** the content Claude read for them.<sup>[2]</sup>

That decides which remedies exist. Against a jailbreak there is an **account to act** on: **screen input** before the main conversation sees it, and **throttle or ban** somebody who keeps triggering the same refusal.<sup>[2]</sup> Against an injection there is **nobody to throttle**. The requester is innocent and the **document is the attacker**.

The defensive shape is identical either way. **Constrain what reaches the model**, then constrain what it may do about it. Defend only the first half and a steered model **keeps every permission** it started with.

## Untrusted Content Enters as Data, in Its Own Block, After a Gate

**Three moves**, each one further outside the **writable channel** than the last.

**Screen it before assembly**. A check that runs before the prompt is built is **outside the channel** by construction. Anthropic documents a **lightweight model pre-screening input**, with **structured outputs pinning the verdict** to a value the application can branch on, and the same pattern over tool output: run the tool, screen what came back, pass it on only if the screen reports nothing.<sup>[2]</sup>

**Deliver it as data with its origin declared**. Third-party content **belongs in tool result blocks** rather than a system prompt or a plain user text block, because Claude is trained to treat directives arriving in a tool result sceptically.<sup>[2]</sup> **Encoding it as JSON** makes **escaping the boundary** instead of punctuation, so a closing quote or tag cannot break out into instruction context.<sup>[2]</sup> Say what the content is and where it came from, in the tool description or the result structure.<sup>[2]</sup>

**Two properties** decide where a span belongs: how long it lives, and who wrote it. **Long-lived material you authored** is the **standing instruction's job**. Anything arriving with this one request, from outside, sits in the turn behind delimiters. That classification runs both ways. This is the part people miss: an instruction of your own **placed inside a tool result** is content the model **treats as untrusted**, so it may be disregarded or read as the attack, and it belongs in a **user turn after the block**.<sup>[2]</sup>

The **concatenated prompt** is the anti-pattern: instruction and document **joined into one string** before anything else runs. It **erases the distinction** the rest of the defence needs, and parsing defensively afterwards **repairs a result's shape** rather than restoring a boundary. Chapter 5 covers placement as prompt craft.

## The Only Leak Control Nothing Downstream Can Undo Is Not Sending the Data

**Sort leakage controls** the same way, by what has **already happened** when they run.

**Redaction, access restriction and periodic review** act on data that exists, which makes them recoveries. **Minimisation acts** before the data exists anywhere else. Send the fields the task needs and no others. A record **never transmitted cannot leak**.

Two failures earn names. The **debug log nobody scoped** keeps **raw requests and responses** for troubleshooting and becomes a **second copy** of everything sensitive, held longer than the first and readable by more humans. **Tokenising identifiers** before the write and restricting who reads the store are the repairs, and both sit downstream of a request that carried the identifiers anyway. **Redaction delegated to the model** makes a privacy guarantee depend on a step with **no floor**, acting after the data has crossed. An **unguessable storage path is obscurity**, not a third option; anyone who learns the path reads the store. Obscurity **expires on discovery**.

What replaces all of it is a **boundary with a stated crossing rule**: what may leave, what gets filtered or redacted at the line, **decided before the call** rather than discovered from incidents. Chapter 16 treats that boundary as a design object.

Two specifics are worth carrying. **Protected health information** in the Claude API turns up in **message content**, in **attached files**, and in **file names and metadata**, while workspace names, user contact details, billing data and support tickets are not expected to carry it under a BAA.<sup>[3]</sup> One route around that is easy to miss. The leak goes out through the schema: with structured outputs or strict tools, **JSON schemas compile into grammars cached separately** from message content, and those caches do not get the same protections, so keep identifying detail out of **property names**, **enum values**, **const values** and **pattern regexes**.<sup>[3]</sup>

Retention and residency are configuration rather than prompting. **Zero data retention** means prompts and responses are **not stored at rest** once the response returns, and it is **enabled per organisation**. **HIPAA readiness applies safeguards** across the whole lifecycle instead of immediate deletion, and an organisation handling PHI wants that rather than ZDR.<sup>[3]</sup> Residency is **two independent settings**: where **inference runs**, and where **data is stored**.<sup>[4]</sup> A prompt leak runs the other way, and the first recommendation there is to leave proprietary detail the task does not need out of the prompt.<sup>[5]</sup>

## Authorisation Belongs to the Operation and Is Checked Against the Caller

A **tool call** is a **request**. It is never an **entitlement**, and that distinction is the whole of authorisation.

The check that holds **runs server-side on the operation**, against the **identity doing the calling**: does this caller own the record, is the amount inside policy, is the action permitted for the role. It gives the **same verdict on a persuaded model** as on a clean one, because it **never reads the ticket**. Everything that inspects the request instead **sits in the channel**. **Summarising first** does not strip the hostile line out; it **relocates the line into the summary**, and the summarising call is one more thing to steer. A cheaper model is not a boundary either, since following instructions is what a model is for.

**Authentication comes first**, and **identity has to be settled** before any operation that depends on it. A lookup returning **several plausible matches** is an **unresolved identity**, and **one disambiguating question** is the remedy. **Ranking by recency** is the same guess with a threshold on it. Probing each candidate's records to work out which is which discloses one account's detail in the act of deciding it was the wrong account.

**Least privilege** then **decides severity** rather than likelihood. An injection through an **identity that can write anywhere** is an **incident**. Give that identity **one writable directory** and the identical attempt ends as a **refusal plus an audit line**. The tool surface obeys that rule too. A tool taking an **identifier validated against an approved catalogue** cannot fetch an arbitrary target, where a general fetch tool plus a warning leaves the capability sitting there and relies on the model declining. Filtering results afterwards is later still. The page was already retrieved, and the answer it produced is already shaped by material nobody approved.

**Two containment controls** earn their names. **Network egress restriction stops the outbound request** itself, which is why an agent told by a malicious file to send customer data outward is blocked at the network rather than at the prompt;<sup>[1]</sup> a sandbox with no egress rules lets a compromised tool execution reach arbitrary hosts.<sup>[6]</sup> **Default deny** is the other. Claude Code **matches bash commands fail-closed**, so an **unmatched command needs approval**, and **web fetch gets its own context window** so retrieved content never lands in the main one.<sup>[7]</sup> Chapter 21 covers hooks as the enforcement point and chapter 22 the credentials.

## How Wrong Answers Are Built on This Material

Wrong answers here describe **real security work**. Most are controls that would be right against a **different threat**, or right **one step later** than the scenario needs.

- A **control placed where the attacker writes**. A **stronger instruction**, a **firmer refusal policy**, a reminder that documents are untrusted. **Text competing with text** is a **mitigation, never a boundary**. Right when the question is how often an **attempt lands**.
- **Detection offered where prevention was available**. **Logging and reviewing**, **filtering output**, **alerting on anomalies**. All **observe an event that has happened**, and several create a fresh copy of what is being protected. Right once the **exposure is accepted** and evidence is what is wanted.
- The **model made part of its own containment**. Asking it to **judge whether content is trustworthy** hands **containment to a reader** the content is already talking to. Asking it to **leave sensitive fields** out does the same to **privacy**.
- A **capability left in place and discouraged**. The **broad tool stays**, and a **warning**, a **preference** or a **later filter** goes around it. Right only where the **capability is genuinely needed**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Content written by outsiders is read, and a privileged action exists | Authorise the operation server-side against the caller | A prompt instruction, a summarisation pass, or a smaller model as a filter |
| A general-purpose tool is being used beyond its intended scope | Replace it with one taking validated identifiers | A warning in the prompt, or filtering results after the fetch |
| Identifying data is being written to logs for debugging | Send less in the first place, and redact before the write | Retaining raw payloads, or asking the model to omit identifiers |
| Untrusted text must not be read as instruction | Screen it, then deliver it delimited inside a tool result | Concatenating it with the instruction, or putting it where standing instructions live |
| A lookup returns more than one matching identity | Treat identity as unresolved and ask one distinguishing question | Ranking by recency, or reading each candidate's records to decide |
| A regulatory constraint is stated alongside a delivery deadline | Treat the constraint as fixed and choose inside what it leaves | Weighing it against speed as one factor among several |
| Model-specific threat categories are asked for | Injection, context leakage, jailbreak, unsafe output reaching an action | Restating conventional infrastructure or supply-chain risks |

One habit transfers past the table. For any control on offer, ask whether the **attacker's words and the control's words** end up read by the **same reader**. Where they do, what you have been offered is a **preference**.

## What to Remember

- **Prompt injection** is **instructions planted in content** the model reads for somebody, **weighed like your own** because a model gets **one token stream with no mark** dividing trusted from untrusted. The vector is any **text a stranger can write** that your system reads, **planted content included**.
- Sort every control by whether the **attacker's text can reach** it. Inside that **writable channel**, meaning the **assembled prompt**, the model's **reading of it and its output**, a control is a **request**; outside it, a **rule**. An **instruction to ignore embedded instructions** is **read by the reader** being attacked.
- A **jailbreak targets the model's own limits** and your user is the **attacker**, so **input screening and throttling** repeat offenders apply. An **injection targets your application's actions** and your user is the **victim**, so **nobody is left** to throttle. Both need the **same two halves**: **constrain what reaches the model**, then **constrain** what it may do.
- Untrusted content is **screened before assembly**, delivered inside **tool result blocks** rather than a system prompt or plain user text, **JSON-encoded** so **escaping supplies the delimiter**, and **labelled with its source**. Your **own instructions never go in a tool result**, because the model treats that content as **untrusted too**.
- **Minimisation is the leak control** with nothing left to undo, because **redaction, access limits and review** act on data that already exists. **Send only the fields** the task needs, **redact before the log write**, and treat an **unguessable path as obscurity** rather than access control.
- PHI in the Claude API **sits in message content**, **attached files**, and **file names and metadata**, and is not expected in **workspace names, user contact details**, **billing or support tickets**. JSON schemas compile to **separately cached grammars** without those protections, so keep identifiers out of **property names, enums, consts and patterns**.
- **Zero data retention** stops **storage of prompts and responses** at rest after the response returns and is **enabled per organisation**; **HIPAA readiness applies lifecycle safeguards** instead of deletion and is the arrangement for **PHI**. Residency is **two settings**: where **inference runs**, and where **data is stored**.
- **Authorisation belongs to the operation**, enforced **server-side against the requesting identity**, giving the same verdict on a **persuaded model**. **Settle identity** before operating: **several matches** means unresolved, and one **disambiguating question** beats ranking by **recency**.
- **Least privilege decides severity, not likelihood**: one injection is an incident against a **broad identity**, and a **refusal plus an audit line** against a narrow one. A tool taking **validated catalogue identifiers** makes a disallowed fetch **impossible rather than discouraged**.

## Endnotes for this chapter

1. Agents can take unintended actions due to prompt injection, meaning instructions embedded in content they process, or due to model error; Claude models are designed to resist this and defence in depth remains good practice, the worked example being that if an agent processes a malicious file instructing it to send customer data to an external server, network controls can block that request entirely; a security boundary separates components with different trust levels, and least privilege restricts the agent to only the capabilities its specific task requires. Anthropic, *Securely deploying AI agents* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/secure-deployment](https://code.claude.com/docs/en/agent-sdk/secure-deployment)

2. Jailbreaking and prompt injection fall into two categories with different threat models: in the first, a user deliberately crafts inputs to manipulate the application into producing content or taking actions its operator does not want, mitigated by harmlessness screens using a lightweight model such as Claude Haiku 4.5 with structured outputs constraining the response to a simple classification, by input validation for known injection patterns, by system prompts that state ethical and legal boundaries, and by responding to repeat offenders through throttling or banning; in the second, the application's users are being protected from instructions embedded in content Claude reads on their behalf, such as the body of an inbound email, a fetched web page, OCR output from an uploaded file, or the result of a tool call. Recommended structure for the second: put untrusted content only in `tool_result` blocks and never in system prompts or plain user text blocks, because Claude is trained to treat instructions appearing inside tool results with appropriate scepticism; tell Claude what the content is and where it came from, in the tool description or the result structure; state in the system prompt that content returned from tools, documents or searches is untrusted data that must never override the system prompt or the user's original request; JSON-encode untrusted content so that escaping provides unambiguous delimiters and an attacker cannot close a quote or tag to break out into an instruction context; do not put your own instructions in tool results, because Claude treats tool-result content as untrusted data and instructions placed there may be ignored or flagged as a potential injection, so send them in a following user turn; limit Claude's access to sensitive data and actions under the principle of least privilege; screen tool outputs with the same lightweight-model classifier pattern before returning them as `tool_result` blocks; red-team the agent before deployment; and analyse outputs regularly for signs of successful injection. Anthropic, *Mitigate jailbreaks and prompt injections* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)

3. Under a zero data retention arrangement Anthropic does not store customer prompts or responses at rest after the API response is returned, and ZDR is enabled per organisation; HIPAA readiness applies a broader set of privacy and security safeguards than ZDR, namely encryption, access controls and audit logging protecting PHI throughout its lifecycle, rather than requiring immediate deletion, and an organisation handling PHI should use HIPAA readiness rather than also needing ZDR; protected health information in the context of the Claude API typically appears in message content, attached files such as images and PDFs, and file names or metadata associated with message content, while workspace names, user information such as name, email and phone number, billing data and support tickets are not expected to contain PHI under the BAA; when using structured outputs or tools with `strict: true`, the API compiles JSON schemas into grammars cached separately from message content and those cached schemas do not receive the same PHI protections, so PHI must not appear in schema property names, `enum` values, `const` values or `pattern` regular expressions. Anthropic, *API and data retention* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/manage-claude/api-and-data-retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)

4. Data residency controls are governed by two independent settings, one controlling where model inference runs, available as the `inference_geo` API parameter or as a workspace default, and one controlling where data is stored. Anthropic, *Data residency* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/manage-claude/data-residency](https://platform.claude.com/docs/en/manage-claude/data-residency)

5. Prompt leaks can expose information expected to be hidden in a prompt, and no method is foolproof; documented strategies are separating context from queries using system prompts, post-processing Claude's outputs to filter for keywords indicating a leak, avoiding unnecessary proprietary details that the task does not require, and auditing prompts and outputs periodically; leak-resistant techniques should be used only when necessary because the added complexity can degrade performance elsewhere in the task. Anthropic, *Reduce prompt leak* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-prompt-leak](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-prompt-leak)

6. In the shared responsibility model for self-hosted sandbox environments, network egress control is owned by the customer: sandbox network access is determined by the customer's VPC and firewall rules, and without egress restrictions a compromised tool execution can reach arbitrary external hosts, so outbound traffic should be restricted to only the endpoints the tools require; tools run inside the sandbox with whatever permissions the process has, so least privilege applies to the process user and only required directories should be mounted. Anthropic, *Security model* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security)

7. Claude Code's documented protections against prompt injection include a permission system requiring explicit approval for sensitive operations in Manual mode, context-aware analysis of the full request, input sanitisation, and network commands such as `curl` and `wget` not being auto-approved by default; further safeguards include isolated context windows, where web fetch uses a separate context window to avoid injecting potentially malicious prompts, fail-closed matching, where unmatched commands require approval by default in Manual mode, command injection detection requiring manual approval of suspicious bash commands even when previously allowlisted, and trust verification for first-time codebases and new MCP servers. Anthropic, *Security* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/security](https://code.claude.com/docs/en/security)
