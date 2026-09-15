# Chapter 21: A Rule in a Prompt Is Not Enforcement

**Summary**: *A control counts as a guardrail only if it still works on a run where the model has already decided wrongly. That one test sorts this whole skill, and it demotes most of what teams actually write down. Sort each control twice: does it need the model to cooperate, and does it run before the effect lands. Hooks, permission rules, identity scope and OS isolation survive both questions. A rule agreed in a meeting and typed into a prompt survives neither, and nobody finds that out on a quiet day.*

## The Rule Was Agreed by Everyone and Refused by Nothing

A team decides their agent writes into **one output directory** and nowhere else. The sentence goes into the **standing instruction** and into the project's **context file**. It gets reviewed, agreed, and shipped.

Months later a **write lands** somewhere else.

Nothing failed. No **component threw**. The rule had been addressed to the only part of the system whose job is to **weigh sentences against other sentences**, and on that run a **different sentence won**. Anthropic states it plainly: **permission rules are enforced by Claude Code** rather than by the model, and instructions in a prompt or a **context file** shape what Claude tries to do without changing what the **runtime allows**.<sup>[1]</sup>

Call the thing that was missing the **point of refusal**: the **moment in the sequence** where something other than the model can still **stop the action**. A control with no point of refusal is advice. Good advice, often. Advice.

## Two Questions Sort Every Control You Are Offered

Begin from the assumption that costs nothing and settles everything: the **model has already decided wrongly** on this run. A **planted instruction** and an **ordinary mistake** change nothing about what follows, and Anthropic's threat model names the **two routes** together for that reason.<sup>[4]</sup>

Ask two things of any control before you count it as a layer: whether it **needs the model to cooperate**, and whether it runs before the **effect lands**.

| Control | Needs cooperation | Runs before the effect | What it produces on a steered run |
|---|---|---|---|
| A rule in the standing instruction or context file | Yes | Not applicable | Intent, with no refusal underneath it |
| The model asked to confirm its own proposal | Yes | Yes | A second opinion from the first opinion |
| A `PreToolUse` hook | No | Yes | A refusal, and a reason returned to the model |
| A permission deny rule | No | Yes | A refusal the settings own |
| The scope of the identity the agent acts as | No | Yes | An operation that was never reachable |
| OS-level sandboxing around the process | No | Yes | A limit no rule had to name in advance |
| A `PostToolUse` hook, or an audit record | No | No | Evidence of what already happened |

The bottom five rows are **layers**. The top two are the **guarded process doing its own guarding**, which adds a step and no independence.

Read the **last column** rather than the first. **Two rows reach refusal** by different routes, and the bottom row earns its place for a reason unrelated to **prevention**.

## A Layer Earns the Name by Failing Without Taking Its Neighbours Down

**Layering** is not **counting**. Three controls resting on **one dependency** behave as one control **drawn three times**, and the drawing is what makes a review feel finished.

The arrangement that holds puts a control at **three positions that fail separately**: what **reaches the model**, what the **model produces**, and what a **privileged action is allowed** to do. Screening decides how often something lands. The hard limit on the action decides how bad it gets when screening misses. Neither substitutes for the other, and Anthropic's hardening guidance lists **isolation, network restriction, filesystem control and proxy validation** as options to combine rather than choose between.<sup>[4]</sup>

The **stack that shares a failure** is the anti-pattern, and it is built out of sensible parts. An instruction to refuse, a reminder to self-check, and a closing prompt to summarise are **three sentences read by one reader**. Chapter 20 sorts screening in detail; carry one property from it, which is that a probabilistic control **sets a rate** and **never a floor**.

The **independence test** is short enough to run while you read a scenario. Name one failure that **removes two** of them at once. If you can, the **count was wrong**.

## What the Identity Permits Bounds Everything a Steered Agent Can Do

Every other control in this chapter can be missing, misconfigured, or never written for the path your agent actually took. **Scope** is different, because a **capability that was never granted** cannot be reached by a **route nobody predicted**. Anthropic states the design rule: restrict the agent to only the **capabilities its specific task requires**.<sup>[4]</sup>

**Severity** is what this buys. An agent whose **identity can write anywhere** turns one steered decision into an **incident**. The same decision, made by an identity holding **one writable directory**, ends as a **denied call** and a line in a log.

**Four mechanics decide** whether the scope you wrote is the scope that runs.

