# Chapter 17: REST, Versioning, Review: the Exam Beyond Claude

**Summary**: *The third-heaviest skill on this paper barely mentions Claude. REST contracts, JSON that tolerates a field it has never met, concurrency you bound yourself, prompts held to the same review gate as code, a build that checks what genuinely regresses, and a migration that pins behaviour before it changes it. Revise the model alone and these arrive as ordinary engineering questions in unfamiliar dress, with nothing model-specific to answer them from. Getting one wrong costs exactly what getting a model question wrong costs, and there are more of them.*

## Seven Named Topics in This Skill, and None of Them Is About Claude

Here is the blueprint clause this chapter owns, with nothing removed. **REST APIs**. **JSON**. **Asynchronous programming**. **Version control**. **Lifecycle integration**. **Code review**. **Refactoring**, at small and large scale.

Not one of those is a **Claude topic**. All of them are what a **competent engineer** does on an **ordinary Tuesday**, and together they are the **third-heaviest skill** on the paper.

That combination produces a **revision failure** with a very predictable shape. Somebody reads every **published page** about the model, arrives well prepared on windows and tokens and tool schemas, and meets a scenario about **forty call sites**, a **deprecated parameter** and a **six-month window**. The scenario is about **migration risk**. Claude is the **set dressing**. A developer who has been revising the set dressing has nothing to bring.

The practices themselves do not change because a model is in the system. What changes is which **properties of the dependency** each practice has to account for, and there are five worth holding. Call the thing on the other end of the call a **model-shaped dependency**: **remote**, **variable** in what it returns, **billed by the call**, **versioned on a schedule** nobody in the room controls, and perfectly capable of **failing while reporting success**.

| Property of the dependency | What it changes about ordinary practice |
|---|---|
| Output varies between runs on identical input | A test asserting exact equality reports noise rather than regressions |
| Every call costs money in proportion to its size | Token spend becomes a regression class, alongside latency |
| Versions retire on the publisher's schedule | Migration is scheduled engineering work, not maintenance |
| A failure can arrive as a well-formed answer | Error handling that reads only the status code misses a seam |
| Behaviour is defined in prose rather than in control flow | Prompt text is code and has to live where code lives |

Read those **five rows** as the agenda for **everything you revise** on this skill. Every section that follows is a **standard practice** with one of them fed into it.

## A Contract Is What Is Promised Plus What Is Reserved

Chapter 2 sets out the parts of a call and chapter 3 sets out the endpoints. The **engineering question** underneath both is narrower and it is the one scenarios reach for: what does this **contract promise**, and what has it **reserved the right to change**?

The promised half is unremarkable and that is the point. Claude is reached over a **RESTful API**. **Requests and responses are JSON**, and every response **carries an identifier** for that single call.<sup>[1]</sup> **Statelessness** comes with the style rather than with the model: the **service keeps nothing between calls**, so continuity is a property your code supplies or does not have.

Versioning is where the reservation starts. An **`anthropic-version` header is required** on every request. The **latest version is recommended**, and earlier versions are treated as **deprecated** and may be unavailable to new users.<sup>[2]</sup> A **pinned version** does not freeze the service. It makes a change something **somebody chose**.

Then the reservation itself, which is the part that decides scenarios. **Errors come back as JSON** carrying a **type, a message and a request identifier**, and the documented versioning policy says plainly that the set of values inside those objects may **grow over time**.<sup>[3]</sup> A publisher who says that has told you how to write the consumer. **Branch on the values you know**. Keep a **default arm** for everything else. Log the identifier rather than the prose.

The **exhaustive switch over somebody else's enumeration** is the **anti-pattern**, and it is written by careful people. Every **documented value is handled**, nothing falls through, the reviewer approves it because it looks complete. It is complete against a list the publisher **reserved the right to extend**, so the first new value lands in a branch that does not exist and the failure surfaces somewhere unrelated. The same reasoning kills error handling that **matches on message text**: message wording is not the contract, and the **typed exception classes** are.<sup>[3]</sup>

