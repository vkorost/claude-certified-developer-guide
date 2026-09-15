# Chapter 18: An Unpinned Version Is an Untracked Change

**Summary**: *Something regressed, three things moved since the last good state, and not one of them left a commit. That is the whole subject. This chapter makes every input to your system a value somebody chose and wrote down: the canonical model identifier rather than the pointer that advances on its own, a prompt version with a name a rollback can reach, plugin dependencies held inside a tested range, and each setting in the file whose reach matches who needs it. Get this wrong and the bill arrives later, as a regression nobody can attribute and nothing to roll back to.*

## Nobody Can Name the Change That Caused the Regression

A **summarisation service** starts returning a payload the **downstream parser cannot read**. It worked in March. The **release log is empty** for six weeks, so on the evidence in front of you, **nothing changed**.

**Three things changed** anyway. Somebody **edited the system prompt** where the running service reads it, to settle a complaint about formatting. A **plugin took a new release** from its marketplace overnight. And the model identifier in the call was a **convenience pointer** rather than a fixed one, so it began resolving to a newer snapshot on a schedule set outside your building.

Each of those **alters what the system produces**. None **left a commit**. So the review opens with **four humans guessing**, and the guess cannot be settled: no earlier state can be restored, and nothing records which change landed first.

Call the gap the **unattributable window**. It is the span between the **last state you can reproduce** and the state in front of you, holding every **change nobody wrote down**. Configuration management **keeps that window empty**. It buys nothing on an ordinary day and it is the only thing that pays on the day something regresses.

Anything deciding what your system produces is either **held at a value somebody chose** or is a **variable nobody declared**. The second kind is silent, and the rest of this chapter is five instances of it.

| What moves | How it moves with no commit behind it | What holding it still looks like |
|---|---|---|
| The model | A convenience pointer resolves to a newer snapshot | The canonical identifier, in a tracked file |
| The prompt | An edit made where the running system reads it | A named version, reviewed, the previous one kept |
| A plugin | Auto-update takes the marketplace's newest release | A declared range, resolved from a tag |
| Team configuration | The file exists on one machine and nowhere else | Committed beside the code it governs |
| The measurement | The set stays fixed while the traffic moves | A dated set, rebuilt from what is arriving now |

## A Model Identifier Names One Snapshot; a Convenience Pointer Names Whatever Is Newest

Every **Claude model ID** identifies a **pinned version**, and the model underneath **stays constant for the lifetime** of that ID.<sup>[1]</sup> From the 4.6 generation onward the IDs carry no date, and the **dateless ID** is the **canonical one for that release**: one fixed snapshot, **never updated in place**, with any newer version arriving under a new ID. Anthropic names the opposite belief, that a dateless ID routes to the latest or best-performing version, as a **common misconception**. Earlier models carry a snapshot date, and on the Claude API those also have **shorter aliases** pointing at the most **recent dated snapshot** for their minor version.<sup>[1]</sup>

The **format changed**. The discipline did not. **Write the identifier** the documentation **calls canonical**, and never the **shorter pointer** beside it. That pointer is convenient because it advances. Check the convention on the model page rather than reciting a shape from memory; it has **already moved once**.

Two facts bound what a pin buys you. **Weights are fixed** for a given ID and the **serving infrastructure** around it is not. Routing, **safety classifiers and sampling logic** can all change, and an infrastructure update occasionally produces small differences in **observable behaviour** on an ID that has not moved.<sup>[1]</sup> So **pinning narrows the set** of things that could have changed. It does not empty it. **Measurement accompanies pinning** rather than replacing it.

Every model ID also carries its own **deprecation and retirement schedule**,<sup>[1]</sup> and **partner-operated platforms set their own dates**, so one model can hold a different lifecycle status depending on where it runs.<sup>[2]</sup> A pin is a **dated commitment**. Chapter 17 has the **migration work** that falls due when the date arrives.

The **convenience pointer that reached production** is the anti-pattern, and it never looks like a decision. It is the shortest thing to type and it works on the day you type it. The **bill arrives when output shape shifts**: the **previous snapshot was never recorded**, so there is nowhere to return to and the repair becomes a hotfix on your parser. Keeping the prior identifier available is **half the value of pinning**, and it is the half teams skip.

## A Prompt Version Is the Only Thing an Incident Timeline Can Point At

Chapter 17 settles where **prompt text lives** and what gate it passes. What a version adds, once it is there, is a name, and that **name does two jobs** nothing else does. It lets a **timeline record which text produced** the output under investigation. It gives a **rollback a destination**. Without it, both become archaeology across editor history and memory.

