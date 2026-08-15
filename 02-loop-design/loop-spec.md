# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop — by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Hook (primary) + Cron (backup)

**Why:** Hook fires in direct response to the event, someone asking for the update. A Friday 9am cron backup sweep guards against a week going by where nobody explicitly asks, so it doesn't silently get skipped.

**Ruled out:** Heartbeat, doesn't need constant checking, there's no reason to poll. Goal, not needed for this task; it's not an open-ended "keep working until X" problem.

**Dedupe:** Cortex dedupes by message ID before drafting, so a retried or duplicate inbound message doesn't trigger a second draft.

## 2. Goal / definition of done

A status update grounded in real pulled activity, critic-validated (or escalated if it can't pass), queued for human review. Cortex never posts it itself.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | Critic returns `"verdict": "pass"` on the draft | Draft is held for human review; loop ends cleanly |
| **Stuck / give up** | Revision cap hit — critic rejects the same draft twice (2/2) | Cortex stops looping, logs the failure, escalates instead of trying a third time |
| **Escalate to human** | Ties to M1 HITL checkpoints #8a (posting anything) and #6 (choosing what to escalate); also fires if a CONFIDENTIAL roadmap item would otherwise leak into a company-wide update | Draft held, human notified, nothing posted |

## 4. State

The roadmap, team norms, and past-update history persist across runs as ground truth Cortex checks itself against, plus a log of already-processed message IDs (for dedupe). Scoped per-project — no cross-project confidential leakage.

## 5. The five things a loop can lean on

_`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP) — otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer._

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed yet, because Cortex doesn't touch code or need an isolated workspace per run, it only reads data and drafts text. |
| **Skills** (reusable capabilities) | Not needed yet, because there's only one task type (status update), no reusable capability library to build out yet. |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | Not wired yet, current tools (`get_project`, `get_activity`, etc.) run against local mock fixtures. Plan: connect to Jira + GitHub via MCP. |
| **Subagents** (independent check when the loop can't grade itself) | _placeholder → M3 orchestration-map.md_ |
| **State tracking** | See §4 State above, roadmap/norms/past-updates + processed message IDs. |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules — you'll add them to their own deliverables then, not here.

## Link to live loop

_[path to your agent in `00-build/`]_
