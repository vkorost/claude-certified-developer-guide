# Chapter 11: Four Mechanisms, Decided by What Must Execute

**Summary**: *A capability arrives and four mechanisms are offered for it, and at least three of them would work. That is the difficulty. Any rule beginning "always prefer" has stopped reading the requirement, because what fits is settled by the thing being packaged and never by a ranking of the packages. Two questions decide nearly every case: whether the work has to reach a live system, and how many consumers there are. This chapter teaches those two, what each mechanism charges, and the loading rules that decide whether a package runs on somebody else's machine.*

## Four Proposals Arrive for One Capability, and None of Them Is Absurd

The **month-end reconciliation write-up** has to come out the way the **finance team produces** it. **Fixed section order**. **Wording nobody improvises**. A checklist run before it goes anywhere.

**Four proposals reach** the meeting. Reach for a **built-in tool**, since Anthropic already ships one that runs code. Define a **custom tool** in the application, since that is where every other capability lives. Package the whole procedure as a **Skill**. Stand up a **protocol server**, since **two other teams** will want this by spring.

Every one of those is **real engineering**, and somebody **shipped** each of them this quarter. Elimination by **obvious nonsense** returns nothing here. All four **survive a first read**, which is exactly why the material is worth a chapter.

What separates them is **fit**, and fit is decided by two **properties of the capability**. Neither property is a property of the mechanisms at all.

## The Execution Question Comes First, Because It Removes Two Options at Once

Ask it before you compare anything. The **execution question**: does this capability have to **reach a system** that holds **live state** and **authenticates its callers**, or is it **knowledge** about how work gets done?

**Procedure** is the second kind. A section order, a template, a house convention, a review checklist, the three things humans always forget. **None of it calls anything**. It is **expertise written down**, and the mechanism built to carry expertise written down is a **Skill**: a directory holding a **`SKILL.md` file** with **YAML frontmatter**, a body of instructions, and whatever **templates and scripts** the procedure needs beside it.<sup>[1]</sup>

**Live access** is the first kind. A balance that changed this morning, a record only certain callers may read, a write that has to land. **Prose describing an endpoint** cannot **make a request** to it, and prose describing an endpoint **drifts from the endpoint** with nothing in the system able to notice. The **procedure served over a protocol** is the **mirrored error**, and it is the more seductive one: standing a server in front of static instructions buys a **network hop** and no capability the instructions did not already have.

One nuance keeps the rule honest rather than breaking it. A Skill may **bundle executable scripts**, and the runtime does run them.<sup>[1]</sup> What that does not buy is **reach**. Through the API a Skill runs inside the **code execution container**, sandboxed, with **no network access** and **no runtime package installation**<sup>[1]</sup>, so a bundled script is **deterministic local work** rather than a route to a production system.

## A Built-in Tool Is Capability Anthropic Already Operates

Once something has to execute, **three mechanisms are left**, and the **cheapest definition** is always the one you never write. Check what **already exists** first.

Built-in does not mean one thing, and the split matters more than the label. **Tools differ by where the code runs**. **Server tools**, among them web search, web fetch, code execution and tool search, **run on Anthropic's infrastructure**: the call and its result both arrive in the response, and **no handler** in the application executes anything.<sup>[2]</sup> Other built-ins, **bash and the text editor** among them, come with a **schema Anthropic publishes** and trains Claude against, while the **application still runs** the operation and returns the result.<sup>[2]</sup>

So a built-in **buys a definition** and sometimes an implementation. What it costs you is **ownership**. Chapter 9 put validation, the approval gate and the audit record at dispatch, and for a **server-executed tool** there is **no dispatch** on a machine you control. That is the tradeoff worth carrying into a scenario: a built-in is right where the capability is **generic** and the **governance** is not the point, and wrong the moment a **human has to answer** for what ran.

**Wrapping a built-in to make it shareable** is an **anti-pattern** with a specific failure. The wrapper **lives inside one application**, so other teams reach neither the wrapper nor the reason it was written.

## The Consumer Count Decides Between a Private Function and a Protocol

**Two mechanisms remain**, they do the same thing, and the **question separating** them is **not technical**. **Count the applications**.

**One application**, a **narrow operation**, an **input contract** you already know: that is a **custom tool**, defined where the application lives. In the **Agent SDK** it is **four parts**, a name, a description, an input schema and the handler that runs, and the server carrying it **runs inside your application** rather than as its own process.<sup>[3]</sup> There is **no deployment**, **no transport** decision, **no second thing** to version.

