# Chapter 22: One Credential: What It Reaches, and for How Long

**Summary**: *Treat a secret as a population rather than a value: however many copies of it exist, each with its own reach and its own remaining life. Three measurements decide what a leak costs, and every practice on this skill moves exactly one of them. How many copies exist. What one copy authorises on arrival. How long it keeps working once somebody else holds it. A fourth question decides whether anybody ever finds out, and it is the one with no symptom when the answer is nobody.*

## The Value Is Shown Once and Then Comes to Rest Wherever It Was Needed

Create an API key in the Console. The **full secret** is **displayed once, at creation**. A **key you lose** cannot be viewed again.<sup>[1]</sup> That single display is a **decision about copies**. The **platform holds nothing** you can read back, so every copy from then on exists because somebody made one.

Follow one key through a week of setup. It goes into a **shell profile** so a local run works. A deployment **configuration takes a copy**. Somebody pastes it into a message to unblock a colleague, a runner caches it, a **backup captures the runner**. **Six copies**, none of them wrong when it was made, and **no inventory** of where they sit.

Which leaves the only question that **decides what a leak costs**. What does one of those **six reach**, and how long does it keep working?

## Three Measurements Decide the Damage, and Each Practice Moves One

Call that inventory the **copy set**: the places a credential's value has **come to rest**, together with the access each of those places carries. Every decision you make here acts on it. **Count** is how **many copies exist** and who can read each. **Reach** is what one **copy authorises on arrival**. **Life** is how long it **keeps working** once somebody else holds it.

| Practice | Which measurement it moves | What it leaves standing |
|---|---|---|
| Keeping the value out of the file that references it | Count | Reach and life of every copy that remains |
| A managed store read at runtime | Count, and it produces a read record | Reach |
| Distinct, narrowly privileged credentials per environment and per service | Reach, downward | Count, which it raises |
| An expiration chosen at creation, and scheduled rotation | Life | Count and reach |
| Access records and usage alerting | None of the three | All three |

Read the **right-hand column** rather than the left. Every **practice leaves something standing**, which is why an option naming one row and stopping answers a **narrower question** than a credential review asks.

## A Value Written Into a Tracked File Survives Its Own Deletion

Separation is the rule that a **configuration file holds a reference** and **never a value**. The reason is not tidiness. A tracked, cloned, reviewed or backed-up file carries what sits inside it to all of those places, and overwriting the value later **edits the current version only**. History keeps the rest, in every clone taken before the repair, which is why **overwrite mistaken for a removal** is the anti-pattern and why a credential ever written inline has to be treated as **exposed and replaced**.

The value needs a home instead. You choose between two. An **environment variable injected at execution** suits a secret that is **local and short-lived**: one process, one run, nothing on disk. A **managed secret store** suits a secret **several services need**, since it **collapses scattered copies into one** and records who asked for it.<sup>[1]</sup> Stronger still is taking the value out of the workload: a **proxy outside the agent's boundary** injects credentials into outgoing requests, a vault substitutes the real secret for an opaque placeholder at egress, and subprocess environments can be stripped of provider credentials. The code doing the work **never holds the value**.<sup>[4]</sup><sup>[7]</sup><sup>[8]</sup>

## Three Properties Are Fixed at Creation and Changed Only by Issuing Another Key

Reach is **decided twice**. Once by how many things **one credential serves**, and once by what it is **permitted** to do.

A credential shared across development, staging and production is a **path between environments separated on purpose**, and it **turns one compromise into three**. Replacement gets expensive too: one object serving everything cannot be replaced anywhere without being replaced everywhere. **Shared key across environments** buys a simplicity that costs isolation and rotation together. **Distinct credentials per environment and per service** raise the count deliberately and shrink what any one is worth to whoever takes it.

Privilege narrows the rest of it. The timing of that choice is unusually plain. A key's **workspace, its expiration, and any scopes** the product lets you select are **fixed when the key is created**, and the only way to change one afterwards is to **issue a different key**.<sup>[1]</sup><sup>[2]</sup><sup>[3]</sup> **Console Admin API keys** carry **no selectable scopes** at all: each holds full access to every endpoint that accepts them, and a key issued in one organization cannot manage another.<sup>[3]</sup> The moment you issue one is the **entire authorisation decision**. **Approval belongs at issue time**: only an organization admin can mint an admin credential, and the create control is unavailable to a requester with no permission in that workspace.<sup>[1]</sup><sup>[3]</sup>