**JSON discipline** is the small version of the same rule. **Read fields by name**. Ignore what you do not recognise rather than rejecting it. Treat a **field's absence** and a **field's null** as the two different statements they are.

## Concurrency Is a Property of Your Process, Not a Feature of the Service

The word **asynchronous covers two unrelated things** on this syllabus, and scenarios separate them on purpose. One is a service that accepts a pile of work and returns results later, which chapter 3 covers and which buys a **discount rather than speed**. The other is the **programming model inside the caller**, and that is what this clause is about.

A thread waiting on a network round trip is doing nothing at all. **Non-blocking work starts many calls**, **keeps the process useful** while each one is in flight, and handles every result as it lands. Concurrency here is **overlapping waiting**, not **parallel computation**, which is why a single process can hold **hundreds of outstanding calls** without owning hundreds of cores.

**Four consequences** follow, and **high-volume scenarios** turn on all of them.

The **limit that governs is the one you set**. A comprehension that launches ten thousand calls at once does not discover its ceiling gently. It meets the **rate limit as a wall**, and every rejected call spends retry budget that the calls behind it now need. A **bounded pool of workers** converts a **burst into a rate**, which is the shape the service is priced and provisioned for.

**Completion order is not submission order**. The moment work overlaps, a result arrives with no positional relationship to the request that caused it. Each unit of work **carries a key**, the key comes back with the result, and you **join on the key**. An index into a shared list is where the wrong customer record gets updated by code that **raised no error**.

**Retries and concurrency multiply**. Retry budget is one **shared resource** across the whole process. Workers each retrying independently against a limit they are collectively causing will **hold that limit open indefinitely**, and the log will show a service problem rather than a client one.

A **retried call has to be safe to repeat**. Concurrency turns a rare retry into a routine one, and a retry stays harmless only while repeating the call is harmless. Where a unit of work writes a record, charges an account or sends a message, it needs a **key the receiver recognises as already handled**. Without one, the **same record gets created twice** by a client behaving correctly and a service behaving correctly, and neither log shows an error.

The **unbounded fan-out** is the **named failure** and it is almost **never a decision**. Somebody writes the sequential version, it is too slow, and the fix that **requires the least typing** is to launch everything. Where the work is high volume and nobody is waiting on any individual answer, the cheaper move is the service's own **asynchronous submission** rather than any concurrency you write yourself. **Volume without a waiting caller** is the cue.

## Version Control Holds Everything That Decides What Ships

The test is one sentence and it does more work than any list of file types. If changing it **changes what the system does in production**, it **belongs in the repository with the code**, tracked and reviewed on the same terms.

That **sentence is uncomfortable**, because it catches artifacts you almost certainly keep somewhere else.

| Artifact | Where it usually ends up | Why the repository is the right place |
|---|---|---|
| Prompt text | A string in a handler, or a database row nobody diffs | It defines behaviour, so an edit is a behaviour change with no commit behind it |
| Model identifier | Inline at the call site | It fixes capability, price and a retirement date, and it appears in more than one file |
| Tool and output schemas | Alongside the code, usually correctly | They are the contract downstream consumers were written against |
| Evaluation sets and thresholds | A spreadsheet somebody owns | A pass threshold means nothing without a fixed set behind it |
| Credentials | Wherever they were first pasted | They do not belong there at all, and chapter 22 has the reason |

Prompts are the row that gets argued about, so take it head on. A prompt is prose, and **prose feels like documentation**. *It is not documentation.* It is **behaviour-defining code**: the same input against two **prompt versions produces two different systems**, and nothing in the deployment record distinguishes them if one of those versions **never passed through a commit**. One home, a **tracked history**, and a **review gate no weaker** than the surrounding code's. A team that keeps a **hand-written change log** instead has built a record that agrees with reality only while somebody remembers to update it.