**Several applications**, released on **schedules none of them share**: that is a **protocol server**, and chapter 10 covered what one exposes and where it runs. What belongs here is the arithmetic. A server is **machinery for reuse**, and the machinery **gets paid** for whether or not anybody reuses it: a process to deploy, a transport to pick, an interface to hold still while its callers move. Buy that for **one consumer** and the **bill arrives** with nothing on the other side of it.

Run the arithmetic in the other direction and **two comfortable answers** stop working. The **copied implementation** puts the operation inside each **team's application**, which **reads as independence** and **produces divergence**: every team now **owns a variant**, and the next change to the underlying system happens in as many **repositories as there are teams**. A **shared library** looks better and **moves the same cost** by a step, because every consumer **still integrates** it and every consumer still has to upgrade before the fix reaches anybody.

| What the capability needs | Who needs it | What fits | What you are signing up for |
|---|---|---|---|
| Nothing executed; procedure, templates, conventions | Anyone whose task matches its description | A Skill | Writing a description precise enough to load, and keeping the body portable |
| Generic execution Anthropic already runs | Whoever is on the request | A built-in tool | Governance you do not own, on work you do not see run |
| A private operation with a known input contract | One application | A custom tool | Nothing shared; a second consumer means doing this again |
| Live, authenticated access with its own release cycle | Several applications, several teams | A protocol server | A deployment, a transport, and an interface that outlives its first caller |

Read the rightmost column as the price list. Most wrong answers in this material are correct choices from the wrong row, and the cue that tells you which row a scenario sits in is always stated in the scenario.

## A Skill Costs About a Hundred Tokens Until Somebody Needs It

The reason a **long procedure** belongs in a Skill rather than in the **instructions every session** already carries is **arithmetic** too, and it is the arithmetic of **progressive disclosure**.

Loading happens in stages. A Skill's **name and description** sit in the **system prompt** from startup, at roughly a **hundred tokens** each. The body of `SKILL.md` reaches context only when a **request matches that description**, and stays under **five thousand tokens**. **Bundled reference files** cost **nothing** until something reads them, and bundled scripts contribute **only their output** rather than their source.<sup>[1]</sup> Because the agent never has to read a whole Skill to use part of it, what a Skill can carry is **effectively unbounded**.<sup>[4]</sup>

Set that against the alternative and the shape of the **wrong answer** appears. The **always-on procedure**, written into a **project instruction file** or a system prompt, is **paid for on every request** including the overwhelming majority that have nothing to do with it. Claude Code documents the same contrast directly: a skill body **loads only when used**, so long reference material **costs almost nothing** until somebody needs it.<sup>[5]</sup>

Which puts unusual weight on one field. The **description is not documentation**, it is the **matching criterion**, and it has to say what the **Skill** does *and* when to **reach** for it.<sup>[1]</sup> A **vague description** does not degrade gracefully. It simply **never loads**, in every runtime equally, and the failure looks like a Skill that was **never installed**.

## One File, Several Runtimes, and Discovery Changes in Each

The same **`SKILL.md`** runs in the **interactive terminal**, through the **API**, and under the **SDK**. What changes across the three is how the **runtime finds** it and what it is **allowed to touch**, so "runs everywhere" is something you design for.

In Claude Code, **Skills are files**: personal ones in **`~/.claude/skills/`**, project ones in **`.claude/skills/`**, with enterprise definitions **overriding personal** and **personal overriding project**.<sup>[5]</sup> The **SDK discovers** the same **filesystem locations** through its setting sources and takes a **`skills` option** naming which of them a session may invoke.<sup>[6]</sup> Through the API a custom Skill is **uploaded to the workspace** instead, addressed by identifier alongside the code execution tool, and production requests **pin a version** so an update to the Skill cannot quietly change deployed behaviour.<sup>[7]</sup>

That last difference generalises. The **file ports**; the **distribution does not**. **Custom Skills do not sync** between surfaces on their own, and a Skill wanted on three surfaces is managed on three surfaces.<sup>[1]</sup>

Then there is the failure everybody meets once. A **packaged workflow installs cleanly** on every machine in the team and **runs on exactly one**, because **installation copies files** while **execution resolves** what those files point at. The **author's-machine assumption** is the named form, it is invisible to the author by construction, and it has three **usual shapes**.

| What the author's machine supplied | Why the install still succeeded | What the package should have carried |
|---|---|---|
| An absolute path into a home directory | A path is plain text until something resolves it | A path relative to the Skill, written with forward slashes |
| A tool or command available in the local shell | Nothing in the package declares a shell dependency | Steps confined to what the runtime guarantees, or the dependency stated |
| An environment variable set months ago | The variable is read at the step that needs it, not at install | Required packages and variables named in the body |

