# Chapter 15: The Constraints Nobody States Decide the Design

**Summary**: *Every system you are asked to change already has properties nobody put in the request, and a cache dropped into a multi-tenant authorization path breaks one of them quietly, months later, once. This chapter covers the two sets of requirements that stand between a business direction and a design you can defend: the behaviours somebody can mark pass or fail, and the deployment constraints that surface only because somebody went and asked. Get the second set late and the repair is a migration, not a setting.*

## The Request Describes the Change and the System Describes the Constraints

A **ticket lands**. Put a **cache in front** of the **authorization module**, because the checks are slow and the page waits on them.

Read it twice and you still find **nothing missing**. There is a **component**, a **symptom**, a **mechanism** and an **owner**.

Now read the system instead of the ticket. **Several customers share** that deployment. Every **access decision** the module makes is written to a **log somebody audits**. The invalidation already in place does not behave the same way on **every path**, which the team knows and has never written down anywhere.

None of that was put there for the benefit of this change. It was true last quarter and it will be true after the cache ships. Call it a **standing constraint**: a **property of the environment** that **predates the request**, appears in **nobody's description** of the work, and **decides the design** anyway.

A **functional gap announces itself** the first time somebody runs the feature. A standing constraint violated by a cache announces itself when **one tenant** is served **another tenant's decision**, months later, once. **Silent, rare and expensive** is the exact profile of a defect that iteration never catches, because **iteration needs a signal** and this one gives none until it gives a very large one.

So your first move on a change of that shape is **neither code nor a design**. It is a **set of targeted questions**: what has to **stay true**, what **invalidates an entry**, what the **audit trail** must record, and how the module behaves when the **cache is wrong**. Anthropic makes the general form of the point about Claude itself, which needs the same thing for the same reason. Going straight to code produces code that **solves the wrong problem**, so exploration and planning are **separated from execution**.<sup>[1]</sup>

## A Goal You Cannot Fail Is Not Yet a Requirement

A **business problem** is a direction. Get invoices approved sooner. Stop the compliance team reading every document. **Both sentences** are **worth having**, and you can **neither build nor check** a system against either one.

A **functional requirement** says what the system must do, in enough detail that somebody can **mark it pass or fail**. **Deriving them turns** one direction into **several checkable behaviours**: flag each document as needing review or not, quote the clause that triggered a flag, leave a flagged document unapproved until a named reviewer clears it. Each of those can be **run and scored**. The **direction cannot**.

The **test is mechanical**, and it is the only one you need. Read the statement, then ask yourself what **observation would show it had been broken**. If nothing would, the statement is a **preference**.

Anthropic sets the same discipline for **success criteria**, and the worked pairs are the fastest way to internalise it. "Good performance" becomes "**accurate sentiment classification**". "Safe outputs" becomes fewer than **0.1% of outputs across 10,000 trials** flagged by the **content filter**.<sup>[2]</sup> The second version of each is **longer**, and length is not what you are buying. It **bought a line** the system can be held to at review.

The **requirement nobody can fail** is the anti-pattern here, and it is **seductive** because it is **agreeable**. Everybody signs off on fast, accurate and secure. **Nothing has been agreed**. Two humans read one sentence, **build different pictures** from it, and meet the difference at the review where one of them says this is not what was asked for.

## The Second Set of Requirements Sits on Other People's Desks

**Functional requirements arrive** on their own, near enough. Whoever wants the feature knows what it should do and will **tell you unprompted**.

**Infrastructure requirements bound the deployment** rather than the behaviour. They say nothing about what the system produces and everything about the **conditions** under which producing it is **acceptable**, and hardly any of them are anywhere in the business problem. They come out by **interrogation rather than by reading**, because each one is **obvious** to the human holding it and therefore **never said out loud**. **Four questions** cover most of the ground.

| The question | Who actually holds the answer | What a late answer costs |
|---|---|---|
| How fast, measured where the user is | Whoever owns the experience, and whoever runs the region | A model, a platform or a topology chosen against the wrong number |
| What volume, and what the peak looks like | Operations, plus whoever keeps the historical figures | Capacity and rate limits sized for the average |
| Processed where, stored where, under which regulation | Legal, compliance, the data protection function | A rebuild, because some of these settings are fixed at creation |
| Acting under whose credentials, auditable how | Security, and the owner of the system being acted upon | An audit trail that cannot answer the question it exists for |

The third column is the reason this work **belongs at the start** rather than in the middle. Four questions **cost you a conversation**. Their answers, **arriving late**, **cost you a migration**. What gets traded against a latency figure once you hold one is chapter 4's material; obtaining the figure at all is this chapter's.

One trap in that table looks like **diligence**. A **latency budget** tells you nothing whatsoever about which **tenant** may see which **record**. Performance targets and correctness rules are different kinds of statement, and **correctness inferred from a performance target** is **guessing** with a spreadsheet open.