Claude Code's own configuration shows the general rule with a documented instance. A **project settings file** reaches a teammate's clone and a cloud session only if it is **committed to version control**, and the local variant beside it is **added to git excludes** the first time it is written so it stays out of commits.<sup>[4]</sup> **Shared configuration** is shared by being committed. **Machine-specific configuration** is **deliberately kept out**.

The **prompt edited in production** is the **anti-pattern**, and its tell is a team that can describe what the system does today and cannot say what it did **last month**.

## A Pipeline Stage Earns Its Place on Two Conditions, and Most Fail One

Integrating this work into an **existing lifecycle** mostly means **answering one question**: what should your **build check** that it does not check already? Chapter 19 owns how **evaluation sets and tests** are built and chapter 23 owns the phases. What belongs here is the **selection rule**.

A stage **earns its place on two conditions** at once. It has to **catch a regression** that would otherwise ship, and its **green result** has to survive an answer that comes back **worded differently** and still right. Both halves. **Three stages** clear both on this class of system.

| Stage | The regression it catches | What keeps it green on a reworded answer |
|---|---|---|
| Scored evaluation against a versioned set | Quality falling after a prompt, model or configuration change | A score threshold, not a required output |
| Contract tests over schemas and parsers | Tool schemas, output shapes and downstream consumers drifting apart | Shape and field agreement, independent of wording |
| Token and cost regression check | A change that quietly multiplies tokens per request | A budget with a stated tolerance |

Now the two stages candidates reach for, each **failing one half**.

**Byte-for-byte comparison** against a **stored response fails the second condition** and fails it immediately. Every acceptable rephrasing trips it, so it reports on runs where nothing is wrong, and a check that cries on green gets **switched off inside a week**. The **exact-match assertion** is the **anti-pattern**, and switching it off is the rational response to it rather than a discipline failure.

**Automatic remediation fails the first condition** in a subtler way. A pipeline that rewrites the prompt when a score drops and **deploys the rewrite unreviewed** has not caught the regression. It has **deleted the signal**, along with the human judgement that the failing score existed to summon.

There is a **reproducibility problem** sitting underneath all of this, and it bites hardest when the pipeline runs Claude Code itself. A **non-interactive run with `-p` loads the same context** an interactive session would, including anything configured in the **working directory or the home directory**, and it **runs a project's hooks** and connects the servers in its configuration with no **trust dialog**. Adding **`--bare` skips that discovery** entirely, which is what gives a scripted run the **same result on every machine**, and **`--output-format json` returns a payload** a script can branch on.<sup>[5]</sup> The general rule under the flag: a build step that reads whatever the host happens to have configured is not reproducible, and the symptom is a job that passes on one runner and fails on another with no change in your code. The code is not where anybody looks.

The same principle governs the pipeline's own **trigger surface**. Claude can run inside **repository workflows**, responding to a **mention on an issue or pull request**, or given a prompt to run on any repository event, authenticating from a repository or **organisation secret**.<sup>[6]</sup> That is integration into an **existing lifecycle** rather than a lifecycle of its own.

## An Automated Reviewer Produces Findings; the Merge Gate Is Still Something You Build

The managed review service is worth reading closely because its documented behaviour is exactly the distinction scenarios test. Reviews run when a **pull request opens or on each push**, findings are posted as **inline comments** on the lines they concern, and a check run appears alongside the other checks. That check run **always completes with a neutral conclusion**, so it **never blocks a merge** through **branch protection**. Gating on it means reading the **severity breakdown** out of the check run inside your own pipeline and deciding there.<sup>[7]</sup>

Read that as the general shape rather than as one product's configuration. A **reviewer, automated or human, produces findings**. A **gate is a rule** about which **findings stop a change**. They are **separate mechanisms**, and a team that installed the first and believed it had the second has a review nobody is obliged to act on.