Two more constraints belong on the same list. A Skill referring to protocol tools **names them fully**, **server prefix** included, or the **lookup misses** when several servers are connected.<sup>[8]</sup> And a **subagent starts clean** rather than **inheriting** what its parent had loaded, so a Skill the parent depended on is **listed for the subagent** explicitly.

Packaging sits on top of all of this rather than beside it. A **plugin bundles Skills**, agents, hooks and protocol servers into one versioned, **installable unit**<sup>[9]</sup>, which is how a chosen set travels to a team. It is **distribution machinery**, **not a fifth answer** to the execution question, and a **guardrail** the author relied on locally travels only if the **bundle actually contains** it.

## How Wrong Answers Are Built on This Material

Wrong answers here **name mechanisms that work**. Every one of them is **somebody's shipped architecture**, which is why they read as reasonable and why **four assumptions** are worth memorising instead.

- A **universal preference stated as a rule**. "**Prefer a Skill** for anything procedural." "**Standardise on servers**." A **statement that holds** names something true of the **work being packaged**; a **statement that fails crowns one mechanism** regardless of what the requirement said. **Cost claims** are the common disguise, since a claim that **one mechanism is always cheaper** has **priced nothing** in the scenario.
- **Instructions offered where execution was required**. Documentation of an API, a procedure describing the query to run, a **Skill standing in for a call**. Ask what the option would **actually invoke**. A **package that reaches nothing** cannot **substitute for a capability** that must **reach something**, and it **drifts from the system** it describes with **nothing able to detect** the drift.
- **Reuse machinery bought for one consumer**, or **per-consumer copies bought for many**. The same **misread in both directions**. A **private function** does not need a deployment; **four teams** do not need **four implementations**, and a **shared library keeps the coupling** by making every consumer **integrate and upgrade**.
- A **recurring cost accepted for an occasional need**. A **long procedure** loaded on **every request**, or a **nightly copy** of data standing in for **live access**. Both **trade** something the requirement asked for, **freshness or context**, against **convenience**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| A documented internal procedure with templates, conventions and a checklist, and nothing to call | A Skill, loaded when a request matches its description | A schema, which names fields and validates them but carries no conventions |
| Several applications on different teams need one live system, released independently | One protocol server, connected by each application | The same integration repeated per application, or documentation of the endpoints |
| One application, one narrow operation, an input contract already known | A custom tool where that application lives | Server overhead bought for reuse that never happens |
| The capability is generic and Anthropic already runs it | The built-in, taken as it comes | A private reimplementation, or a wrapper nobody else can reach |
| A capability must be usable independently by teams that do not share a codebase | One maintained artifact many consumers invoke | Copying it per team, or a library every consumer must upgrade |
| The procedure must be absent from unrelated requests | On-demand loading, paid for at the description | Always-on instructions, paid for on every request |
| A packaged workflow installs everywhere and runs only for its author | Environment assumptions removed from the package | Treating a clean install as evidence the package works |

One habit transfers past the table. Before comparing options, answer the **two questions** from the **scenario's own words**: what has to **execute**, and how many **consumers** were named. An option that would be correct under a **different answer** to either question is the one **placed there to catch** you.

## What to Remember

- What **fits is settled by the capability**, never by a **ranking of mechanisms**. Any claim that **one mechanism always wins**, on cost or on anything else, has **ignored the requirement** it was asked about.
- The **execution question sorts first**: does the capability **reach a live, authenticated system**, or is it **knowledge** about how work is done? Procedure with **templates, conventions and checklists** is what a **Skill packages**.
- **Consumer count sorts second**. **One application** with a **known input contract** gets a **custom tool defined** in that application; **several applications** on **independent release cycles** get a **protocol server**, which is **reuse machinery you pay** for whether or not the reuse happens.
- A built-in is **capability Anthropic already ships**, split by where code runs: **server tools execute** on **Anthropic's infrastructure** with **no handler** in your application, while **bash and the text editor** carry **Anthropic's schema** and still **execute locally**. What you give up is **governance at dispatch**.
- **Skills load in stages**: **name and description** always, at about a **hundred tokens** each; the body only on a **description match**, under **five thousand tokens**; **bundled files and scripts** cost **nothing** until **read or run**. That is why a **long procedure** belongs in a Skill rather than in **always-on instructions**.
- The **description is the matching criterion**, not documentation. It states what the **Skill does and when to use** it, and a **vague one never loads** in any runtime.
- One **`SKILL.md` runs across surfaces**, but **discovery differs**: **filesystem directories** in Claude Code and the SDK, an **uploaded workspace Skill** on the API with a **pinned version** in production. **Custom Skills do not sync** between surfaces on their own.
- A **clean install proves nothing** about execution. **Absolute paths**, **local commands** and **environment variables** baked into a package **install everywhere** and **run only for the author**, and **subagents inherit no Skills** from a parent.
- A **plugin bundles Skills**, agents, hooks and servers into one **versioned installable unit**, which **distributes a choice already made** rather than making one. A **guardrail left out of that bundle** reaches nobody.

