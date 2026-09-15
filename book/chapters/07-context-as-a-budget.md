# Chapter 7: The Window Is a Budget, and Everything Competes

**Summary**: *By the time an agent reaches its twentieth tool call, most of the window is holding work that is already finished. It was useful once. It is still being paid for on every request, and it is still competing for attention with the constraint you set three turns ago. This chapter covers where the budget goes before the current question is even added, why overspending shows up as odd behaviour rather than as an error, the two levers that reclaim room, and what compaction is allowed to throw away.*

## Everything in a Request Draws on One Budget, Including What Is Already Finished

Take an agent **twenty tool calls** into a task and count what the **window carries** before the **current instruction** is added.

The **system prompt**. Every message in the request, and that includes **tool results, images and documents**. Your **tool definitions**. Then the **output** for the turn, **extended thinking** included.<sup>[1]</sup> **Cached prefixes** are in there too: caching changes what those tokens cost you, not whether they **take up room**.<sup>[1]</sup>

Most of that has already done its job. Call it **spent context**: content fetched for a **step that finished**, **resent on every later request**, drawing on the same budget as the question you actually want answered. Anthropic's account of context engineering makes the argument about **attention rather than storage**. Context is finite, every token added draws down a **limited attention budget**, and the work is curating the smallest set of **high-signal tokens** rather than acquiring more room.<sup>[4]</sup>

Development hides all of this, reliably and for one reason. **Test fixtures are small**. **Production tool outputs** commonly run several times longer than the fixtures a workload was built against, so a session that held together across **twenty turns** in testing **fills at turn eight** once real documents flow through it. The **window did not change**. Each turn now spends more of it, which makes the budget something to measure rather than assume: the **token counting endpoint** takes the same body as a real request and **returns a count** without running inference.<sup>[6]</sup>

## Bloat Is a Size Problem and Drift Is an Attention Problem

Overspending has **two presentations**, and only one of them **raises anything**.

**Bloat** is volume: **requests grow**, the **bill grows** with them, **latency follows**, and if input alone finally **exceeds the window** the API rejects the request with a **400**.<sup>[1]</sup> All of it shows up in a dashboard.

**Drift** is behaviour, and it **reports nothing**. Claude answers from **specifics settled ten turns ago**, quietly **stops honouring a rule** set at the start, and **picks the wrong tool** for a step it would have got right earlier. Humans reading the transcript see plausible work.

| The symptom | What it means | The misreading it invites |
|---|---|---|
| Every turn costs more than the last on one unchanged task | Spent material is being resent in full | Blaming the model tier for latency |
| Tool choices correct for four turns, wrong on the fifth | Accumulated results now outweigh the instructions | Rewriting the tool descriptions |
| Answers lean on stale detail nobody has mentioned in ten turns | Old specifics still carry full weight | Reading it as a retrieval or memory defect |

The third row is where most debugging time goes. Nothing about a **tool schema changed** between turn four and turn five. What changed is the **volume of finished work** sitting between the **instructions** and the **decision**.

Which is why **more room is not a treatment for drift**. A **window twice the size** preserves every distraction at full detail and **leaves space for more**. Chapter 1 drew the line between capacity and selection on one request; this is that line one level up. When a corpus of tens of thousands of documents is proposed for every prompt, the objection that survives is not a document limit: grant the room and you still **pay input on the whole corpus** per query and spread attention across all of it.

## Two Levers Reclaim Room: Delete What Is Spent, or Compress It

**Context editing deletes** and **compaction rewrites**, and scenarios on this material turn on the difference between them.

**Clearing tool results** removes the oldest tool interactions once a prompt passes a trigger, **100,000 input tokens** by default, keeping the **three most recent** by default.<sup>[2]</sup> Each cleared result leaves **placeholder text** behind, so Claude reads that something was removed rather than that it never happened.<sup>[2]</sup> The parameters are where the judgement lives: **`exclude_tools`** names tools whose results are **never cleared**, **`clear_at_least`** refuses to clear at all unless the pass frees a **worthwhile minimum**, and **`clear_tool_inputs`** extends the clearing to the call parameters as well as the results.<sup>[2]</sup> A companion strategy **clears thinking blocks**.<sup>[2]</sup> All of it runs **server side**, so your application keeps its own full history and never resyncs.<sup>[2]</sup>

**Compaction summarises** instead. The strategy **triggers on input tokens**, **150,000** by default and **never below 50,000**, and Claude writes a summary into a **`compaction` block** that you pass back; on later requests **every content block before it is ignored**.<sup>[3]</sup> It costs a **separate sampling step**, billed and counted against rate limits, and the summary is written by the model already in your request, with no cheaper option.<sup>[3]</sup>