Which leaves the reviewing itself, and the shape of a Claude-powered change that a review has to catch. Take the ordinary case. A teammate opens a pull request against your service, the code works, and **three things are missing**. The **prompt and the model** are **hardcoded inline**. There is no **handling for an error** from the remote service. There are no **tests over the integration**.

The temptation is to rank those and let the smaller two through. They do not rank, because each one **fails in production on its own** and for a **different reason**.

- **Hardcoded configuration** means the prompt and the model cannot **change without a deploy**, and neither is under review as a thing that changes.
- No **error handling** means the first rate limit, timeout or overload becomes an **unhandled exception**, against a dependency that will certainly **return errors**.
- Without tests, **nobody can demonstrate** later that either of the first two was **ever repaired**.

So the answer a review has to give **asks for all three before the merge**, and every **partial answer** lets exactly one of them through. **Approving because the working path works** pushes every one of them into production. **Deferring the error path** to a later release holds back the only gap that breaks while users are on it. And approving with a **promise to fix it yourself** is the interesting wrong answer, because it looks generous: it removes the purpose of the review, moves the work rather than resolving it, and leaves the author no better at writing the next one. The next request repeats it.

Underneath the specific three sits what makes any **change reviewable** at all. A change that **does one thing**. An **example that shows it running**. A **test that proves the behaviour** rather than describing it. A **short statement** of what the change **assumes about its environment**. A reviewer who has to reconstruct the author's intent before evaluating it will do that last, which is why a contribution you know to be correct can sit untouched for weeks.

## Refactoring Splits by Whether Behaviour Has to Be Pinned First

**Small-scale refactoring changes structure** and **leaves behaviour alone**. **Extract a hardcoded value into configuration**, name a seam, split a function that quietly grew a second job. The correctness test is that **nothing observable moved**, which is what makes this work safe to do continuously and cheap to review. A reviewer who believes that claim only has to check the shape of the change. That is the whole saving.

The claim is also the value, and it is why the two scales stay apart in the commit history. A **restructuring and a behaviour change** landing together produce a record where **nobody can say afterwards** which half caused the symptom. The **mixed commit** is the **anti-pattern**, and it is written by people in a hurry rather than by people being careless. Separating it costs one **extra commit** and buys the ability to **revert half of the work**.

**Extracting configuration** is the **small-scale refactor** this material asks for most, because it **repairs the review defect** from the previous section and it is a precondition for the migration in this one. A value you can **change without a deploy** is a value you can change one environment at a time.

**Large-scale refactoring** is a different activity wearing the same word, and the exam reaches for one instance of it repeatedly: a **deprecated parameter**, used in **dozens of places**, with a **deadline attached**.

The deadline is real and it is documented. Customers with active deployments get at least **sixty days' notice** before a **public model retires**. Requests to a **retired model fail outright**. And an **audit of API usage** exists precisely so you can find where the deprecated thing is still called.<sup>[8]</sup>

The deadline is not the risk. **Semantics** are.

Two documented migrations make the point better than an abstraction does. **Manual extended thinking** with an **explicit token budget** is **unsupported on the newest models** and returns an error, and the budget parameter has no **direct replacement** at all: **thinking is adaptive**, and the **effort parameter** is an **output-level control** rather than a thinking budget.<sup>[9]</sup> **Sampling parameters** went the same way, deprecated from a stated generation onward, returning an error when set to a non-default value, and removed outright from one SDK so that passing them **raises a type error** rather than being ignored.<sup>[8]</sup>

In both cases the **replacement is not a rename**. It acts on a different thing, so a **mechanical substitution** across forty call sites **changes behaviour** at every one of them and tells you at none.

Which fixes the order of work. **Pin the behaviour, then change** it. **Regression tests** covering what the parameter currently does, written while the **old parameter still works**, because afterwards there is nothing left to observe. Then **migrate in batches** small enough to attribute a failure to, validating each batch against those tests before starting the next.

