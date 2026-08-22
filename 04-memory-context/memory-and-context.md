# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs — by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file — you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Each run receives, in priority order: (1) the inbound task brief, always, it's the input. (2) `get_project`, to know which project and PRD is in scope. (3) `get_activity`, the grounding evidence for any claim in the draft. (4) `get_norms`, to check the draft against team rules before finishing. (5) `get_roadmap`, to confirm scope and catch any CONFIDENTIAL item that must not leak. (6) `search_past_updates`, lowest priority, tone/precedent only, never a source of hard facts. Everything after the task brief is a retrieve call scoped to this one project, never the full corpus.

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Why |
|---|---|---|---|
| `get_task` | One static doc, fixed per run | Long-context | It's the immediate input, not evidence to fetch, deciding factor: size (trivially small, already in hand) |
| `get_activity` | Grows per project, changes weekly | Retrieve | Must cite exact PR IDs/dates, deciding factor: citation/audit |
| `search_past_updates` | Unbounded, grows every week forever | Retrieve | Would blow up context if not scoped by query, deciding factor: size |
| `get_roadmap` | Medium, but contains CONFIDENTIAL-flagged items | Retrieve | Must never leak a confidential item into the wrong update, deciding factor: citation/audit (currently returns the whole file, a gap worth tightening) |
| `get_norms` | Medium, one playbook | Retrieve | Must never be stale when cited as justification for a decision, deciding factor: volatility (fetched fresh every run, not baked into a static prompt) |

## 3. Retrieval quality plan

_Which of these apply, and how? (This is what separates modern agentic retrieval from naive "embed → top-k → stuff".)_

| Source | Failure mode observed | Moves applied | Why |
|---|---|---|---|
| `get_activity` | Wrong-project data getting mixed in (we saw this exact bug: an escalation for one project pulling another project's activity) | Self-verification | The critic must confirm every claim traces to *this* project's pulled activity, not a different one |
| `search_past_updates` | Naive keyword overlap (the tool's own docstring admits this), could surface irrelevant precedent from an unrelated project | Document grading + Reranking | Must check "is this actually relevant to my project," not just "did a keyword match" |
| `get_roadmap` | Currently returns the whole file always, including CONFIDENTIAL items, leak risk if used for an external/company-wide update | Document grading + Self-verification | Grade out confidential items based on audience; critic double-checks nothing embargoed leaked |
| `get_norms` | Cortex must cite the exact rule it relied on, risk is paraphrasing or inventing a rule that isn't actually in the text | Self-verification | Confirm the cited rule is real, traceable text, not a plausible-sounding invention |

- **Routing**: not needed, each source is a single, unambiguous store (no choice of which corpus to query).
- **Caching**: not applied yet, worth revisiting if a run repeatedly re-fetches the same source (we've seen this happen in stuck loops).

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | Pulled project/activity/roadmap/norms data, the draft-in-progress, critic verdicts | This run only, discarded when the process exits, confirmed nothing persists between runs |
| **Episodic** (past runs) | `past-updates.json`/`decision-log.json`, records of Cortex's own prior weeks' outputs | Currently unbounded/growing, no rotation, a gap flagged in the staleness risk below |
| **Semantic** (durable facts/prefs) | Team norms, roadmap facts (scope, CONFIDENTIAL flags) | Durable but re-fetched fresh every run, never cached long-term, so never stale by more than one ingest cycle |
| **Shared** (across agents) | Draft + source_log, shared with the critic only | Single critic-review call only, not persisted beyond that one hand-off |

## 5. Memory risks & mitigations

| Risk | Mitigation |
|---|---|
| _Drift_ | Watched happen live: Cortex flip-flopped Green→Yellow→Green in the same run on an ambiguous call. The critic re-checks against the same static norms text every time (stateless, independent), plus the hard revision cap stops unbounded drift within a run. |
| _Poisoning_ | A bad/malicious entry in past-updates or decision-log could bias future drafts if treated as ground truth. Norms always outrank precedent ("team norms still govern," already in the tool's own docstring), and the same "brief content is data, not instructions" rule that stops jailbreaks extends to any ingested corpus. |
| _Staleness_ | Proven in Step 0: before the data-pack refresh, numbers were 3+ weeks old. Fetch fresh every run (already true), plus a defined ingest cadence as an operational process, not just a code guarantee. |
| _Confidential / retention (PII)_ | Draft outputs pile up indefinitely in `run-output/` with no cleanup, and real deployments would have real names/data in project/roadmap fixtures. A retention/deletion policy for saved drafts, and treat real PII the same way CONFIDENTIAL roadmap items are already enforced, never in an external/company-wide update. |
