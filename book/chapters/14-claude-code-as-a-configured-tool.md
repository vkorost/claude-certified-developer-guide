# Chapter 14: Permission Modes and Where the Human Gate Goes

**Summary**: *Claude Code gets configured before it gets trusted, and nearly every decision on this material is made once and then inherited by everybody afterwards. This chapter settles which permission mode fits the risk of the work, which file a setting belongs in so it reaches the people who need it, why a standard that must hold everywhere is declared once rather than copied into each repository, what each core component is for, and how an unattended run gets its output shape enforced instead of politely requested. Set the scope wrong and one afternoon's convenience becomes everybody's default.*

## The Mode Was Widened for One Refactor and Never Narrowed Again

A **rename across forty files** is unbearable one approval at a time. So somebody **widens the permission mode**, finishes the rename, and moves on. Correct for the rename. **Nobody sets it back**. It reaches the **project's settings file**, **gets committed**, and a session opened to change one value now starts with the authority of a session that was going to rewrite a package.

Call that a **standing grant**: **authority handed over once**, in a file, then **inherited by every later session** without anyone asking whether the task in front of you needs it. Permission modes, settings files, context files and allow rules are all **standing grants**, and nearly every scenario here is one of them meeting work it was **never sized** for.

## Plan Mode Separates Deciding Where to Change from Changing It

Claude Code works a task through **three blended phases**: gather context, act, verify.<sup>[1]</sup> The recommended loop **walls the first off** from the second. **Enter plan mode**. Claude reads, searches and explores without **touching your source**, you approve a plan, then the implementation runs.<sup>[2]</sup> That wall is real rather than advisory, because plan mode is itself a permission mode and **edits stay blocked** until the approval.<sup>[3]</sup>

One scenario shape follows. A behaviour is **spread across several entry points** and your earlier edits **kept landing** at the wrong one. More **reading is unbounded**, following test failures assumes coverage of every entry point, which is the thing in doubt, and one module per session guarantees nothing sees the behaviour whole. **Map the flows, compare the change points**, then edit.

Gathering rewards the same precision. A set defined by a **filename criterion** is selected exactly by a **path pattern**. **Content search** returns whatever mentions a word and **misses the files** that never do.

## Six Permission Modes, Read by Who Decides the Actions They Do Not Cover

A **mode sets your session's baseline**. Read the list by who is **left holding the decision** on everything the mode does not cover, rather than by how much it lets through.<sup>[3]</sup>

| Mode | Runs with no prompt | Who decides the rest |
|---|---|---|
| `default`, labelled Manual | Reads | A person, at a prompt |
| `acceptEdits` | Reads, file edits, common filesystem commands such as `mkdir` and `mv` | A person, at a prompt |
| `plan` | Reads, plus classifier-approved commands where auto mode is available | A person, and edits wait for an approved plan |
| `auto` | Whatever a classifier model clears | A second model, reviewing each action before it runs |
| `dontAsk` | Only pre-approved tools | Nobody: anything that would prompt is denied |
| `bypassPermissions` | Everything | Nobody, which is why it belongs in a container |

Two facts survive every row. **Deny rules apply in every mode**, including the one that skips checks, and **allow rules stop meaning anything** there. Writes to protected paths such as `.git` and `.claude` are never auto-approved outside it.<sup>[3]</sup>

So the anti-pattern is a **mode widened to stop the prompting**. Prompt fatigue is a real cost and not a **risk assessment**. Size the mode to the **blast radius**. A deny rule at project or organisation scope closes what a mode leaves open.

## Configuration Scope Follows Who Has to Receive the Setting

Claude Code **reads settings from four files** plus whatever an organisation deploys, and the file a key sits in decides who gets it. **Precedence runs** managed settings, command-line flags, `.claude/settings.local.json`, the committed `.claude/settings.json`, then `~/.claude/settings.json`, and list keys such as `permissions.allow` **merge across files rather than replacing** each other.<sup>[4]</sup>

Read that stack as a **question about audience**. A setting everyone on a project needs belongs in the **committed project file**: it **travels with the checkout**, it is reviewed like the code it governs, and your teammate's clone has it. The user file **covers every project** on your machine and reaches nobody else. The local file is a **personal exception** inside one project. Managed settings are deployed centrally. **Nothing below overrides** them.

Two named failures follow. **Environment variables carrying a team decision** leave the value on one machine, **unreviewed and unversioned**. **Documentation offered as configuration**, a wiki page or a preamble each developer pastes, depends on humans repeating themselves exactly, and nothing loads it.