| | Clearing tool results | Compaction |
|---|---|---|
| What happens to the material | Removed, with a placeholder marking the gap | Rewritten as a summary; earlier blocks then ignored |
| What survives | Whatever your parameters protect | Whatever the summariser thought to write down |
| What it costs | Cache invalidation where the clearing happens | A billed extra sampling step |
| Reach for it when | Old tool output is the bulk of the growth | The thread is long and continuity still matters |

**Tool definitions** are a third source of pressure and answer to neither lever, because they sit at the **front of every request** rather than accumulating behind it. **Deferring them until Claude asks**, or **collapsing a call chain** into **one sandboxed script**, are the documented answers.<sup>[5]</sup> Chapter 9 owns those; chapter 8 owns caching.

## A Constraint That Lives Only in the History Is a Constraint Compaction May Delete

**Compaction keeps the gist**, which is both the whole point of it and the whole hazard.

Run a long task through several passes and a **rule stated once**, in turn three, has been summarised, and then **summarised again from that summary**. Each pass is free to decide it was not worth carrying. **Nothing raises an error** when it goes, and the first evidence is an agent calmly breaking a constraint it honoured an hour earlier.

The **remembered constraint** is the anti-pattern, and it is seductive because it looks like the right instinct. A team notices the rule getting lost, so they **state it again in the conversation**, sometimes with an explicit instruction to remember it. That restatement **sits inside the region being rewritten**, subject to the process that dropped the rule.

So the decision rule is about **location rather than emphasis**. A constraint that must **hold for the whole run** belongs **outside the compactable region**, in the **system prompt** or in a record your application reinjects each turn.

**Threshold tuning** looks like the other fix and is not. Raise the trigger and compaction runs less often on more material; lower it and it runs more often on less. Both change the **frequency of the loss**. Neither changes what a **summary may leave out**.

Claude Code makes the boundary unusually concrete. **Path-scoped rules and nested memory files** load into message history when their trigger file is read, so **compaction summarises them away** with everything else, and the documented repair for a rule that must persist is to move it to the **project-root memory file**.<sup>[8]</sup> Skill bodies are reinjected after a compaction, but a large one is **truncated to a cap** and the truncation **keeps the start of the file**, so the **instructions that matter** belong at the top.<sup>[8]</sup>

One more control matters: custom summarisation instructions **replace the default prompt completely** rather than supplementing it.<sup>[3]</sup> A summariser asked only to summarise the conversation returns a readable account of what happened and drops the **file paths, the decisions and the errors** already resolved.

## Isolation Moves Material Out of the Window Rather Than Shrinking It

**Pruning and compaction** both work on material that has already arrived. The third technique **keeps it from arriving**.

A **subagent** runs in its **own conversation**. Its **intermediate tool calls and results** stay there, and only its **final message returns** to the parent, so a subagent can **read dozens of files** without any of that content landing in the caller's window.<sup>[7]</sup> Send the large read somewhere else and you get the answer without the journey.

**Separate agents share no context**. The only content crossing from parent to subagent is the **prompt string** on the tool call that launched it, which means **file paths, error text and decisions** already taken are present only if something put them there.<sup>[7]</sup> The **assumed handoff** is the anti-pattern: designing as though a later step can see what an earlier one found, because both happened inside one run. Reliability at a boundary comes from **writing the findings into the prompt**.

**Multi-step workflows** buy the same isolation without separate agents, by giving each step only the **material its own job needs**. Reviewing a long document, **narrow passes over each section** catch local errors, while a **contradiction between two sections** is visible only to a pass whose scope is that comparison. Running the whole document three times and voting repeats one attention problem three times, and agreement among humans or model passes on something everybody missed is **not detection**.

Chapter 12 covers choosing between a workflow and an agent, and chapter 13 covers building the loop. What **isolation costs is visibility**: the **subagent's reasoning goes** when its context goes.

## Prompt Engineering Writes One Exchange; Context Engineering Curates the Whole Run

Everything in this chapter has a name, and it is not the name most people reach for.

The two terms get used as synonyms and they describe **different jobs at different scales**. **Prompt engineering composes a single exchange**: the **wording of an instruction**, the examples beside it, the shape the answer has to take. Its unit is **one request** and its question is **what to say**. Anthropic's own guidance is scoped that way and says so, noting that not every failing result is best solved by changing a prompt at all.<sup>[9]</sup>

**Context engineering decides** what occupies the **window across a run**: what gets fetched, what survives the next turn, what is compacted, what is handed to a subagent so it never lands here. Its unit is the **whole session** and its question is **what should be present**, which is a different question from what to say. Anthropic frames the window as a **finite resource** with diminishing returns, where every extra token draws on the same **attention budget**.<sup>[4]</sup>

On one call the two collapse into each other, which is why the terms blur. They separate the moment a task runs long. A prompt can be **word-perfect** and the run still **degrades**, because by turn forty the instruction is **competing with thirty-nine turns** of material that finished its job long ago. No rewrite of the instruction reaches that. **Length is the discriminator**: **short and single-turn**, the lever is the **wording**; **long and multi-step**, the lever is **what you let accumulate**.

