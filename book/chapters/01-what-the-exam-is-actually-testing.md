# Chapter 1: What This Exam Tests That an Interview Does Not

**Summary**: *An interview asks whether you can build the thing. This exam asks whether you reached for the right mechanism, and almost every wrong answer it offers is a mechanism that works perfectly well somewhere else. That is the difficulty, and it is not a difficulty you can read past. This chapter sets the distinction up on the smallest material in the syllabus: what a token is, what the window is, why one request run twice returns two different answers, and which dial does which job. Four facts. A surprising number of later questions collapse once you hold them straight.*

## The Same Request Twice Returns Two Different Answers, and Nothing Is Broken

Send a request. Send it again, **byte for byte identical**. The two responses will often **differ in wording**, sometimes in structure, occasionally in whether a particular caveat shows up at all.

The **reflex** is to **open a ticket**. Something **upstream must be varying**: a stray timestamp in the prompt, a config that got redeployed, two versions of the code running behind a load balancer.

Usually **nothing is varying**. **Generation proceeds one token at a time**, and every one of those choices is **sampled rather than looked up**. Two runs can therefore diverge at any single position and stay diverged for the rest of the response. Anthropic states the strong form of this plainly: **identical inputs can produce different outputs** even with **temperature set to 0**, on Anthropic's own inference service and on third-party providers.<sup>[1]</sup> Temperature 0 is not a **determinism switch**. It narrows the distribution. It does not collapse it to a point.

**Humans absorb the rewording** without noticing it happened. A **parser** does not, and neither does a **downstream step** that expected a field where a sentence now sits.

That is the first place this **exam separates from an interview**. An interviewer watching you debug would accept "it varies, that is expected" and move on. A scenario here will **describe the variation**, put a **plausible defect** beside it, and ask what you tell the teammate who found it. The **scored answer** is the one that **names the mechanism** and then says what the **application** does about it, because a developer who stops at "that is expected" has **explained the behaviour** and **shipped nothing**.

What the application does about it is **validation**, a **retry on failure**, and **sampling settings** where **consistency** matters more than **variety**. None of those make the model repeat itself. They make the **surrounding system tolerant** of the fact that it will not.

Hold on to the shape of that answer. It **recurs**: a **property of the technology**, then the **component that absorbs** it. An option that offers only the first half is **describing the world**. An option that offers only the second half is treating a **permanent property as a bug** with a fix.

## A Token Is the Unit You Are Billed In, Limited By, and Debugging Against

A **token** is the **smallest unit** the model works in, and it maps to **words, subwords, characters or bytes** rather than to anything as tidy as one word each. For Claude, a token is **roughly 3.5 English characters**, and the ratio moves with the language.<sup>[2]</sup>

That definition is worth more than it looks, because **three separate things** in this syllabus are **measured in tokens** and people who have not internalised the unit treat them as **three unrelated topics**.

| What is counted in tokens | Where it bites |
|---|---|
| Everything you send, on every request | The bill, and how much of the window is left |
| Everything the model generates | The bill again, at a different rate, and the output cap |
| Everything the model spends reasoning before it answers | The bill a third time, on work no human ever reads |

The **third row** is the one that surprises people, and it is why the **effort setting** exists at all.

## The Window Is Capacity, and Temperature Is Selection

The **context window bounds** how much material one request may carry. It is finite, and it behaves as **working memory rather than storage**: **accuracy and recall degrade** as it fills, which Anthropic names **context rot**.<sup>[3]</sup>

Now the single most **productive confusion** to clear up in this chapter, because it appears in **scenarios repeatedly** and it is always offered by a **colleague who sounds confident**.

**Temperature** does not **affect capacity**. It **decides which candidate wins** at each step. It has no bearing whatsoever on how much **text a request can carry**. So when a **transcript will not fit** and a colleague proposes **raising the temperature**, they have not proposed a weaker fix. They have proposed acting on a part of the request that has no bearing on the problem. Turn it up or down and the wording of the failure varies while the **failure stands**.

| Dial | What it changes | What it cannot change |
|---|---|---|
| Temperature and sampling settings | Which token gets chosen from the candidates | How much input fits, and what the response costs |
| Context window | How much material one request can hold and cross-reference | How varied or repeatable the wording is |
| `max_tokens` | How long the response may run | How much input fits |
| Effort | How many tokens are spent reasoning before answering | How much input fits |