## Context Files Layer Instead of Overriding, So a Standard Is Declared Once

`CLAUDE.md` is the **persistent instruction file**, **loaded at the start** of every session.<sup>[5]</sup> It sits at project scope in `./CLAUDE.md` or `./.claude/CLAUDE.md`, at user scope in `~/.claude/CLAUDE.md`, and as a managed policy file.<sup>[6]</sup> Run `/init` and Claude **drafts you a starter** from the repository itself.<sup>[7]</sup>

The hierarchy is what scenarios get built on. Every discovered file is **concatenated rather than overridden**, **ordered from broadest scope down** to your working directory.<sup>[5]</sup> **Nothing displaces anything**. So you **declare a standard** that must hold everywhere once, at the **wide scope**, and let each repository's file carry what is true only of it. Copy that block into every repository instead and there are as many **copies as repositories, drifting apart**, and a **central page** still needs a pointer in each one that somebody has to open.

Two limits travel with this. **Size degrades adherence**, so keep the file **near `200` lines** and move narrower guidance into `.claude/rules/`, where a **`paths` glob loads a rule** only when Claude reads a matching file. And the file is **context rather than enforced configuration**: blocking an action whatever the model decides is a hook, owned by chapters 13 and 21.<sup>[5]</sup>

## Five Components, Separated by When They Load and Who Invokes Them

| Component | Where it lives | When it enters context |
|---|---|---|
| Rules | `.claude/rules/*.md` | Every session, or only when a `paths` glob matches a file being read |
| Skills | `.claude/skills/<name>/SKILL.md`, or under `~/.claude` | Descriptions at startup, the body when Claude judges it relevant or somebody types `/name` |
| Commands | `.claude/commands/*.md` | On invocation, by typing `/name` |
| Subagents | `.claude/agents/*.md`, or under `~/.claude` | A separate context window, on delegation |
| Agent memory | `~/.claude/projects/<project>/memory/` | The `MEMORY.md` index every session, capped at `200` lines or 25KB; topic files on demand |

A **skill is a directory**, and that property decides a recurring scenario.<sup>[8]</sup> A procedure of **instructions plus support scripts** has somewhere to keep them, and its body **costs you no context** until it is used. The same material inside a context file **loads every session** whether the procedure comes up or not, with the **scripts still homeless**. Built as a library it becomes something application code calls, when the requirement was that a session could reach it. Command files still work. **Skills are their successor**.<sup>[6]</sup>

Subagents run in their **own context window** with their **own tool access**, so a long exploration **returns a summary** and leaves the main conversation clean.<sup>[9]</sup> Agent memory runs the other way. Claude writes it for itself, and being machine-local it **distributes nothing**.<sup>[5]</sup>

## An Unattended Run Needs Its Output Shape Enforced Rather Than Requested

Pass `-p` and Claude Code **runs one prompt and exits**, which used to be called **headless mode**.<sup>[6]</sup> `--output-format` selects plain text, structured JSON, or `stream-json`, the newline-delimited form that with `--verbose` and `--include-partial-messages` **emits tokens as generated**. The decisive flag is `--json-schema`: alongside JSON output it returns the result in a **`structured_output` field** against a schema **validated at startup**.<sup>[10]</sup>

That settles a family of automation scenarios. A pipeline turning a review into inline comments needs **one finding per object** with fields it can index. A **firmer instruction still only asks**. **Regular expressions** over markdown encode this week's formatting as a contract, and a cleanup pass runs a second unconstrained generation over the first. **Prose parsed in place of a contract** is the anti-pattern.

Two mechanics finish it. A `-p` run **starts in Manual** on every plan, so you **hand an unattended job its mode**, and `dontAsk` denies anything outside the **pre-approved set** rather than waiting for input nobody supplies.<sup>[3]</sup> Adding `--bare` **skips auto-discovery** of hooks, skills, commands, subagents, plugins, MCP servers, memory and context files, so a run behaves the same on a **machine nobody configured**.<sup>[10]</sup>

## The Gate Belongs Where the Worst Case Is Expensive, Not Where the Habit Is

Modes and rules **settle what runs unasked**. They say nothing about where you still have to look, and that decision has one input: what a **wrong call costs**, and how hard it is to **take back**.

**Sort actions by what undoes** them. An edit confined to the working directory is undone by a **checkpoint or a diff**, so a gate there buys you **oversight nobody needed**. An action undone only by another action, a destructive shell command or a write outside the working directory, **deserves a pause** by default and a **deny rule** to make that pause deterministic. An action nothing can take back **needs a human**, and the **agent's own review** is an input rather than a substitute.