## How Wrong Answers Are Built on This Material

Wrong answers here are **competent engineering** aimed at the **wrong quantity**. **Four assumptions** produce nearly all of them.

- **Capacity offered where competition is the problem**. A **larger window**, a **bigger model**, **extended context**. Each answers material that does not fit, and none touches material that fits and **drowns out what matters now**.
- A **frequency dial offered against a loss**. Raising a **compaction threshold**, lowering it, compacting on a schedule. All three change **how often summarisation runs**, not **what it may discard**.
- **Continuity destroyed to remove volume**. Restarting every few calls, truncating to the latest turn, dropping tool calling. **Growth stops** because the **task state went with it**.
- **Context assumed to travel**. Treating a **later step**, a **separate agent** or a **post-compaction turn** as able to see what an earlier one established. Nothing carries unless **something carries it**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| An agent degrades after a fixed number of tool calls, window already large enough | Prune spent tool output or compact, protecting the current task state | A larger window, a different framework, merged tools |
| A rule is violated only after several compactions | Move it outside what compaction may rewrite: system prompt or a reinjected record | A higher threshold, or an instruction to remember it |
| History is resent in full and requests grow steadily | Summarise or trim older turns, keeping what is still in use | Capping the reply length, or discarding history outright |
| A step must reason without seeing the whole prior conversation | Scope each step to what it needs, by subagent or multi-step workflow | One global prompt handed to every step |
| A corpus far larger than any window is proposed for every prompt | Retrieve the relevant passages per query | A document-count limit, or splitting the corpus across parallel requests |
| A later step depends on something an earlier one established | Pass it explicitly in the prompt that starts the step | Assuming it is visible because it happened in the same run |
| Answers keep leaning on stale specifics from many turns ago | Compact, so the gist stays and the specifics lose weight | Truncating to the latest turn, or resetting every turn |
| A prompt is already precise and a long multi-step run still drifts | Change what the window holds, not what the instruction says | Rewriting or lengthening the instruction, which cannot reach material that accumulated around it |

One habit transfers past the table. For any option offered, ask whether it changes how **much room exists**, how **often something is discarded**, or what may be **discarded**. Drift answers only to the third.

## What to Remember

- The window is **one budget** drawn on by everything: **system prompt**, every message including **tool results, images, documents, tool definitions**, and the turn's own **output and thinking**. **Cached prefixes still occupy** it; caching changes **price, not space**.
- **Spent context** is material that **already did its job** and is **still resent every turn**, still **costing tokens** and **competing for attention**. It is the bulk of a long agent run.
- **Bloat is size** and **drift is behaviour**. Bloat raises cost and eventually a **`400`**; drift **reports nothing**, showing as **stale detail, dropped constraints and degrading tool choice**. **Tool selection failing** after a fixed number of turns is a **window symptom**, not a **schema symptom**.
- A **bigger window does not treat drift**, because it **preserves every distraction** at full detail.
- Two levers, two mechanisms: **clearing removes old tool results**, leaving a **placeholder**, tuned by **`keep`, `clear_at_least` and `exclude_tools`**; **compaction rewrites history** into a summary and **ignores every block before** it, costing an extra billed **sampling step**.
- Compaction is **lossy by design**, so anything that must **hold for the whole run** lives outside it: **system prompt**, or a **record reinjected each turn**. **Restating the rule** in the conversation puts it back inside the **region being summarised**, and threshold changes **alter frequency**, never what a **summary may drop**.
- **Isolation keeps material out** rather than shrinking it: a subagent's **tool calls and results** stay in its **own context**, only its **final message returns**. **Separate agents share no context**, so the **launching prompt** is the **only channel** and what it omits is absent. The price is **visibility**.
- **Prompt engineering and context engineering** are **different jobs at different scales.** Prompt engineering **composes one exchange**, its unit a **single request** and its question **what to say**. Context engineering **decides what occupies the window** across a run, its unit the **whole session** and its question **what should be present** at all. **Length separates them**: on one call they collapse together, and on a long run only the second reaches the problem.

## Endnotes for this chapter

1. Everything in the request counts toward the context window: the system prompt, every message in `messages` including tool results, images and documents, and the tool definitions; the output Claude generates for the turn, including its extended thinking, counts too. Cached prompt prefixes still occupy the context window, because prompt caching changes what you pay for those tokens rather than whether they count. If the input alone already exceeds the model's context window, the API returns a `400` `invalid_request_error`. The context window functions as working memory, and accuracy and recall degrade as the token count grows, a phenomenon named context rot. Anthropic, *Context windows* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/context-windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)