Two **named failures** cluster here, and both are answers to a question **nobody asked**.

The **compatibility wrapper keeps the old signature alive** while the **meaning changes underneath** it. Every call site keeps compiling, the diff looks tiny, and the **semantic change** has simply been moved into one function where no test is watching it. The parameter still disappears on the **published date**.

The **deadline-day migration** is the **honest version** of the same avoidance. Work is scheduled for the end of the window, which **concentrates every semantic change** into one **unvalidated push** at the moment there is no **slack left** to recover in.

Both **trade a real risk** for a schedule that feels tidier.

## How Wrong Answers Are Built on This Material

Wrong answers on engineering practice are **unusually hard to eliminate**, because they describe things **competent teams** genuinely do. Wrappers get written. Deadlines get planned around. Reviewers do sometimes fix small things themselves. **Four assumptions** produce most of them.

- **Schedule risk substituted for semantic risk**. The option **addresses when work happens** rather than whether the **change is safe**: a **wrapper**, a **batching plan**, a **later release**. The **test is quick**. Ask whether the option would still be the **right answer** if the **deadline were a year away**. If it would, it **never addressed the risk** the **scenario described**, and it becomes correct only where the replacement really is a **drop-in rename**.
- A **check with no tolerance for rewording**. **Exact string equality**, a **fixed response length**, a **stored output compared literally**. Each is a real check that **catches real changes**. Each also **fires when the answer is right** and **phrased differently**, so it **reports noise and gets disabled**, leaving the system worse covered than before it was added. Ask what a **correct-but-reworded answer** does to the check being proposed.
- **Judgement removed** at the point where **judgement was the product**. **Auto-rewriting a prompt** on a **failing score**, approving a change with a **promise to repair it privately**, treating an **automated reviewer's presence** as a **merge gate**. Each **converts a signal meant for a human** into an **action by a machine**. Ask which human was **supposed to decide**, and whether the option leaves them anything to decide with.
- A **partial repair accepted** because it is the largest one. **Three independent gaps** are named and the option **closes the biggest**. It is a **genuine improvement**, which is exactly why it **survives a first read**. Where each **gap fails in production** on its own, the **answer is the set**, and an option **addressing two of three** is **wrong by the same rule** as one **addressing none**.

**Match on what the scenario** is doing rather than on the technology it names.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| A parameter is deprecated and its replacement behaves differently | Cover current behaviour with tests, then migrate in batches you can attribute | A wrapper that keeps the call shape, or one swap at the deadline |
| A pipeline is being designed for a Claude-backed service | Stages that catch real regressions and stay green when wording shifts | Literal output comparison, or rewriting and deploying automatically |
| A change hardcodes configuration, skips error handling and has no tests | Request all three before merge, because each fails independently | Approving and repairing it yourself, or deferring the error path |
| The same job has to behave identically on every machine that runs it | Load nothing the host happens to have, and state the context explicitly | Assuming a scripted run sees what a developer's session sees |
| Prompts are edited by hand and nobody can say what changed when | One home, a tracked history, and review on the code's terms | A hand-kept change log, or review left optional for prose |
| Volume is high, units are independent, nobody waits on any one result | Bounded concurrency with a correlation key, or asynchronous submission | Launching every call at once, or a sequential loop defended as simple |
| A retry is firing and the same action has landed twice | Make the unit of work safe to repeat, keyed so the receiver knows it | More retries, fewer retries, or backoff, none of which change a repeat |
| An automated reviewer already comments on every pull request | Read its findings in your own pipeline and decide what stops a merge | Treating the reviewer's presence as the gate itself |
| A response field gains a value the code was never written for | Branch on known values and default the rest | An exhaustive switch, or matching on the error message text |