Promotion is where a **version earns its keep**. A new prompt version reaches production the way a new model does. It runs against a **portion of traffic**, it is **scored against the baseline** the current version holds, and it is promoted or reverted on that number. Anthropic's guidance is that criteria must be **specific and measurable** before anything can be applied, and that the grading method **trades speed against nuance**: **code-based grading is fastest** and most reliable, **model-based grading is flexible** once its reliability has been tested, and **human grading** is the most flexible and the **slowest**.<sup>[3]</sup> Chapter 19 owns building that set. Here it becomes a gate.

One property separates prompt versioning from model pinning. A **prompt has consumers**, and a version with several of them carries **several sets of expectations**. A **rollout that skips assessing** them is betting that none depends on a property this version changes. Three near-misses circle that gap. **Telling downstream owners** a change is coming puts it on their calendar and **discovers nothing about its effect**. A **clean staging run** exercises the properties staging covers and stays silent about the rest. Shipping to the assessed consumers now and the others later leaves one **application answering under two prompt versions**, a second unattributable difference rather than half of one.

The **prompt carrying no version number** is the anti-pattern. Its tell is a team that can show you the **current text** and cannot produce the previous one.

## Four Configuration Surfaces, and the Reach That Picks Between Them

The **decision rule** is one sentence and it settles most of these scenarios on its own. Put a setting in the **file whose reach matches the set of people** the setting has to be true for.

| File | What it carries | Who it reaches |
|---|---|---|
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Standing instructions for the project | Everyone who clones it, through version control |
| `CLAUDE.local.md` | Instructions for one person's own copy | One machine, kept out of commits |
| `.claude/settings.json` | Permissions, hooks, telemetry, enabled plugins | Teammates and cloud sessions, once committed |
| `.claude/settings.local.json` | One person's exceptions in one project | One machine, excluded from commits automatically |
| `.mcp.json` in the project root | Servers the project needs in order to work | Everyone who clones the project |
| `managed-settings.json` and its console equivalent | Organisation policy | Every machine the organisation deploys it to |

Those reaches are **documented rather than conventional**. A project settings file arrives in a **teammate's clone** and a cloud session **only once it is committed**, while the local file beside it is added to your **global git excludes** the first time Claude Code writes it.<sup>[4]</sup> Project instructions travel the same way, which is why that file **carries standards and not preferences**.<sup>[5]</sup> Of the three server scopes, only **`.mcp.json` in the repository root** reaches anybody else; local and user scope both write to `~/.claude.json`.<sup>[6]</sup> Organisation policy goes to a system path on every machine.<sup>[7]</sup>

Set one key in more than one of them and the value comes from the **highest level that sets** it: **managed settings**, then **command line arguments**, then **project local**, then **shared project**, then **user settings**. List keys such as the permission arrays **merge across files** instead of overriding, so each file adds entries rather than replacing another's.<sup>[4]</sup>

The **shared dependency parked in personal configuration** is the anti-pattern, and it is **invisible to whoever created** it. A server every session needs, **registered at user scope** on the machine where it was first tried, works flawlessly there and exists nowhere else. The compensations are worse than the gap: a **setup procedure in a README**, which makes correctness depend on each person performing it identically, or a synchronisation step invented to copy personal files between machines. Run the mistake the other way and it is equally wrong. An experiment nobody else asked for, **committed to the project file**, is now everybody's.

Instruction files carry one more precision. They **load into the context window** at the start of every session and are treated as context rather than as **enforced configuration**.<sup>[5]</sup> A rule you write there **guides Claude** and **guarantees nothing**.

## A Plugin Dependency Follows the Newest Release Until Somebody Constrains It

By default a plugin dependency **tracks the latest available version**, so an **upstream release** can change what runs under your plugin with no warning and no action on your side.<sup>[8]</sup> A **declared version constraint** holds that dependency inside a **tested range** until the team decides to move.

The mechanics matter because scenarios describe them without naming them. Constraints are declared in the **`dependencies` array** of a plugin's **`.claude-plugin/plugin.json`** and **resolve against git tags** on the repository hosting the dependency, tagged as **`{plugin-name}--v{version}`**. Where several installed plugins constrain the same dependency, the **ranges intersect** and resolution takes the **highest version satisfying** all of them, so **auto-update keeps working** inside the allowed range.<sup>[8]</sup> Which plugins are switched on has its own key, **`enabledPlugins`**, keyed by **plugin name and marketplace** and settable in any settings file.<sup>[9]</sup>