2. The `clear_tool_uses_20250919` strategy clears tool results when conversation context grows beyond a configured threshold, clearing the oldest tool results in chronological order and replacing each cleared result with placeholder text indicating to Claude that it was removed; its `trigger` defaults to 100,000 input tokens, `keep` defaults to 3 tool uses, `clear_at_least` ensures a minimum number of tokens is cleared each time and withholds the strategy if that minimum cannot be met, `exclude_tools` lists tool names whose uses and results should never be cleared, and `clear_tool_inputs` controls whether tool call parameters are cleared alongside the results; the `clear_thinking_20251015` strategy manages thinking blocks on the same footing; context editing is applied server-side before the prompt reaches Claude, and the client application maintains the full unmodified conversation history with no need to sync; tool result clearing invalidates cached prompt prefixes when content is cleared. Anthropic, *Context editing* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/context-editing](https://platform.claude.com/docs/en/build-with-claude/context-editing)

3. Compaction automatically summarizes older context when approaching the context window limit, enabled by adding the `compact_20260112` strategy to `context_management.edits`; its `trigger` defaults to `{"type": "input_tokens", "value": 150000}`, `input_tokens` is the only supported trigger type, and `value` must be at least 50,000 tokens; the API returns a `compaction` block containing the summary, the block must be passed back on subsequent requests, and when the API receives it all content blocks before it are ignored; compaction requires an additional sampling step which contributes to rate limits and billing; the model specified in the request is used for summarization, with no option to use a different or cheaper model; custom instructions supplied through the `instructions` parameter do not supplement the default prompt but replace it completely. Anthropic, *Compaction* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/compaction](https://platform.claude.com/docs/en/build-with-claude/compaction)

4. Context engineering is described as the set of strategies for curating and maintaining the optimal set of tokens during inference; context is treated as a finite resource with diminishing marginal returns, drawing on a limited attention budget that every additional token depletes, and the stated objective is finding the smallest possible set of high-signal tokens that maximise the desired outcome. Compaction, structured note-taking in external memory, and sub-agent architectures that return condensed summaries are given as the long-horizon techniques. Anthropic, *Effective context engineering for AI agents* (Anthropic Engineering, accessed 2026-09-09), [https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

5. Tool definitions and accumulated `tool_result` blocks consume the context window, and four approaches address this at different points: tool search keeps tool definitions out of the context window until Claude asks for them, trading one extra turn of latency for a large reduction in baseline context usage; programmatic tool calling collapses a sequence of tool calls into a single script run in a sandbox so the intermediate results never enter the conversation history; prompt caching reduces what you pay for tokens already in context rather than reducing their number; and context editing removes old `tool_result` blocks once they have served their purpose. The approaches compose. Anthropic, *Manage tool context* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/agents-and-tools/tool-use/manage-tool-context](https://platform.claude.com/docs/en/agents-and-tools/tool-use/manage-tool-context)

6. The token counting endpoint accepts the same structured list of inputs used for creating a message, including system prompts, tools, images and PDFs, and returns the total number of input tokens without running inference; it is supported on all active models and is free to use, subject to requests-per-minute rate limits. Anthropic, *Token counting* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/token-counting](https://platform.claude.com/docs/en/build-with-claude/token-counting)

7. Because subagents are separate agent instances, each subagent runs in its own conversation which starts fresh unless it is a fork; intermediate tool calls and results stay inside the subagent and only its final message returns to the parent, so a research subagent can explore dozens of files without that content accumulating in the main conversation. A non-fork subagent's context window starts fresh with no parent conversation, and the only content passed from parent to subagent is the Agent tool's prompt string, so any file paths, error messages or decisions the subagent needs must be included directly in that prompt. Anthropic, *Subagents in the SDK* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/agent-sdk/subagents](https://code.claude.com/docs/en/agent-sdk/subagents)

8. When a long session compacts, Claude Code summarizes the conversation history to fit the context window, and what happens to instructions depends on how they were loaded: path-scoped rules and nested CLAUDE.md files load into message history when their trigger file is read, so compaction summarizes them away with everything else, and a rule that must persist across compaction should have its `paths:` frontmatter dropped or be moved to the project-root CLAUDE.md; skill bodies are re-injected after compaction, but large skills are truncated to fit a per-skill cap and the truncation keeps the start of the file, so the most important instructions belong near the top of `SKILL.md`. Delegating large reads to a subagent keeps the file contents in the subagent's context window rather than the caller's. Anthropic, *Explore the context window* (Claude Code Docs, accessed 2026-09-09), [https://code.claude.com/docs/en/context-window](https://code.claude.com/docs/en/context-window)

9. Prompt engineering guidance is scoped to success criteria that are controllable through prompting, and states that not every success criterion or failing evaluation is best solved by prompt engineering. Anthropic, *Prompt engineering overview* (Claude Platform Docs, accessed 2026-09-13), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview)