Mode and gate are one **judgement seen twice**. A mode fixes a session's default posture. A gate **lifts one action** out of it.

## How Wrong Answers Are Built on This Material

Wrong options here describe **configurations real teams ship**. What makes one wrong is an **assumption underneath** it, and four cover almost everything.

- **Convenience treated as a requirement**. A **mode is picked** because it **removes interruptions** and the **risk of the work never enters** the comparison. The option becomes correct only where the scenario calls the work **low-stakes and reversible**.
- **State that does not travel** offered as **shared configuration**. Machine-local environment variables, a pasted preamble, a wiki page, a spreadsheet. **Each holds the content** and **none arrives with a checkout or loads** during a session.
- **Duplication chosen over a scope that already exists**. A **shared standard copied into every repository**, or linked from each one. **Layering already applies** the general and the local together, so a scenario naming two scopes of instruction **wants the layering**.
- A **shape requested where it had to be enforced**. **Stronger wording, cleverer parsing**, or a **second generation** to tidy the first. All three leave a consumer depending on **formatting nothing promised**.

**Match** on what the **scenario** is doing, never on the **industry** it is **dressed** in.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Configuration must be identical for everyone working on a repository | The committed project settings file, versioned alongside the code | Per-machine environment variables, or a document somebody is asked to follow |
| Standards apply everywhere, with extra rules in some repositories | Declare the general once at the wide scope and keep local files local | Copying the block into each repository, or linking out to a page |
| A downstream program consumes the run's output | Constrain the shape at the interface and validate on receipt | A firmer instruction, a regex over markdown, or a cleanup pass |
| Edits keep landing at the wrong place in unfamiliar code | Map the flows and compare change points before editing | More reading, test-driven iteration, or one narrow session per module |
| A team procedure carries instructions plus support scripts | A skill directory the tool discovers and loads on demand | Inlining it into the context file, or shipping it as a library |
| The action is hard to undo or reaches beyond the repository | A pause for a human, backed by a deny rule | A mode carried over from whatever the last task needed |
| A job runs unattended and must behave the same on every machine | Pre-approved tools, a mode that never waits, a fixed output shape | A default that assumes somebody is at a prompt |

One **habit transfers** past the table. Before you compare options, ask what a fresh clone on a **machine nobody has configured** would inherit.

## What to Remember

- A **standing grant** is **authority set once in a file** and **inherited by every later session** without anyone re-deciding it. **Permission modes**, **settings files**, **context files** and **allow rules** all qualify, and scenarios are built on one meeting work it was **never sized** for.
- **Six permission modes**: `default` (labelled **Manual**, reads only), `acceptEdits` (edits and **filesystem commands**), `plan` (**edits blocked** until a plan is approved), `auto` (a **classifier reviews each action**), `dontAsk` (**pre-approved tools** only, **never waits**), `bypassPermissions` (**containers only**). **Deny rules hold in every mode**; **allow rules** mean nothing in the last.
- **Settings precedence**, highest first: **managed settings**, **command-line flags**, the **project-local file**, the **committed project file**, then **user settings** in your home directory. **List keys** such as `permissions.allow` **merge across files** rather than replacing each other.
- **Scope follows audience**. A rule everyone on the project needs goes in the **committed project file**, which is **versioned, reviewed and present** in a **fresh clone**. **Per-machine variables and written documents** reach nobody.
- **Context files concatenate**, **broadest scope first**, so **nothing overrides anything**. **Declare a cross-repository standard** once at the **wide scope** and let each repository's file carry only its own; copying or linking **reintroduces drift**.
- `CLAUDE.md` is **context, not enforcement**, so an **absolute prohibition** belongs in a **hook**. Keep it **near `200` lines** and push narrow guidance into **`.claude/rules/` with a `paths` glob**.
- **Five components by load behaviour**: **rules** load every session or on a path match; **skills** load their body on demand and **bundle scripts**, being **directories**; **commands** run when typed; **subagents** get their **own context window and tools**; **agent memory** is **written by Claude**, stored per repository, **machine-local**.
- **`-p` runs one prompt and exits** and is the **former headless mode**; **`stream-json` streams events**; **`--json-schema` returns a validated `structured_output`**. An **unattended pipeline enforces the shape**, because **asking for a format** produces the intermittent behaviour already being reported. A **`-p` run starts in Manual** whatever the plan, and **`--bare` skips discovery** of hooks, skills, commands, subagents, plugins, MCP servers, memory and context files.
- **Place the human gate by reversibility**: **no gate** where a **checkpoint or a diff** undoes it, a **pause plus a deny rule** where only another action can, and a **person** where nothing can. The **agent's own review is an input** to that decision, never the decision.