## Endnotes for this chapter

1. Skills are reusable, filesystem-based resources that load on demand rather than requiring guidance to be repeated across conversations; each Skill is a directory containing a `SKILL.md` file whose YAML frontmatter requires `name` and `description`, with the description stating both what the Skill does and when Claude should use it, because it is what Claude matches a request against; content loads in levels, with metadata always loaded at startup at roughly 100 tokens per Skill, the `SKILL.md` body loaded when the Skill is triggered at under 5k tokens, and bundled resources consuming nothing until accessed, where reference files load when read and scripts run through bash with only their output entering context; Skills may bundle instructions, executable scripts and reference resources; custom Skills in Claude Code are filesystem-based, placed in `~/.claude/skills/` or `.claude/skills/`; Skills on the API run in a sandboxed container with no network access and no runtime package installation; and custom Skills do not sync across surfaces, so they are managed and uploaded separately for each surface. Anthropic, *Agent Skills* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

2. Tools differ primarily by where the code executes: client tools, including user-defined tools and tools with Anthropic-defined schemas such as `bash` and `text_editor`, run in your application, which executes the operation and returns a `tool_result`; server tools such as `web_search`, `web_fetch`, `code_execution` and `tool_search` run on Anthropic's infrastructure, with results visible directly and no handler code in the application. For tools with Anthropic-defined schemas, Anthropic publishes the schema and trains Claude on it while the application still executes each call. Anthropic, *Tool use with Claude* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)

3. A custom tool is defined by four parts, a unique name, a description Claude reads to decide when to call it, an input schema, and an async handler that receives the validated arguments and returns result content; tools are registered on an SDK MCP server that runs in-process inside the application rather than as a separate process. Anthropic, *Give Claude custom tools* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/custom-tools](https://code.claude.com/docs/en/agent-sdk/custom-tools)

4. Skills are presented as composable resources that complement Model Context Protocol servers by teaching agents workflows involving external tools, and because an agent with a filesystem and code execution does not need to read the entirety of a Skill into its context window for a given task, the amount of context that can be bundled into a Skill is effectively unbounded. Anthropic, *Equipping agents for the real world with Agent Skills* (Anthropic Engineering, accessed 2026-09-09), [https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

5. Unlike CLAUDE.md content, a skill's body loads only when it is used, so long reference material costs almost nothing until it is needed; where a skill is stored determines who can use it, and when skills share a name Claude Code resolves the conflict by source, with enterprise overriding personal and personal overriding project. Anthropic, *Extend Claude with skills* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)

6. In the Agent SDK, skills are filesystem artifacts created as `SKILL.md` files in their own directories, loaded from the filesystem locations governed by the setting sources, discovered at startup as metadata with full content loaded when Claude invokes the skill, invoked either by the model autonomously or by the user directly, and scoped through a `skills` option accepting a list of names, all, or none. Anthropic, *Extend agents with skills* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/skills](https://code.claude.com/docs/en/agent-sdk/skills)

7. Skills integrate with the Messages API through the code execution tool, with the Skill referenced by `skill_id` and `version` in the `container` parameter; custom Skills are uploaded through the Skills API and shared workspace-wide; a new version is a complete snapshot rather than a delta; and the documented production guidance is to pin a specific version so that Skill updates never change deployed behaviour. Anthropic, *Using Agent Skills with the API* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/skills-guide](https://platform.claude.com/docs/en/build-with-claude/skills-guide)

8. Skill authoring guidance: use forward slashes in file paths rather than backslashes; do not assume packages are available, and state the required installation explicitly; and always use fully qualified tool names in the form `ServerName:tool_name` when a Skill uses MCP tools, because without the server prefix Claude may fail to locate the tool, especially when multiple MCP servers are available. Anthropic, *Skill authoring best practices* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

9. Plugins extend Claude Code with skills, agents, hooks and MCP servers bundled into a single installable unit, and plugin skills are always namespaced with the plugin's name to prevent conflicts when several plugins ship a skill of the same name. Anthropic, *Create plugins* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins)