| What you write | What it does to the capability | What can widen it |
|---|---|---|
| A rule in the prompt or the context file | Nothing at all | Any sentence weighed against it |
| A scoped deny rule such as one naming a command pattern | Leaves the tool present, blocks matching calls | Nothing below the scope it sits in |
| A bare tool-name deny rule | Removes the tool from context, so Claude never sees it | Nothing below the scope it sits in |
| A deny rule in managed settings | Same, organisation-wide | Nothing on the machine, command-line flags included |

Two consequences follow. **Deny beats allow at every scope**, so a deny rule cannot carry an **allowlist exception** and the **narrow allow** is the rule you write.<sup>[1]</sup> And a **grant needs trust** where a restriction does not: a project's allow rules and additional directories apply only after the **workspace trust dialog** is accepted, while its **deny and ask rules** take effect regardless, because they only ever restrict.<sup>[1]</sup>

The **broad deny with a carve-out** is the anti-pattern the first consequence kills.

There is a quieter rule underneath all of it. Whatever can **edit the scope holds the scope**, so changing permission configuration is itself a **privileged action**. Your administrator's copy has to win, which makes a **managed rule** no local setting or flag can override a **security property** rather than an inconvenience.<sup>[1]</sup>

Credentials sit one layer further out and belong to chapter 22. **Command parsing** checks a shell command against permission rules and asks for approval when it cannot parse cleanly or **matches no allow rule**, and Anthropic labels that machinery a **permission gate rather than a sandbox**, because it does not reason about what a **command's target path** would actually do.<sup>[4]</sup> **Rules cover** what you thought to name. **Isolation covers** the rest. Sandboxing is the complementary layer, restricting the **filesystem and network reach** of shell commands at the **operating-system level**, which holds even where an injection got past the model's own decision-making.<sup>[1]</sup>

## A Hook Chooses Where the Refusal Sits in Time

Chapter 13 built the hook and covered its precedence. Safety needs one thing from it: hook events **divide the loop** into places that can **still refuse** and places that can **only report**. A **`PostToolUse` hook fires** after a tool has **already executed successfully**, so it cannot undo the action.<sup>[2]</sup><sup>[3]</sup>

That fact answers a scenario shape worth recognising on sight. A policy has to be applied to **every privileged action**, and a **compliance record** has to exist for a human reviewer whether or not the action was permitted. Those are **two requirements at two vantage points**. **Denial** is available only while the **call is still pending**, and a **truthful record** only once it has resolved, so you need a **pair of hooks** rather than one clever one. A check at **session start** has no specific action in front of it. A sweep once the **session has closed** arrives **too late** to have refused anything.

Refusal is not the only thing position buys. Where a **deterministic transformation** has to happen before the model reasons over a value, Anthropic documents the **two seats** for it: intercept at **`PreToolUse` for outbound tool inputs** and at **`PostToolUse` for inbound tool results**, where a returned field **replaces the result** the model goes on to read.<sup>[3]</sup> Tool results arriving in **incompatible formats** get **canonicalised** there rather than by an instruction to remember which service uses which.

The **deterministic rule handed back to the model** is the anti-pattern, and it wears rigour well. Prompt-based hooks exist and put a model in the decision seat deliberately, which suits a **judgement call** and not an **absolute rule**.<sup>[2]</sup>

Two documented behaviours finish it. A **`PreToolUse` hook fires** before any **permission-mode check**, in every mode, and a deny from it blocks the tool even in **bypass mode**, which is what lets policy survive a user changing their own settings. The reverse does not hold: a **hook returning allow** does not defeat a **deny rule**, so **hooks tighten and never loosen**.<sup>[2]</sup>

Scope them where the **consequence lives**. Hooks on every action buy you **latency** on the ones that could not hurt anybody, and the repairs that look adjacent all **remove enforcement** instead of aiming it: **turning hooks off** during a rework, running the agent less often, or **relocating the rule** into the prompt. Keep them on **destructive operations and sensitive access**.

Two limits keep an aimed hook honest. A **matcher names the tool, not the effect**: Claude can **rewrite a file** by running a **shell command**, and a hook matching the **file-editing tools** does not fire when that happens, which is why compliance scanning uses a **`Stop` hook** that inspects the working tree once per turn.<sup>[2]</sup><sup>[3]</sup> And the **argument-level filter** that keeps a hook from spawning on irrelevant calls **fails open** when a command cannot be parsed, which is why the documentation directs a **hard allow or deny** to the permission system instead.<sup>[2]</sup> Scope with the **best-effort filter**. Enforce with the layer that does not **fail open**.