Read that table as **four separate machines** that happen to sit on the same request. The **capacity-versus-selection confusion** is the **named anti-pattern** here, and it is worth naming because it also runs the other way: a scenario where output wording is **inconsistent** is not repaired by **trimming the input**.

One **consequence** falls straight out and settles a whole class of question. When a scenario needs a **long document held together**, so that a reference in section nine resolves against something in section two, both sections have to be in the **same request**. That is **capacity**. No **sampling setting**, no **latency improvement** and no **output cap** has anything to say about it.

## Reasoning Depth Is a Separate Setting From Model Choice

Developers arriving at this material tend to assume there is one **dial** called "how good do you want it", and that the dial is the **model name**. There are **three settings**, they are **independent**, and **scenarios pull them apart** deliberately.

The **model** decides the **capability tier**. **Thinking** decides whether the model **reasons visibly before answering**, and **adaptive thinking** lets the model judge how much of that a given request warrants. **Effort** controls how many tokens Claude spends when responding, trading **thoroughness against token efficiency** on a single model, and it is available on all supported models with no beta header.<sup>[4]</sup> **Fast mode** is different again: it buys up to **2.5 times higher** output **tokens per second** on supported Opus models at **premium pricing**.<sup>[5]</sup>

Two of those are **quality dials**, one is a **speed dial**, and they are **priced differently**. That is the distinction a scenario tests.

| Setting | Buys you | Costs you |
|---|---|---|
| A larger model | Capability on genuinely harder work | Money per token, and usually latency |
| Thinking, or adaptive thinking | Reasoning before the answer, on work that needs it | Tokens the user never reads |
| Effort | Control over how much of that reasoning happens | Thoroughness, when turned down |
| Fast mode | Output tokens per second | Premium pricing, on supported models only |

There is a **practical trap sitting** under this, and it is mechanical rather than conceptual. **Toggling thinking modes invalidates prompt caching**.<sup>[6]</sup> A team that flips thinking on for a **subset of requests**, against a **large stable prefix** they had been paying to reuse, has quietly **given up the reuse**. That is the kind of interaction between two features that an interview never asks about and this exam does.

## Three Prompting Modes, Priced in Tokens

The last piece of **foundation** is the **cheapest technique** in the book, and it is **under-reached-for** because it **feels too simple** to be the answer.

**Zero-shot** describes the task and **supplies no examples**. **One-shot** supplies a **single worked example**. **Multi-shot** supplies **several**. Moving up that list **trades tokens for adherence**. A description states a rule and leaves its margins to interpretation; a worked instance shows one margin already decided. Where prose keeps producing the wrong capitalisation, or the wrong treatment of an awkward input, one decided case settles what another paragraph of instruction has not. The **cost recurs**, because the **examples ride along** with every request.<sup>[7]</sup>

The **scenario shape** to recognise: a team holds a **small set of labelled examples** demonstrating exactly the output they want, and the options offered are to **put them in the prompt**, to **store them in a database**, to **train a custom model**, or to **warn users away** from the hard cases. Those examples are already what the technique runs on. Putting them in the prompt is the **shortest route** from what the team holds to the output it wants, and nothing has to be built before anyone can try it.

**Reaching past the cheap mechanism** is the **anti-pattern**, and it will be **dressed as rigour**. Training a custom model on a handful of labelled cases is not more serious than showing the model those cases. It is the **same information**, spent through a far more **expensive channel**, with a **deployment** to maintain afterwards.

## How Wrong Answers Are Built on This Material

Wrong answers here are **rarely absurd**. They describe **mechanisms that exist**, that work, and that a competent developer uses weekly. What makes one wrong is a **single assumption** underneath it, and on this material there are **three worth memorising**.

- A **generation setting offered against a capacity problem**. **Temperature, `max_tokens` and sampling** all act on how the **response is produced**. None of them changes how much **input a request can carry**. Whenever a scenario is about **material that does not fit**, every option naming a sampling parameter is **wrong on mechanism**, not on **degree**.
- **Expected behaviour diagnosed as a defect**. **Run-to-run variation in wording**, with the **request unchanged**, is how **generation works**. An option that **confirms a bug**, **tickets** it, or **proposes pinning something** to eliminate it has **accepted a false premise**. The **corollary** matters too: an option that **explains the mechanism and stops** there has not answered a question that asked what to do.
- A **heavier mechanism preferred** to the one the **material already fits**. **Training, retrieval infrastructure and separate applications** all turn up as answers to problems that a few **examples in the prompt would settle**. **Cost is the tell**. Ask what the **option needs built** before it can be **tried** at all.