One **habit carries** past the table. For each option, name which of the **five properties** of a **model-shaped dependency** it accounts for, and which one it **quietly assumes away**. Wrong options on this material are almost always **correct engineering** aimed at a dependency that **behaves better** than this one does.

## What to Remember

- A **model-shaped dependency has five properties**: it **varies between identical runs**, **bills by the call**, retires on the **publisher's schedule**, **fails while returning success**, and has its **behaviour defined in prose**. Ordinary engineering practice is unchanged; what changes is which **property each practice must absorb**.
- A **contract** is what is **promised plus what is reserved**. Errors arrive as **JSON** with a **type, a message and a request identifier**, and the **versioning policy reserves the right** for those value sets to **grow**, so a consumer **branches on known values and defaults the rest** and never matches on message text.
- An **`anthropic-version` header is required on every request**; the **latest version is recommended** and **earlier ones are deprecated** and may be closed to new users. Pinning does not freeze the service, it makes an **upgrade a decision somebody made**.
- **Concurrency is overlapping waiting, not parallel computation**, so one process can hold many calls in flight. The **governing limit is the one you set**: **bounded workers** turn a burst into a rate, while **launching everything at once** meets the **rate limit as a wall** and **burns retry budget** the queued work needs.
- A **retried call must be safe to repeat**, because concurrency makes a rare retry routine. Work that **writes, charges or sends** carries a **key the receiver recognises as already handled**; without one the **same record is created twice** by a correct client and a correct service.
- **Completion order is not submission order**. Every unit of concurrent work carries a **correlation key that returns with its result**, because joining on position updates the wrong record and **raises no error** doing it.
- Anything whose **change changes production behaviour** belongs in **version control**: **prompt text**, **model identifiers**, tool and **output schemas**, **evaluation sets** and thresholds. **Credentials** do not, and configuration is **shared by being committed** while machine-specific settings are **deliberately excluded**.
- A **prompt is behaviour-defining code**, meaning **two prompt versions are two systems** on the same input. It needs **one home**, a **tracked history**, and a **review gate no weaker** than the surrounding code's; a **hand-kept change log** matches reality only while somebody maintains it.
- A **pipeline stage earns its place on two conditions**: it **catches a regression** that would otherwise ship, and its **green result survives an answer reworded** and still correct. **Literal output comparison fails the second** and gets disabled; **automatic remediation** fails the first, deleting the signal it was meant to raise.
- **Three stages** fit this class of system: a **scored evaluation against a versioned set** with a threshold, **contract tests** over schemas and downstream parsers, and a **token and cost regression check**.
- A **scripted run** that loads whatever the **host has configured is not reproducible**. Non-interactive Claude Code **reads the working directory and home directory** by default, including hooks and configured servers, and a **bare mode skips that discovery** so every machine produces the same result.
- A **reviewer produces findings**; a gate **decides which findings stop a change**. The managed review posts inline comments and a **check run that always completes neutral**, so it **never blocks a merge** on its own; gating means **reading its severity output** in your own pipeline.
- **Three gaps in a Claude-powered change** each **fail independently**: **configuration hardcoded** cannot change without a deploy, **no error path** turns the first rate limit into an unhandled exception, and **no tests** leave both unverifiable. The answer **requests all three**, and **approving while fixing it yourself** moves the work instead of resolving it.
- What makes a **change reviewable**: it does **one thing**, an **example shows it running**, a **test proves the behaviour**, and a **short statement names what it assumes** about its environment. A reviewer forced to **reconstruct intent** reviews last.
- **Restructuring and behaviour change belong in separate commits**. Landed together they leave a **history nobody can attribute** a symptom to, and separating them costs one **extra commit**.
- **Small refactoring changes structure** and nothing observable; **large refactoring changes semantics**, which is why **behaviour is pinned in tests** before it is changed and migrated in **batches small enough to attribute** a failure to.
- A **deprecated parameter whose replacement acts on something else** is not a rename. Anthropic gives at least **sixty days' notice** before a public model retires, **requests past retirement fail**, and a **usage audit locates remaining call sites**. A **compatibility wrapper keeps the signature** and **loses the semantics**, and the **deadline-day migration** concentrates every semantic change into one push with no slack behind it.