## How Wrong Answers Are Built on This Material

Wrong answers here describe **controls that exist**, that experienced humans build, and that appear in **real deployments**. Each **fails one test**, and it is always the same run: the model has already **decided wrongly**.

- **Instruction mistaken for enforcement**. A **system prompt**, a **context file**, a **refusal policy**, a **rule naming the directories** that are off limits. Each **shapes what gets attempted** and none **changes what the runtime permits**, so all are **absent on the one run** a guardrail exists for. They become correct where the question asks how to make an **attempt rarer**.
- **Recording offered where refusal was available**. **Logging the action**, **alerting** on it, **reviewing it later**, **reversing it afterwards**. Each needs the effect to have **happened first**. They are correct where **evidence is the requirement**, and they are the wrong half of one naming both **prevention and a record**.
- The **guarded process asked to guard itself**. **Self-confirmation**, a **second pass by the same model**, an instruction to **double-check before acting**. This adds a step and **no independence**, because a run that **decided wrongly the first time** is the run doing the checking.
- **Enforcement traded away** while appearing to tune it. **Suspending controls** during a rework, **narrowing the hours** the agent runs, **swapping a deterministic check for guidance**. Each answers a **real complaint about cost** by **removing the control** rather than pointing it at the **actions that carry consequence**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Write access to production, and which controls are genuinely independent | Scoped credentials, a deterministic check before execution, and validation of proposed actions before dispatch | Any option addressed to the model that is being guarded |
| A rule must hold and a record must exist whether or not the action proceeded | Two events: refuse before the call, record after it | One check doing both, or a check at session start or session end |
| The entire safety approach is one screening step or one instruction | Independent layers on input, on output, and on the privileged action | Objections about transport security, billing, or instructions being forgotten |
| Controls fire on every action and the run has become slow | Keep them on destructive operations and sensitive access, drop the rest | Suspending them, restricting when the agent runs, or moving the rule into a prompt |
| Values must be normalised before the model reasons over them | Convert in the gap after a tool returns and before its result is read | Asking the model to remember each format, or repairing the wording afterwards |
| A restriction must survive the person it constrains | Managed settings, which no local file or flag overrides | A documented convention, or a rule in files the constrained party can edit |

One habit transfers past the table. For any control offered, **describe the run** where the **model has already been steered** and say what that control still does on it. An answer that stops at making the **attempt rarer** has told you where the control belongs, which is **above the floor** rather than at it.

## What to Remember

- A **guardrail holds** on a run where the **model decided wrongly**. **Sort every control** by two questions: does it **need the model's cooperation**, and does it **run before the effect lands**. Controls needing cooperation **set a rate**; controls running after the effect **produce a record**.
- The **point of refusal** is the **moment** something other than the model can **stop the action**. **Permission rules are enforced by the runtime**, so a **prompt or context file** changes what is **attempted** and never what is **allowed**.
- **Layering means three controls that fail separately**: screening what **reaches the model**, screening what it **produces**, and a **hard limit on privileged actions**. Test it by naming one failure that **removes two** of them; if you can, the **count was wrong**.
- **Least privilege bounds severity**, because a **capability never granted** cannot be reached by a **route nobody predicted**. Write the narrow allow: **deny beats allow at every scope** and a deny rule carries **no exceptions**.
- A **bare tool-name deny removes the tool** from context so Claude never sees it; a **scoped deny** leaves the tool and **blocks matching calls**. **Managed settings override every local file and flag**, which is what stops the constrained party **widening their own scope**.
- **Permission rules cover what somebody named**; **sandboxing covers the rest**. Command matching is a **permission gate rather than a sandbox** and does not reason about what a command would do, while **OS-level isolation limits filesystem and network reach** whatever the model decided.
- **`PreToolUse` refuses, `PostToolUse` reports**. A hook after the call **fires** only once the **tool has executed** and **cannot undo** it, so a requirement naming both a **policy check** and a **compliance record** needs **two hooks at two events**.
- **Hooks tighten and never loosen**. A `PreToolUse` deny holds in every **permission mode**, **bypass included**, while a **hook returning allow** does not defeat a **deny rule**.
- **Scope enforcement** to where the **consequence lives**, keeping hooks on **destructive operations and sensitive access**. A **matcher names the tool, not the effect**: a **shell command** rewriting a file misses a hook matched to the **file-editing tools**.