## A Residency Rule Removes Options and Never Weighs Against Them

**Latency, scale, residency and identity** decide a deployment more often than anything else does, and they do not all behave the same way once **design starts**.

A **latency target** is a **threshold**. Candidate designs get measured against it, some clear it, and the ones that clear it are then compared on other grounds. A **residency rule** is **not a threshold** at all: it **strikes candidates off the list** before any comparison begins, and whatever survives is compared as though the rule had **never come up**.

Where **inference runs** and where **data is stored** are two **independent settings**. The first travels on the request: `inference_geo` takes `"us"` or `"global"`, can be set **per call** or as a **workspace default**, is supported from **Claude 4.6 onward** and returns a `400` on earlier models, and US inference is priced at 1.1 times the standard rate. The second does not travel at all. **Workspace geo is fixed at creation** and cannot be changed afterwards.<sup>[3]</sup>

Retention has the same shape and **bites harder**. **Zero data retention** is arranged **per organization** through an account team, and enabling it for one organization does **not extend** it to another under the same account. **HIPAA readiness** needs a **signed agreement**, and once enabled the **configuration is permanent** and **no administrator** can switch it off. A request carrying a feature the arrangement does not cover **fails** with a `400` rather than **degrading quietly**.<sup>[4]</sup> Partner platforms move the ground underneath as well: on Amazon Bedrock and Google Cloud the **cloud provider** is the **data processor**, so their **retention documentation** is the one that governs.<sup>[4]</sup>

**Residency treated as one factor to balance** is the **standard wrong answer** on this material, and it always arrives dressed as engineering judgement. A faster region. A cheaper platform. A simpler integration. None of the three is an argument, because a constraint of that kind decides which **options are admissible** at all, and the three being weighed had **stopped being options** before anybody weighed them.

## The Record Exists Because the Reviewer Was Not in the Room

**Requirements get written down** for one reason that survives contact with a busy team. Whoever **reviews a deployment choice** was not in the conversation where its **constraints came out**, and the review is where the choice has to be defended.

A short record does the job: the **behaviours the system must exhibit**, the **infrastructure constraints**, and which **regulation** or which **owner** each constraint came from. With it, your **platform choice** reads as a **consequence** of the requirements. Without it, it reads as **familiarity**, and familiarity is precisely what a reviewer **pushes** on.

That record has to **stay current**, because **configuration does not retro-fit** itself. **Permission policies** show the general behaviour cleanly: tightening a policy on an agent governs **sessions created after the change**, while sessions already running **keep the configuration** they started with.<sup>[5]</sup> Operating your own infrastructure carries the same asymmetry. Once session content reaches a **self-hosted worker** it is retained, redacted or deleted under the **operator's own policies**, and Anthropic has no visibility into what happens to it after delivery.<sup>[6]</sup>

None of this scales down to every ticket, and pretending it does is how the discipline gets abandoned wholesale. Anthropic's calibration is a good one to borrow: **planning earns its overhead** when the approach is **uncertain**, when the change touches **several files**, or when the code is **unfamiliar**, and a change whose **diff fits in one sentence** should skip it.<sup>[1]</sup> **Multiple tenants**, an **audit obligation** and **behaviour nobody describes** the same way twice clear that bar three times over.

## How Wrong Answers Are Built on This Material

Wrong answers on requirements **describe good engineering**. Prototyping, staged rollout, monitoring and reuse are all things **competent teams** do, which is why they survive a first read. What makes one wrong is what it **assumes has already been established**.

- The **unstated property assumed absent**. The option **treats the system** as though the only **things true** of it are the ones the **request mentioned**. Iterating toward a fix, shipping behind a flag and building a small version first all **belong to a system whose invariants are known**. **Each becomes correct** once the **change is reversible** and the **damage is loud and contained**.
- **Discovery relocated to after the build**. Monitoring, incident review and a canary release do **find constraints**. They find them by **violating one**. **Fair trade** when a failure is **cheap, quick and visible**; **bad trade** when it is **silent, rare and consequential**, which is the profile every **tenancy and audit scenario** is built on.
- A **constraint inferred instead of elicited**. **Reading correctness rules** off a performance target, off a neighbouring module's implementation, or off what the previous team wrote. **Copying an approach imports its assumptions** along with its **structure**, and an **authorization path** is where an **imported assumption does the most damage**.
- A **hard constraint weighed as a preference**. A **regulation**, a **jurisdiction** or a **retention obligation** offered as **one input to a tradeoff** against cost, latency or effort. Constraints of that kind **produce the candidate list**; they **never compete** inside it.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Properties the request never mentioned: several tenants, an audit trail, behaviour that varies by path | Elicit the invariants, the invalidation rules, the audit obligation and the failure modes first | Iterating, prototyping or shipping behind a flag, all of which learn the same facts from an incident |
| A goal stated as faster, better, accurate or seamless | Convert it into behaviours somebody can mark pass or fail | Accepting the goal as the requirement and moving on to mechanism |
| A regulation, a jurisdiction or a retention obligation anywhere in the setup | Treat it as the filter that produces the candidate list | Balancing it against latency or cost as though every input were commensurate |
| Functional behaviour fully specified and a platform decision due | Obtain latency, volume, residency and identity, then decide | Choosing on familiarity, existing tooling, or what shipped last time |
| Somebody proposes copying an approach from a neighbouring system | Check which of that system's assumptions actually hold here | Reuse justified by similarity of shape rather than shared invariants |
| A design is defensible inside the team and opaque outside it | Write the record, naming the source of every constraint | Documenting the decision without the constraints that forced it |