## Endnotes for this chapter

1. The Claude API is a RESTful API at `https://api.anthropic.com`, requires `content-type: application/json` on every request, and returns a `request-id` response header carrying a globally unique identifier for that request. Anthropic, *API overview* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/overview](https://platform.claude.com/docs/en/api/overview)

2. API requests must send an `anthropic-version` request header, for example `anthropic-version: 2023-06-01`; Anthropic recommends using the latest API version whenever possible, and previous versions are considered deprecated and may be unavailable for new users. Anthropic, *Versions* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/versioning](https://platform.claude.com/docs/en/api/versioning)

3. Errors are always returned as JSON with a top-level `error` object that always includes a `type` and a `message` value, alongside a `request_id` field; in accordance with the versioning policy the values within these objects may expand and the set of `type` values is expected to grow over time; the official SDKs raise typed exceptions rather than returning raw JSON, and the documented guidance is to catch those typed classes rather than string-matching error messages. Anthropic, *Claude API errors* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/api/errors](https://platform.claude.com/docs/en/api/errors)

4. A project's `.claude/settings.json` reaches a teammate's clone and a cloud session only if the file is committed to version control; until then it is a file on one machine's disk. The adjacent `.claude/settings.local.json` is added to the user's global git excludes the first time Claude Code writes it, so that it stays out of commits. Anthropic, *Claude Code settings* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)

5. Passing `-p` runs Claude Code non-interactively, loading the same context an interactive session would, including anything configured in the working directory or `~/.claude`, and running the hooks in a project's `.claude/settings.json` and connecting the servers in its `.mcp.json` with no workspace trust dialog and no per-server approval prompt. Adding `--bare` reduces startup time by skipping auto-discovery of hooks, skills, custom commands, subagents, plugins, MCP servers, auto memory and `CLAUDE.md`, and is described as useful for CI and scripts where the same result is needed on every machine. `--output-format json` returns structured JSON with the result, session ID and metadata. Anthropic, *Run Claude Code programmatically* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless)

6. The Claude Code GitHub Action runs Claude Code inside a repository's workflows, responding to an `@claude` mention in a pull request or issue comment, or running automatically on any GitHub event when given a prompt, authenticating from a repository or organisation-level Actions secret. Anthropic, *Claude Code GitHub Actions* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/github-actions](https://code.claude.com/docs/en/github-actions)

7. Reviews trigger when a pull request opens, on every push, or on manual request; findings are posted as inline comments on the specific lines where issues were found, with a summary in the review body, and each review populates a check run alongside the repository's other CI checks. That check run always completes with a neutral conclusion, so it never blocks merging through branch protection rules; gating merges on findings requires reading the severity breakdown from the check run output in your own CI. Anthropic, *Code Review* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/code-review](https://code.claude.com/docs/en/code-review)

8. Once a model is deprecated all usage must be migrated to a replacement before the retirement date, and requests to models past the retirement date will fail; Anthropic notifies customers with active deployments and provides at least 60 days' notice before retirement for publicly released models; an audit of API usage is available to help locate remaining usage of deprecated models. The `temperature`, `top_p` and `top_k` parameters are deprecated from Claude Opus 4.7 onward and return a `400` error when set to a non-default value on those models, and the Python SDK from v1.0 removes them from its request types so that passing them raises a `TypeError`. Anthropic, *Model deprecations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/model-deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)

9. Manual extended thinking with an explicit `budget_tokens` value is not supported on the newest models and returns a `400` error; `budget_tokens` has no direct replacement because thinking is adaptive, and the effort parameter is a separate output-level control rather than a thinking budget. Anthropic, *Migration guide* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/migration-guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide)