## Rotation Is the Only Repair, and Expiry the Only Version That Runs Unattended

A **leaked value cannot be made secret** again. No operation unknows it. That leaves one repair. Issue a replacement and revoke the old one, which is what Anthropic tells you to do on both surfaces: keep keys in a **secrets manager, rotate them periodically, revoke any key** you think may have leaked.<sup>[2]</sup><sup>[5]</sup>

**Scheduled rotation** and **rotation on suspicion** do different jobs. The schedule **bounds an undetected leak**. The immediate one is incident response, and its cost falls as the copy set shrinks, because a consumer reading a name needs no change when the value behind it is replaced. Rotation that **returns the value to its old home** wastes both: writing a fresh key straight back into the file it came from clears the error and **rebuilds the exposure**.

Expiry is the version of life that runs without anybody remembering. It is chosen when the key is created; once it passes, **requests fail** and the **key cannot be reactivated**. It bounds a **leaked credential's usable life** while substituting for storage and rotation in nothing.<sup>[2]</sup> **Federation removes** the durable value instead. A workload trades a trusted provider's identity token for a **short-lived access token**, so **nothing static is minted, distributed or rotated**, and the chain is then worth what that provider's configuration is worth.<sup>[2]</sup>

## An Asserted Identity Is a Claim Until Something Verifies the Signature

Sessions in a self-hosted environment carry a **signed token** that services you operate can check. A valid token proves that Anthropic issued it for a **specific session in a specific environment**, and how that session was created. It **proves no more**. The **token does not establish** which process presented it, because it sits in an **environment variable** that any code the session runs can read.<sup>[6]</sup>

So verification is mechanical rather than assumed. Check the **signature against the published key set**, refuse an **unexpected algorithm**, and check the **audience claim** against your own environment identifier. Verification is **also offline**. A token that verifies stays valid until it expires, whatever has happened to the session since, and **no revocation feed** exists for it.<sup>[6]</sup>

Then the part identity does not settle. **Identity accepted as authorisation** is the anti-pattern, and it shows up wherever a verified caller is **granted whatever that caller holds elsewhere**. The token names the human who created the session. It is **not that human signing in**. Credentials derived from it get the capabilities one session needs and a life bounded by the **token's own expiry**.<sup>[6]</sup>

## Nobody Outside Your Organization Can Tell You a Key Has Leaked

Anthropic states the limit plainly. **Anomalous usage patterns are detectable** from the platform side, and whether a **particular key was compromised is not**.<sup>[5]</sup> That knowledge is produced locally or not at all, and three records answer three different questions. A **managed store logs who read a value**, which narrows the copy set after the fact. The Admin API lists every key with its expiration, returning a **redacted hint and never the secret**, so an inventory can be audited and rotated ahead of expiry.<sup>[1]</sup><sup>[2]</sup> And where work runs under a derived credential, recording the **session identifier beside the human identity** resolves a question about one action to one session rather than to a busy person.<sup>[6]</sup>

**Credential nobody watches** is the anti-pattern that **fails quietly**. Every other control here shows its work at the moment it refuses something. Monitoring is the only one whose absence **looks exactly like success**.

## How Wrong Answers Are Built on This Material

Wrong answers on this skill **describe real practices**. Each **improves something**, which is why discarding options by **obvious nonsense** does not work here.