The table below is the **compressed form**. **Match** on what the **scenario** is doing, not on the **industry** it is set in.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Identical requests return differently worded responses | Name sampling as the mechanism, then say how the application validates, retries or constrains | Confirming a bug in request construction, or blaming the model version |
| A document is too long to process, or references across it are being lost | Capacity: fit it in one request, or split and carry state deliberately | A sampling parameter, a latency improvement, or a larger output cap |
| A small set of labelled examples of the desired output already exists | Put them in the prompt as examples | Training a custom model, or storing them somewhere the model never reads |
| Output quality is uneven on work that needs reasoning | Thinking or effort, chosen against what the tokens buy | A larger model offered as the only quality dial |
| Response speed is the stated problem, and quality is fine | The speed mechanism, priced accordingly | Anything that changes what the model produces rather than how fast |
| A stable prefix is being reused across many requests | Preserve the conditions the reuse depends on | Toggling a setting that silently invalidates it |

One habit is worth more than the table. When two options both **look reasonable**, ask which **mechanism** each one **acts** on, then check which **mechanism the scenario named**. On this material the wrong option is almost always operating on a **real mechanism** that the **scenario never mentioned**.

## What to Remember

- A **token is the unit** of **billing**, of the **window**, and of **reasoning spend**, worth roughly **3.5 English characters** in Claude. **Input, output and thinking** are all counted in it, at **different rates**.
- Each **token is chosen by sampling**, so one **request run twice** can **return different text**, and that holds **even at temperature 0**. It is a **property of how generation works**, not a **defect** in how the request was built.
- **Temperature selects; the window holds**. A **capacity problem never responds** to a **sampling parameter**, and **inconsistent wording** never responds to **trimming the input**.
- The **context window is working memory, not storage**: it is **finite** and **recall degrades** as it fills, which Anthropic calls **context rot**. **Anything that must be cross-referenced** has to be in the **same request**.
- **Model, thinking and effort** are **three independent settings**, not one **quality dial**. **Fast mode** is a **speed setting**, buying output tokens per second at **premium pricing** rather than capability.
- **Toggling thinking modes invalidates prompt caching**, so a **per-request thinking decision** can **silently cancel the reuse** a large stable prefix was earning.
- **Zero-shot, one-shot and multi-shot** trade **tokens for adherence**. When a team already **holds labelled examples** of the output they want, **showing the model those examples** is the **cheapest mechanism that fits**, and reaching past it is the **standard wrong answer**.

## Endnotes for this chapter

1. Identical inputs can produce different outputs even with temperature set to 0, on Anthropic's own inference service and on third-party providers. Anthropic, *Glossary* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/glossary](https://platform.claude.com/docs/en/about-claude/glossary)

2. Tokens are the smallest individual units of a language model and can correspond to words, subwords, characters or bytes; for Claude a token approximately represents 3.5 English characters, varying by language. Anthropic, *Glossary* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/about-claude/glossary](https://platform.claude.com/docs/en/about-claude/glossary)

3. The context window functions as working memory rather than storage, and accuracy and recall degrade as the token count grows, a phenomenon Anthropic names context rot. Anthropic, *Context windows* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/context-windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)

4. The effort parameter controls how many tokens Claude spends when responding, trading response thoroughness against token efficiency on a single model, and is available on all supported models with no beta header required. Anthropic, *Effort* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/effort](https://platform.claude.com/docs/en/build-with-claude/effort)

5. Fast mode delivers up to 2.5 times higher output tokens per second from supported Claude Opus models, at premium pricing. Anthropic, *Fast mode* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/fast-mode](https://platform.claude.com/docs/en/build-with-claude/fast-mode)

6. Toggling thinking modes invalidates prompt caching. Anthropic, *Thinking* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/thinking](https://platform.claude.com/docs/en/build-with-claude/thinking)

7. Prompting techniques including the use of worked examples are documented as a means of improving adherence to a required output shape. Anthropic, *Prompt engineering overview* (Claude Platform Docs, accessed 2026-09-09), [https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview)