Three symptoms then collapse into one cause. A team that **never recorded a version** cannot say where a regression came in, cannot **hold the working version still**, and has nothing to return to. All three follow from the **missing record**, so the repair has to **create the record**.

The **upgrade schedule mistaken for a version record** is the anti-pattern and the **confident wrong answer** here. Moving everything to the newest release monthly **feels like control**. It raises the rate of unattributed change and **writes nothing down**. Waiting for authors to announce breaking changes fails the same way, by leaving the record in somebody else's keeping.

## Environments Differ on Purpose, and the Difference Has to Be Declared Somewhere

Development, staging and production running **different model identifiers**, different **prompt versions** and different **plugin sets** are not yet a problem. **Staging exists** in order to hold something production does not. The defect appears when **nobody can produce the list**, at which point a deliberate difference and an accident look identical.

One **tracked place**, carrying the **same keys for each environment**, **converts a difference into a declaration**. Every difference is then either in the file, where somebody chose it, or it is **drift**. Reading two of them side by side separates the categories in a minute.

Two repairs sound like that one and are not. **Standardising every environment to end the drift** erases differences that exist for a reason, and takes **staging's purpose** with them. **Unpinning until the environments agree** removes a **control you need** and puts behaviour changes back outside the commit history. A third belongs beside them: moving the values into **application code** ties every configuration change to a release, and configuration and code change on unrelated schedules.

## Restoring Measurability Comes Before Improving Anything

Here is the situation this material returns to. A feature shipped a year ago and has been serving traffic ever since. Everyone who built it has moved on, **prompts were edited in place** more than once, the **evaluation set** has not been touched since launch and the **model identifier was never fixed**. Somebody proposes **upgrading to a newer model**. It is the **wrong opening move** for a mechanical reason: it adds a change to a system whose **current behaviour has no recorded value**, so the result is as unattributable as everything before it.

```
1  Pin the moving inputs      canonical model ID, plugin ranges, configuration in one tracked place
2  Give the prompts as they stand a version, whatever anyone thinks of the wording
3  Rebuild the measurement from recent production traffic rather than from the launch assumptions
4  Change one thing, and compare it against what steps 2 and 3 recorded
```

Step 2 draws the argument, so take it head on. The prompts in front of you **encode everything the feature was taught** over a year, in **edits nobody documented**. **Rewrite them wholesale** and that record is gone, along with the baseline that would have shown what the rewrite broke. Step 3 attracts the mirror-image mistake. **Deleting the stale evaluation set** removes a weak measurement and **supplies no replacement**, leaving you less than you started with. A stale set is evidence about the launch. **Rebuilding from current traffic** makes it evidence about now.

## How Wrong Answers Are Built on This Material

**Wrong answers on configuration management** describe things that responsible teams genuinely do. **Upgrade schedules** exist. **Setup guides** get written. Downstream owners do get told. What separates one of these from the right answer is a **single assumption**, and four of them cover almost everything.

- A **routine offered in place of a record**. The option **supplies a process** rather than a value: a **documented setup procedure**, a **monthly upgrade**, an agreement that **authors will announce breakage**, a **change log kept by hand**. Every one depends on a **human performing it correctly** each time, and none of them produces something **two states can be compared against**. Ask what the option would let you **look up six weeks later**, and from where.
- A **control removed to silence the symptom**. **Unpinning so environments agree**, **deleting the stale evaluation set**, **dropping the component that broke**. Each genuinely ends the complaint being made. Each also removes the thing that would have made the **next occurrence attributable**. Ask whether the option leaves you **fewer declared variables** than you had.
- **Communication supplied where assessment was missing**. **Notifying downstream owners**, **booking a window**, **staging the rollout**, **deferring the consumers** nobody has looked at. All four are **competent handling** of a change whose **effect has not been established**. Ask whether the option **generates new information** about what the change does, or only about when it happens.
- **Improvement attempted before a baseline exists**. **Upgrading the model**, **rewriting the prompts**, **adding a capability**, on a system **nobody has measured**. These read as initiative, which is why they **survive elimination**. Ask what the result would be compared against.