One habit carries past the table. For each option, ask what it **needs you to know already**, then check whether the scenario said that thing was **settled** or said that **nobody had established** it.

## What to Remember

- A **standing constraint** is a **property of the environment** that **predates the request**, appears in **no description** of the work, and **decides the design** regardless. **Multiple tenants**, an **audit obligation** and **inconsistent existing behaviour** are the three that recur.
- **Elicit before implementing** wherever a mistake would be **silent, rare and expensive**. Iteration **needs a signal**; a **tenancy or authorization defect emits none** until it emits a large one.
- A **functional requirement** names the **behaviour the system must exhibit**, written so somebody can **mark it pass or fail**. Test it by asking what **observation would prove it broken**; if none exists, it is a **preference, not a requirement**.
- **Infrastructure requirements bound the deployment**, not the behaviour: they fix the **conditions** under which producing the output is **acceptable**. They are **obtained by asking**, never by reading, because **each is obvious** to whoever **holds** it. The four: **latency measured at the user**, **volume at peak**, **processing and storage location** under a **named regulation**, and whose **credentials act**, with what **audit record**.
- A **residency, retention or identity rule** decides which options are **admissible**. **Speed, cost and familiarity** order what survives the rule and **never overturn** it.
- Where **inference runs** and where **data is stored** are **separate settings**. `inference_geo` is **per request** or **per workspace default**; **workspace geo is fixed at creation** and cannot be changed, so a residency answer **arriving late costs a rebuild** rather than a config change.
- A **performance target** implies **no correctness rule**. A **latency budget** says nothing about which **tenant** may see which **record**, and **inferring one from the other** is guessing.
- **Requirements are recorded** because whoever **reviews the decision** was **absent** when the **constraints were gathered**. The record holds the **behaviours**, the **constraints**, and the **source of each constraint**, which is what turns a **platform choice into a consequence** rather than a **habit**.

## Endnotes for this chapter

1. Letting Claude jump straight to coding can produce code that solves the wrong problem, so plan mode separates exploration from execution; planning is most useful when the approach is uncertain, when the change modifies multiple files, or when the code being modified is unfamiliar, and a change whose diff could be described in one sentence should skip the plan. Anthropic, *Best practices for Claude Code* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

2. Good success criteria are specific and measurable: "good performance" is replaced by "accurate sentiment classification", and "safe outputs" by less than 0.1% of outputs out of 10,000 trials flagged for toxicity by the content filter. Anthropic, *Define success criteria and build evaluations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/develop-tests](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)

3. Two independent settings govern where data is processed and stored. The `inference_geo` parameter accepts `"us"` or `"global"`, can be sent on a request or set as a workspace default, is supported on Claude 4.6 and later models and returns a `400` error on earlier ones, and US inference is priced at 1.1 times the standard rate across all token pricing categories. Workspace geo is set when a workspace is created and cannot be changed afterwards. Anthropic, *Data residency* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/manage-claude/data-residency](https://platform.claude.com/docs/en/manage-claude/data-residency)

4. Zero data retention is enabled per organization by an account team, and enablement does not automatically extend to other organizations under the same account. HIPAA readiness requires a signed business associate agreement, and once enabled the configuration is permanent and cannot be disabled by an administrator; a request that uses a feature outside the arrangement returns a `400` error. On Amazon Bedrock and Google Cloud's Agent Platform the cloud provider is the data processor, and those platforms' own retention and compliance documentation applies. Anthropic, *API and data retention* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/manage-claude/api-and-data-retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)

5. Permission policies are set in the agent's tools configuration and can be changed later by updating the agent; running sessions keep the toolset configuration they were created with, and updates apply to sessions created afterwards. Anthropic, *Permission policies* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/managed-agents/permission-policies](https://platform.claude.com/docs/en/managed-agents/permission-policies)

6. Under the shared responsibility model for self-hosted sandboxes, conversation content and tool outputs pass through the operator's worker and stay in the operator's environment, which makes retaining, redacting or deleting that data in line with the operator's own policies their responsibility; Anthropic has no visibility into what a worker does with session content once it has been delivered. Anthropic, *Security model* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security)
