# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked — by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

One reason holds: the independent validator. Cortex can't grade its own draft, the same "mind" that wrote the update would inherit its own blind spots if it also checked itself.

Ruled out: Parallelism doesn't apply (drafting and proposing stories don't need to run concurrently). Context-window pressure doesn't apply (the pulled data doesn't strain Cortex). Separation of concerns doesn't apply as a distinct reason, it collapses into the same validator point: drafting doesn't contaminate story-proposal, and "drafting vs. checking" *is* the validator split, not a second reason.

## 2. Topology

**Pattern:** single+subagents

```
[Inbound PM task (hook)] → [Cortex: pulls data, drafts update + proposes stories]
                          → [Validator/Critic]
     fail → back to Cortex (max 2 revisions) → escalate to human
     pass → [PM review checkpoint] → queued, never posted
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex (chief-of-staff) | Pulls project data, drafts status update, proposes stories | M2 loop (hook + cron backup) |
| Critic / Validator | Checks Cortex's draft against the 6 rules before it reaches a human | One independent call per draft, fresh context |

## 4. Communication & hand-offs

Cortex passes `{proposed draft text, source_log (the concatenated tool-call results it used)}` to the critic via a plain in-process Python function call (`review()`), not MCP/A2A. The critic passes back `{verdict: pass/fail, reasons: [...]}`.

## 5. The validator

- **What the critic checks:** All six, (1) correct project + real PR/issue IDs, (2) every claim traceable to pulled data, no invented numbers, (3) stays within team norms (no unconfirmed date, no launch gate, no CONFIDENTIAL leak), (4) posts/commits/creates nothing, (5) refuses and escalates on jailbreak attempts, (6) treats a tool/bound rejection as a correct escalation, not a failure to override.
- **Fail action:** Revise, tiered into escalate, Cortex gets the critic's reasons and tries to fix the draft first, up to the revision cap. Only after that cap is hit does it escalate to a human.
- **Revision cap:** 2, to keep cost and latency bounded, a critic that bounces forever is a cost/latency bomb. Confirmed live in this session: hitting 2/2 rejections reliably triggers escalation, never a 3rd attempt.
- **Pass action:** Advances to the PM review checkpoint. Never auto-sent, Cortex has no post/send tool at all, so this is enforced in code, not just instructed.

## 6. State: shared vs isolated

**Shared:** the draft + source data (the critic needs to see exactly what Cortex saw, to judge groundedness).

**Isolated:** Cortex's own conversation history/deliberation stays isolated from the critic, it gets a fresh context each call (confirmed in `critic.py`, no shared message history). The critic's full reasoning (the `reasons` list, which is its complete output, no hidden deliberation) does feed back into Cortex on a fail, that's how Cortex knows what to fix.

## 7. Cost & latency budget

The validator adds 1 critic call per drafting attempt, a full separate API round-trip, not free.

**Worst case at the revision cap (2):** up to 3 critic calls total (1 initial + 2 after revisions), roughly tripling the model-call count for that run.

**Observed real cost range** this session: $0.0017 (clean pass, no revisions) up to $0.0055 (hit the full revision cap), so worst case is roughly 3x the cost of a clean pass.

**Latency:** each critic call is an extra network + inference round-trip before a draft can reach the PM, worst case adds the time for 2 extra critic calls + 2 extra drafting revisions on top of the base loop. This becomes a bound to enforce explicitly in M5.