Match on what the scenario is doing rather than on the **components it names**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Something regressed and several things moved since the last good state | Make versions explicit and pinnable so the change can be attributed and reversed | Upgrading more often, or removing the component that broke |
| Configuration a whole team depends on lives on one person's machine | Move it into the project file that travels with the repository | A documented manual setup step, or a copy in each personal file |
| A personal experiment sits beside a shared requirement in one place | Split them by reach: the shared thing with the project, the experiment with the user | Merging them, or synchronising personal files between machines |
| Prompts were edited in place and the evaluation set predates the current traffic | Pin the inputs and capture current behaviour before changing any of it | Upgrading first, rewriting the prompts, or deleting the old set |
| Each environment runs a different model, prompt and plugin set | Consolidate into tracked configuration that keeps the differences declared | Making the environments identical, or unpinning so they agree |
| A prompt change has several downstream consumers | Assess the effect on each consumer, then coordinate on what that finds | Announcing the change, or treating a clean staging run as the assessment |
| A dependency changed overnight and nothing in the repository moved | A declared version range resolved from a tag | Trusting release notes to arrive before the breakage does |
| Instruction and settings files are being set up for a new project | Version them with the code they govern, reviewed on the same terms | Duplicating them per environment, or keeping them in a separate repository |

One **habit carries** past the table. For every option, ask what you could **read back six weeks from now**, and where you would read it. An option that only asks people to be careful leaves you **nothing to read**.

## What to Remember

- The **unattributable window** is the span between the **last reproducible state** and the current one, holding every **change nobody recorded**. **Configuration management keeps it empty**, and it pays only when **something regresses**.
- Anything deciding what the **system produces** is either **held at a chosen value** or is an **undeclared variable**. **Five things move**: **model identifier**, **prompt text**, **plugin versions**, **team configuration**, and the **evaluation set** the traffic drifts away from.
- A **Claude model ID** identifies a **pinned version** that **stays constant** for that ID's lifetime. From the 4.6 generation the **dateless ID is canonical**, mapping to one **fixed snapshot**; an updated version **ships under a new ID**. **Earlier models carry a snapshot date**, and their **shorter API aliases advance** to the newest snapshot for that minor version.
- **Pinning bounds the variables**; it does not remove them. **Weights are fixed per ID** while **routing, safety classifiers and sampling logic** can change, so **observable behaviour can shift** on an ID that has not moved.
- Every model ID has its own **retirement schedule**, and **partner platforms set their own dates**, so a **pin is a dated commitment** and the same model can **differ in lifecycle status** by where it runs.
- **Keep the previous pinned identifier** available. Without it a **regression becomes a hotfix** instead of a **rollback**, which is **half the value of pinning** and the half most often skipped.
- A **prompt version** gives an **incident timeline something to name** and a **rollback a destination**. **Promote on a score** against the **current baseline**, not on a reading of the new text.
- A prompt version with **several consumers** carries **several sets of expectations**. **Announcing, scheduling and staging** are not **assessment**, and shipping to some consumers now leaves one **application answering under two versions**.
- **Scope follows reach**: project instructions and `.claude/settings.json` **reach teammates once committed**; `CLAUDE.local.md` and `.claude/settings.local.json` **stay on one machine**; **`.mcp.json`** in the repository root carries **servers the project requires**; user-scope servers in `~/.claude.json` **reach nobody else**.
- **Settings precedence** runs **managed, command line, project local, shared project, user**, with the **highest level** that sets a key winning. **List keys merge** across files rather than overriding.
- **Instruction files load as context**, not as **enforced configuration**. A rule written there **guides Claude** and **guarantees nothing**, so a scenario **demanding a guarantee** is asking for another mechanism.
- A **plugin dependency tracks the newest release** by default, so an **upstream tag changes your behaviour** with no commit. A **version constraint in `plugin.json`** holds it in a **tested range**, resolved against tags named **`{plugin-name}--v{version}`**, and multiple **constraints intersect** to the **highest version satisfying** all.
- No **recorded version means three failures** at once: the **regression cannot be located**, the **working version cannot be held**, and nothing exists to **roll back** to. An upgrade cadence **records nothing** and raises the rate of **unattributed change**.
- **Environments differ deliberately**, so the fix is **consolidating into tracked configuration** that declares each difference, never **making them identical** or **unpinning until they agree**. Configuration in application code forces a **release for every settings change**.
- On an **untracked system**, **restore measurability first**: **pin the inputs**, give the **prompts as they stand** a version, and **rebuild the evaluation set** from recent traffic. **Rewriting the prompts** discards what the feature learned, and **deleting the stale set** leaves less than nothing.

## Endnotes for this chapter

