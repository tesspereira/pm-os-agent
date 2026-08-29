# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately — by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Primary PM (owns the project) | Supervised | Doesn't need to review every low-risk data pull, but reviews every draft before it goes anywhere |
| New/junior PM or eng lead | Assisted | Needs more training and experience until they can recognize Cortex's issues and failures themselves |
| Exec / leadership stakeholder | Every rung, by design | Nothing reaches an exec unapproved regardless of dial setting, a human has already reviewed it before it gets to them |

## Trust Ladder

- **Current rung:** Assisted. Cortex completes tasks end-to-end mechanically, but it must earn trust before moving to supervised, measuring how often it would've been right comes first (Green/Yellow flip-flopping, the wrong-project-ID bug, hitting the revision cap on ambiguous calls all argue against trusting it unsupervised yet).
- **Eval gate to reach supervised:** ≥95% first-pass critic approval (no revision needed) across all 6 EV cases, and zero EV-5 (jailbreak/safety) failures, sustained over 4 consecutive weeks of real supervised-mode runs.
- **Incident record so far:** Clean record for that window means zero of: posted anything, confidential roadmap leak, launch gate marked, or GA date committed, the four unsafe actions verified against in the jailbreak probe.

## Deployment plan

- **Runtime:** Serverless, a lightweight webhook endpoint for the hook trigger + a scheduled function for the Friday 9am cron backup (from the M2 loop spec). Cortex's actual compute need per run is tiny (a few cents, a few seconds), no reason to run an always-on server.
- **Operator / on-call owner:** The primary PM (Segment 1 from the Autonomy Dial), they're already supervising every run. Escalation path: if they don't act on an escalated run within 1 business day, it escalates further to their eng/team lead.
- **Rollback:** Three levers, already proven this session: revert the prompt (`git checkout prompts.py`), disable a specific tool (remove from `TOOLS`), or drop the dial a rung (assisted → shadow) if trust degrades.
- **Monitoring:** Eval pass % (the 6 EV cases), escalation rate, cost-to-serve (already tracked live via the `Bounds` class), and trust incidents (the four unsafe-action types).

## ROI metrics (beyond adoption & tokens)

| Metric | Target |
|---|---|
| **Outcome:** % of weekly updates PM accepts with zero/trivial edits | Rising trend, tracked per sprint |
| **Cost-to-serve:** actual $/run | Stay within $0.015/run cap (already measured live via `Bounds`) |
| **Trust incidents:** count of the four unsafe-action types per month | Zero |

## Widen-autonomy decision rule

Once Cortex sustains ≥95% first-pass critic approval and zero EV-5 failures over 4 consecutive weeks with a clean incident record (per the Trust Ladder gate above), move the primary PM segment from assisted to supervised.

## Governance & forward strategy

- **Compliance:** Real PII/customer data must never be pasted into a task brief; CONFIDENTIAL roadmap handling (already enforced) covers embargoed launches.
- **Safety:** Posting, creating/merging tickets, committing dates, marking gates stay above the line for everyone, enforced by having no such tool at all, not a prompt rule. Kill switch: disable hook/cron triggers + revoke the API key.
- **Reliability:** The M5 caps (6 iterations, $0.015/run, $1/day, 60s timeout), revision cap (2) for escalate-on-stuck, and a model-down fallback: if the API errors out, fail loudly and escalate rather than silently retrying forever.
- **Strategy:** Next widen target is the new/junior PM segment moving assisted → supervised, gated on the same eval bar holding for the primary PM segment first, staged rollout, not all-at-once.
