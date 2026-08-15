# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner — by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | · |
| Decide relevant context | M | M | L | Above | · |
| Draft the update | H | L | H | Below | · |
| Decide tone/commitment level | L | H | L | Above | · |
| Flag at-risk/escalation | H | L | H | Below | · |
| Choose what to escalate | M | M | M | Below | required |
| Propose a story batch (capped) | H | L | H | Below | · |
| Post an update (single project) | L | M | M | Below | required |
| Approve a company-wide update | L | H | M | Above | · |

## Agent anatomy (sketch)

- **Model:** `gpt-4o-mini` as the default (cheap, fast) — escalate to a frontier model when the critic rejects a draft twice (hits the revision cap) or when the task involves an external/company-wide audience, where higher-stakes reasoning is worth the cost.
- **Tools:** `get_project`, `get_activity`, `search_past_updates`, `get_roadmap`, `get_norms` (all read-only) · `propose_stories` (capped, queued for approval — never auto-committed).
- **Memory:** roadmap, past updates, and team norms persist across runs as the ground truth Cortex checks itself against; nothing about a specific run (draft text, critic verdicts) carries forward.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

1. Pull project state + activity sits below the line because it's highly reversible, has a low blast radius, and is highly measurable — deciding factor: blast radius.
2. Decide relevant context sits above the line because it's only moderately reversible, has a moderate blast radius, and is hard to verify after the fact — deciding factor: measurability.
3. Draft the update sits below the line because it's highly reversible, has a low blast radius, and is highly measurable — deciding factor: reversibility.
4. Decide tone/commitment level sits above the line because it's hard to reverse, has a high blast radius, and is hard to verify — deciding factor: blast radius.
5. Flag at-risk/escalation sits below the line because it's highly reversible, has a low blast radius, and is highly measurable — deciding factor: blast radius.
6. Choose what to escalate is HITL because all three axes are moderate — deciding factor: none dominates, so a human checkpoint substitutes for certainty.
7. Propose a story batch (capped) sits below the line because it's highly reversible, has a low blast radius, and is highly measurable — deciding factor: reversibility (the cap keeps blast radius low even if wrong).
8a. Post an update (single project) is HITL because it's hard to reverse once sent, even though blast radius and measurability are only moderate — deciding factor: reversibility.
8b. Approve a company-wide update sits above the line because it's hard to reverse and has a high blast radius — deciding factor: blast radius.

## Hardest call

*Choose what to escalate.* My gut said above the line (human decides), but the scores said all-Med/HITL. Blast radius settled it: a wrong call here could look fine in the moment and only turn into damage once it's too late to catch.
