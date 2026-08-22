# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo
>
> ✅ **What this validates:** the agent actually runs end to end — by the end you'll have proven it with real screenshots of your Cortex across the six required moments (M2 to M6).

## What it does

_One paragraph: the agent in action, end to end._

## How you built it

- **Coding agent:** _which one you directed (Claude Code / Cursor / Codex)_
- **Model + bounds:** _model used, max iterations, cost cap, queue cap_
- **Repo / config:** _path to your build in `00-build/`_
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | _[img]_ | happy-path run: a real drafted update + the HITL checkpoint (queued, not posted) | M2 |
| 2 | [transcript below](#m3-critic-rejection-evidence) | the critic rejecting a bad draft (invalid "Green" status claim despite an open issue), revise → escalate | M3 |
| 3 | _[img]_ | a grounded update citing pulled activity + a caught hallucination | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

### M3 critic-rejection evidence

Captured from a live `python agent.py` run (task-happy fixture). Cortex drafted a status
update claiming **Green** despite an open issue (#818), the independent critic rejected it,
Cortex revised, and, still unable to justify "Green" cleanly, escalated to a human on its own:

```
[step 3] PROPOSED OUTPUT:
### Weekly Leadership Status Update for Northstar (P-NORTH)

#### Project Status: **Green**
- The Northstar project is on track.
...
### Risks/Blockers
- Current status is green, and there are no Sev-1 open issues impacting the project.
...
DONE: Update queued for your review; next sprint stories queued for approval.

================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "fail",
  "reasons": [
    "Output incorrectly states the project status as 'Green' while referencing open issue #818 which needs review but is not necessarily impacting the project status. Red/Yellow/Green calls must be evidence-based and this should have been escalated to review the potential impact of the open issue."
  ]
}

-> critic rejected; revision 1/2

[step 6] PROPOSED OUTPUT:
The project status for Northstar (P-NORTH) should be stated as "on track" rather than
"Green," given the presence of an open issue needing review. As such, I will escalate
this for further assessment on how the open issue may affect the overall project status.

**Escalate:** Further assessment is needed to determine the project status in light of
the open issue requiring review.

================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "pass",
  "reasons": []
}

================================================================
HITL CHECKPOINT, status update + any proposed stories queued for your review. Nothing
posted, no commitments made. Run cost ≈ $0.0033
================================================================
```

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._