- **One measurement improved and the others left alone**. A key moved out of source control into **plaintext build configuration**. A shared credential **rotated on a schedule and still shared**. Each improves on what preceded it and **leaves the copy set as wide** as it was. Both become correct where the scenario **asks what to fix first**, and lose where it asks which **posture is strongest**.
- A **single shared object presented as an advantage**. **One credential everywhere** really is simpler: one thing to store, one thing to replace. That is also the **defect stated out loud**. Ask what a **single replacement costs** and how many **environments one compromise crosses**.
- A **control that acts** after the **value has already travelled**. **Deleting the committed value**, **restricting which networks** may use the key, **tightening review**. None of them **reduces the copies already made** or shortens the life of the one somebody took.
- A **verified identity spent as an authorisation**. A caller whose token checks out is **handed everything that caller holds elsewhere**. Verification answers who is asking and **never answers what may be done**.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| A review of credential handling across several environments | Move every measurement at once: separate, scope narrowly, store centrally, rotate on schedule and on suspicion | One practice named as sufficient, or one credential defended for its simplicity |
| A credential is sitting inside a tracked configuration file | Treat it as exposed, replace it, and leave a reference where the value was | Overwriting the value in a later change, as though history forgot |
| One key serves development, staging and production | Distinct credentials per environment and per service | Rotating the shared one more often, or accepting the crossover as a fair trade |
| Several services each need a key and somebody asks where to keep them | A managed store, or configuration outside source control, loaded at runtime | Anywhere convenient that keeps a permanent copy nobody can revoke |
| A request arrives carrying a signed session token | Verify signature, issuer and audience, then grant only what that session needs | Treating the named user's standing access as the session's access |
| Somebody asks whether a credential has been misused | Records: who read the value, which session acted, what the key inventory says | Assuming the platform would have raised it |

One habit transfers past the table. For any option you are offered, **name which measurement it moves** and which one it leaves where it was. The **strongest answer** here is nearly always the one whose description **needs all three**.

## What to Remember

- A **credential is a copy set**: the **places its value came to rest**, plus the **access each of those places carries**. Three measurements decide the damage: **count** of copies, **reach** of one copy, and **life** before it stops working.
- A value written into a **tracked file enters history**, so overwriting it later **edits the current version only**. Treat any **inline credential as exposed** and **replace** it, because it persists in every **clone taken before the repair**.
- **Configuration holds a reference** and **never a value**. An **environment variable** fits a secret **one process needs for one run**; a **managed store** fits one that **several consumers need** or that must be audited, since a store **collapses scattered copies** into one and **records who read** it.
- **Distinct credentials per environment and per service** are what make isolation real. A shared key is a **path between environments separated on purpose**, and it makes replacing it an **outage everywhere**.
- **Rotation is the only repair** after exposure, since a **leaked value cannot be made secret** again. Rotate on a **schedule** to bound an undetected leak and **immediately on suspicion**; writing the **replacement back into the file** it leaked from reproduces the defect. **Expiry bounds** a leaked credential's usable life and **replaces neither storage nor rotation**.
- A key's **workspace, expiration and any scopes** are **fixed at creation** and changed only by **issuing another key**, so approval belongs at **issue time**. Console **Admin API keys** carry **no selectable scopes** and hold **full access** to every endpoint accepting them, which makes issuing one the **whole authorisation decision**.
- **Federation removes the durable secret** rather than shortening its life: a workload exchanges a **provider-issued identity token** for a **short-lived access token**, with **nothing static** to **mint, distribute or rotate**. It is worth what the **identity provider's configuration** is worth.
- A **verified identity is authentication and not authorisation**. A signed session token proves who **created the session** and **not which process presented** it, so **verify signature, issuer and audience**, and scope **derived credentials** to that session's task with a life bounded by the **token's expiry**.
- Use of a credential is **observable only where somebody recorded** it. The platform can **detect anomalous usage** and cannot **know a key was compromised**, so **read logs, key inventories and session identifiers** are what turn assumed access into **evidence**.

## Endnotes for this chapter

1. The Claude Console displays a new API key's full value only once, at creation, and a lost key cannot be viewed again, so a replacement must be created; a key can be scoped to a workspace and given an expiration at creation, and the recommended storage is a secrets manager. The Admin API includes endpoints for managing an organization's API keys programmatically, which require a separate Admin API key and never return a key's secret value, only a partially redacted hint. Anthropic, *Get your Claude API key* (Claude Platform Docs, accessed 2026-09-10), [https://platform.claude.com/docs/en/get-api-key](https://platform.claude.com/docs/en/get-api-key)