1. Each Claude model ID identifies a pinned version of the model, and the underlying model remains constant for the lifetime of that ID; starting with the Claude 4.6 generation model IDs use a dateless format, and the dateless ID is the canonical model ID for that release, mapping to a single fixed model snapshot whose weights and configuration Anthropic does not update, with an updated version shipping under a new model ID. Anthropic identifies the belief that dateless IDs behave as evergreen pointers routing to the latest or best-performing version as a common misconception. Models before the 4.6 generation include a snapshot date in the ID and, on the Claude API, also have shorter aliases that point to the most recent dated snapshot for that minor version. Every model ID, dated or dateless, has its own distinct deprecation and retirement schedule. Model weights are fixed for a given ID, but the serving infrastructure around the model, including the request router, safety classifiers and sampling logic, can change, and infrastructure updates occasionally produce minor differences in observable behaviour even when the model ID and weights have not changed. Anthropic, *Model IDs and versioning* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions](https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions)

2. The retirement dates published by Anthropic apply to Anthropic-operated platforms; partner-operated platforms set their own retirement schedules, so a model's lifecycle status and dates can differ. Anthropic, *Model deprecations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/model-deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)

3. Success criteria should be specific and measurable, using quantitative metrics or well-defined qualitative scales; when choosing a grading method, code-based grading is the fastest and most reliable and extremely scalable but lacks nuance for complex judgements, LLM-based grading is fast, flexible and scalable and suitable for complex judgement provided its reliability is tested before scaling, and human grading is the most flexible and highest quality but slow and expensive. Anthropic, *Define success criteria and build evaluations* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/test-and-evaluate/develop-tests](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)

4. A project's `.claude/settings.json` reaches a teammate's clone and a cloud session only if it is committed to version control, and committing it gives everyone who clones the repository the same permissions, hooks, telemetry and plugins; `.claude/settings.local.json` holds one person's settings for that project and is added to the user's global git excludes the first time Claude Code writes it. When the same key appears in more than one place, Claude Code uses the value from the highest level that sets it, in the order managed settings, command line arguments, project local settings, shared project settings, user settings; list keys such as `permissions.allow` are combined across files instead of one file's value replacing another's. Anthropic, *Claude Code settings* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)

5. CLAUDE.md files give Claude persistent instructions and are loaded into the context window at the start of every session; Claude treats them as context rather than enforced configuration. A project CLAUDE.md can be stored in either `./CLAUDE.md` or `./.claude/CLAUDE.md`, holds instructions that apply to anyone working on the project, and is shared with the team through version control, so it should carry project-level standards rather than personal preferences. A `CLAUDE.local.md` at the project root loads alongside CLAUDE.md and is treated the same way, and should be added to `.gitignore` so it is not committed. Anthropic, *How Claude remembers your project* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)

6. MCP servers are registered at one of three scopes: `local`, stored in `~/.claude.json` under the entry for the current project and available only to you in that project; `project`, stored in `.mcp.json` in the project root and available to everyone who clones the project; and `user`, stored under the top-level `mcpServers` key in `~/.claude.json` and available only to you across all projects. `.mcp.json` is the file most worth writing by hand because it is checked into the repository. Anthropic, *Connect to MCP servers* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/mcp-quickstart](https://code.claude.com/docs/en/mcp-quickstart)

7. Managed settings are deployed by an organisation as a `managed-settings.json` file placed in the system directory for each operating system, as an MDM policy, or as server-managed settings from the claude.ai console. Anthropic, *Deploy managed settings* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/managed-settings](https://code.claude.com/docs/en/managed-settings)

8. By default a plugin dependency tracks the latest available version, so an upstream release can change the dependency under your plugin without warning; version constraints let you hold a dependency at a tested version range until you choose to move. Dependencies are listed in the `dependencies` array of a plugin's `.claude-plugin/plugin.json`, and Claude Code resolves version constraints against git tags on the repository hosting the dependency, each release tagged as `{plugin-name}--v{version}`. When several installed plugins constrain the same dependency, Claude Code intersects their ranges and resolves to the highest version satisfying all of them, and auto-update fetches the highest tag satisfying every installed plugin's range rather than the marketplace's latest version. Anthropic, *Constrain plugin dependency versions* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/plugin-dependencies](https://code.claude.com/docs/en/plugin-dependencies)

9. The `enabledPlugins` setting turns individual plugins on or off, keyed by `plugin-name@marketplace-name`, and can be set in any settings file. Anthropic, *Claude Code settings reference* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/settings-reference](https://code.claude.com/docs/en/settings-reference)