## Endnotes for this chapter

1. Permission rules are enforced by Claude Code rather than by the model, so instructions in a prompt or CLAUDE.md shape what Claude tries to do but do not change what Claude Code allows, and access is granted or revoked through permission rules, a permission mode or a PreToolUse hook; a deny rule blocks every matching call including calls that also match a narrower allow rule, so a deny rule cannot carry allowlist exceptions, and the same precedence applies between ask and allow; a bare tool name in a deny rule removes the tool from Claude's context entirely so Claude never sees it, while a scoped rule leaves the tool available and blocks matching calls; a hook that exits with code 2 stops the tool call before permission rules are evaluated so the block applies even when an allow rule would let the call proceed, while hook decisions do not bypass deny and ask rules; permission rules follow settings precedence with managed settings highest, so no other level including command line arguments can override a managed permission rule and a tool denied at any level cannot be allowed at another; permissions and sandboxing are complementary layers, with permissions controlling which tools, files and domains Claude Code can reach and sandboxing providing OS-level restriction of the Bash tool's filesystem and network access that prevents commands reaching resources outside defined boundaries even if a prompt injection bypasses Claude's decision-making; and a project's `permissions.allow` rules and `permissions.additionalDirectories` entries grant capability so they apply only after the workspace trust dialog is accepted, while deny and ask rules are unaffected since they only restrict. Anthropic, *Configure permissions* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)

2. Hooks are user-defined shell commands run at specific points in the lifecycle, giving deterministic control so that certain actions always happen rather than relying on the LLM to choose to run them; `PostToolUse` hooks cannot undo actions since the tool has already executed; `PreToolUse` hooks fire before any permission-mode check in every permission mode including `dontAsk`, and a hook returning `permissionDecision: "deny"` blocks the tool even in `bypassPermissions` mode or with `--dangerously-skip-permissions`, which allows enforcement of policy that users cannot bypass by changing their permission mode, while the reverse is not true because a hook returning `"allow"` does not bypass deny rules from settings, so hooks can tighten restrictions but not loosen them past what permission rules allow; where a hook must see every file change, for compliance scanning or audit logging, a `Stop` hook that scans the working tree once per turn is recommended because Claude can also create or modify files by running shell commands; the `if` field filters hooks by tool name and arguments together but fails open, running the hook regardless of pattern, when the Bash command cannot be parsed, and because the filter is best-effort the documentation directs the use of the permission system rather than a hook to enforce a hard allow or deny; and `type: "prompt"` hooks exist for decisions that require judgment rather than deterministic rules, sending the prompt and the hook's input data to a Claude model to make the decision. Anthropic, *Automate actions with hooks* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)

3. `PostToolUse` hooks fire after a tool has already executed successfully, with input carrying both the arguments sent to the tool and the result it returned, and the event cannot block because the tool already ran; `updatedToolOutput` replaces the tool's result, and for redaction or transformation use cases the documented interception points are `PreToolUse` for outbound tool inputs and `PostToolUse` for inbound tool results; and Claude Code does not run a `PostToolUse` hook matching `Edit|Write` when a `Bash` command or a process outside Claude Code rewrites the same file. Anthropic, *Hooks reference* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks)

4. Agents can take unintended actions due to prompt injection, meaning instructions embedded in content they process, or due to model error; command parsing for permissions parses bash commands into an AST and matches the result against permission rules, with commands that cannot be parsed cleanly or that do not match an allow rule requiring explicit approval and a small set of constructs always requiring approval regardless of allow rules, and this is described as a permission gate rather than a sandbox which, apart from built-in safety checks, does not infer whether a command is dangerous from its target path or effects; a security boundary separates components with different trust levels, so sensitive resources can be placed outside the boundary containing the agent; least privilege restricts the agent to only the capabilities required for its specific task; and defense in depth for high-security environments layers multiple controls, with the listed options being container isolation, network restrictions, filesystem controls, and request validation at a proxy. Anthropic, *Securely deploying AI agents* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/agent-sdk/secure-deployment](https://code.claude.com/docs/en/agent-sdk/secure-deployment)