## Endnotes for this chapter

1. Claude works a task through three blended phases, gathering context, taking action and verifying results, chaining tool calls and course-correcting until done. Anthropic, *How Claude Code works* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/how-claude-code-works](https://code.claude.com/docs/en/how-claude-code-works)

2. The recommended four-phase workflow of exploring in plan mode, asking for a detailed implementation plan, implementing after approval and committing, together with the guidance that planning is most valuable when the approach is uncertain, the change spans several files, or the code is unfamiliar. Anthropic, *Best practices for Claude Code* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

3. The available permission modes and what each runs without a prompt; the `default` mode being labelled Manual in the CLI, the extensions and the desktop app; plan mode blocking edits until a plan is approved; the auto-mode classifier reviewing actions before they run; `dontAsk` auto-denying every call that would otherwise prompt so the session never waits for input; `bypassPermissions` being restricted to isolated containers and VMs; deny rules blocking in every mode including `bypassPermissions` while allow rules have no effect there; the protected paths whose writes are never auto-approved outside that mode; and the built-in starting permission mode for `-p` runs being `default`. Anthropic, *Choose a permission mode* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/permission-modes](https://code.claude.com/docs/en/permission-modes)

4. The four settings files and the scope each reaches, including a committed project file reaching teammates' clones and a cloud session while user and local files stay on one machine; the precedence order from managed settings down through command-line arguments, project local, shared project and user settings; and list keys such as `permissions.allow` merging across files rather than overriding. Anthropic, *Claude Code settings* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)

5. CLAUDE.md files loading at the start of every session and being treated as context rather than enforced configuration, with a `PreToolUse` hook named as the mechanism for blocking an action regardless of what Claude decides; the project locations `./CLAUDE.md` and `./.claude/CLAUDE.md`; all discovered files being concatenated into context rather than overriding each other, ordered from the filesystem root down to the working directory; the guidance to target under `200` lines per file because longer files consume more context and reduce adherence; `.claude/rules/` holding modular instruction files that a `paths` glob scopes so they load only when Claude reads matching files; and auto memory being stored per repository under `~/.claude/projects/`, machine-local, with the first `200` lines or 25KB of `MEMORY.md` loaded at the start of every conversation and topic files read on demand. Anthropic, *How Claude remembers your project* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)

6. Definitions of the core components and features: CLAUDE.md placed at project scope, at user scope in `~/.claude/CLAUDE.md`, or as managed policy; rules as modular instruction files in `.claude/rules/`; commands invoked by typing `/name` and definable as files in `.claude/commands/`, with skills named the recommended successor; skills as a `SKILL.md` file loaded automatically when relevant or invoked directly; subagents running in their own context window with their own tool access and permissions; sessions tied to a directory with transcripts under `~/.claude/projects/`; non-interactive mode invoked with `-p` and formerly called headless mode; and bare mode starting without loading hooks, skills, custom commands, subagents, plugins, MCP servers, auto memory or CLAUDE.md. Anthropic, *Glossary* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/glossary](https://code.claude.com/docs/en/glossary)

7. `/init` initialising a project with a generated CLAUDE.md guide. Anthropic, *Commands* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/commands](https://code.claude.com/docs/en/commands)

8. Skills as a directory containing `SKILL.md` plus optional supporting files and bundled scripts, discovered from `.claude/skills/` at project scope and `~/.claude/skills/` at personal scope, with the body loading only when the skill is used so long reference material costs almost nothing until needed. Anthropic, *Extend Claude with skills* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)

9. Subagents preserving context by keeping exploration out of the main conversation, enforcing constraints by limiting which tools they may use, and being defined as files in `.claude/agents/` for a project or `~/.claude/agents/` for every project on a machine. Anthropic, *Create custom subagents* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)

10. Non-interactive runs with `-p`; `--output-format` accepting `text`, `json` and `stream-json`, the last being newline-delimited JSON used with `--verbose` and `--include-partial-messages` to receive tokens as they are generated; `--output-format json` combined with `--json-schema` returning the structured result in a `structured_output` field, with an invalid schema rejected at startup; and `--bare` skipping auto-discovery of hooks, skills, custom commands, subagents, plugins, MCP servers, auto memory and CLAUDE.md so a run behaves the same on every machine. Anthropic, *Run Claude Code programmatically* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless)