2. API keys are static secrets passed on every request, and Anthropic directs that they be stored in a secrets manager, rotated periodically, and revoked whenever a key is suspected of having leaked. Expiration is chosen at key creation from a preset, a custom duration, or Never, and cannot be changed afterwards; after a key expires, requests return a `401` authentication error and expired keys cannot be reactivated; the Admin API reports each key's `expires_at` so keys can be audited and rotated before expiry; and expiration limits the lifetime of a leaked credential but is not a substitute for secret hygiene. Workload Identity Federation lets a workload exchange a short-lived identity token issued by a trusted identity provider for a short-lived Claude API access token, so there is no static key string to mint, distribute or rotate; federation shrinks the blast radius of a leaked credential but does not on its own guarantee end-to-end security, because the trust chain is only as strong as the identity provider's configuration and a long-lived secret one hop upstream can still undermine it. Anthropic, *Authentication* (Claude Platform Docs, accessed 2026-09-10), [https://platform.claude.com/docs/en/manage-claude/authentication](https://platform.claude.com/docs/en/manage-claude/authentication)

3. Only organization members with the admin role can create Admin API keys; Claude Console Admin API keys do not have selectable scopes and every such key carries full access to all endpoints that accept Admin API keys; a key created in one organization cannot be used to manage a different organization; where scopes are selectable they are fixed at creation, and adding a scope later requires creating a new key; and the full secret is displayed only once and should be stored in a secrets manager. Anthropic, *Create an Admin API key* (Claude Platform Docs, accessed 2026-09-10), [https://platform.claude.com/docs/en/manage-claude/admin-api-keys](https://platform.claude.com/docs/en/manage-claude/admin-api-keys)

4. A security boundary separates components with different trust levels, and sensitive resources such as credentials can be placed outside the boundary containing the agent; the recommended approach is a proxy running outside that boundary which injects credentials into outgoing requests, so the agent never sees the actual credentials and credentials are stored in one secure location rather than distributed to each agent. Anthropic, *Securely deploying AI agents* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/agent-sdk/secure-deployment](https://code.claude.com/docs/en/agent-sdk/secure-deployment)

5. The environment service key must be stored in a secrets manager rather than in environment files or sandbox images, and rotated immediately if exposure is suspected; the key is scoped to one environment's work queue, so provisioning a separate workspace and environment per trust boundary limits each key to a single user's sessions instead of a shared pool; Anthropic can detect anomalous usage patterns but cannot know that a customer's key was compromised, so a suspected leak must be revoked and replaced by the customer; and revocation is validated on every request, so it takes effect on the worker's next call. Anthropic, *Security model* (Claude Platform Docs, accessed 2026-09-10), [https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security)

6. A valid session token proves that Anthropic issued it for a specific session in a specific environment and how the session was created, and deliberately does not prove which process on the runner host presents it, since the token sits in an environment variable that any code Claude runs and any tool or MCP server the session starts can read and present. Services must verify the signature against Anthropic's published key set, reject tokens whose algorithm header is not the expected one, and verify the audience claim against their own environment identifier to reject tokens issued to another organization's environment. Verification is offline, so a token that verifies stays valid until its expiry whatever has happened to the session since, and Anthropic does not publish a revocation feed for session tokens. Credentials derived from the token should be limited to what one coding session should reach rather than everything the creating user holds, bounded in lifetime to the token's expiry or shorter, and audited by recording the session identifier and token identifier alongside the user identity. Anthropic, *Verify session identity in self-hosted environments* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/self-hosted-environments-identity](https://code.claude.com/docs/en/self-hosted-environments-identity)

7. Vault credentials of the environment-variable category are keyed by the environment variable name and stored in the sandbox as an opaque placeholder; when the agent initiates an outbound request the placeholder is substituted with the real secret at egress, and the agent never sees the secret value. Anthropic, *Authenticate with vaults* (Claude Platform Docs, accessed 2026-09-10), [https://platform.claude.com/docs/en/managed-agents/vaults](https://platform.claude.com/docs/en/managed-agents/vaults)

8. `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` strips Anthropic and cloud provider credentials from subprocess environments including the Bash tool, hooks and MCP stdio servers, so the parent Claude process keeps those credentials for its API calls while child processes cannot read them, reducing exposure to prompt injection attempting to exfiltrate secrets through shell expansion. Anthropic, *Environment variables* (Claude Code Docs, accessed 2026-09-10), [https://code.claude.com/docs/en/env-vars](https://code.claude.com/docs/en/env-vars)
