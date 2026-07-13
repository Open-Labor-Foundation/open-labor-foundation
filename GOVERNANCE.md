# Governance

This document is the ratified answer to a question `ARCHITECTURE.md` names but doesn't settle: what does it take for a determination — a specialist's advice, a chair's decision, an artifact's fitness for reuse — to be trusted as binding, and by whom? It splits into two distinct questions that were previously conflated as one.

## Per-organization authority

Already answered by a mechanism that exists today: commons-board's autonomy tiers.

- **Advisor** — every determination is advisory until a human approves it.
- **Orchestrator** and **Autopilot** — progressively more is presumptively binding within a pre-approved scope, with anything outside that scope, or above a defined risk threshold, still escalating to a human.

Autonomy never self-promotes. A deployment's tier — and therefore the weight its determinations carry within that org — is set by that organization's own governance process: owner approval in business mode, member vote in collective mode. New deployments start in Advisor. Nothing new needs to be built for this; this section states it as policy, plainly, rather than leaving it implicit in the autonomy-tier code.

## Platform-level authority: certification

Per-org authority answers what's binding *inside one deployment*. It doesn't answer what makes something trustworthy enough to be treated as binding *across the whole commons* — surfaced as a match rather than merely discovered, reused without every consumer re-verifying it from scratch. That's certification, and until now the mechanism existed without a ratified statement of who runs it, what happens on disagreement, or how it gets revoked.

### Who certifies

Certification must run independent of the thing it certifies — the same reasoning [commons-keeper](https://github.com/Open-Labor-Foundation/commons-keeper) and [labor-commons-curator](https://github.com/Open-Labor-Foundation/labor-commons-curator) already state for their own scope, restated here as platform policy rather than a per-repo design note. The target division of responsibility, once both sides are fully built, for labor-commons:

- **labor-commons-curator** defines the certification gate itself: test scenarios derived from a specialist record's own stated claims. A record cannot reach `published` status without passing them, and a record that predates the gate is certified retroactively (`backfill_sweep`) rather than grandfathered in.
- **commons-keeper** runs that gate on a schedule, scores the catalog against it, detects regressions against stored history, and — this is the part that matters for standing — never applies a disputed change to the catalog directly. Safe, deterministic refinements land automatically; anything else surfaces as a PR into `autonomous/review` for a human with commit rights on labor-commons to accept or reject.

**Current implementation status (2026-07-13):** neither side is built yet. labor-commons-curator has one merged PR — project scaffolding with a placeholder health-check function, no test-scenario derivation and no `published`-gate logic. commons-keeper's scoring loop runs on schedule as described, but scores against a file set (`evaluation/scenarios.md`, `readiness/evidence.json`, and others) that the catalog's schema migration removed — no specialist record has these files anymore, so every record currently scores far below the configured thresholds regardless of actual quality. Fixing that scoring loop and building labor-commons-curator's actual gate (tracked as labor-commons-curator issues #4–#15) are open work, not yet done. This section states the model contributors should build toward; it is not yet a description of what runs today.

No specialist record certifies itself, and no automated pass has final say — it has *first* say, gated by an independent process, with a human as the actual authority of record.

artifact-commons will need the equivalent pairing once it exists: an independent certification gate (what makes a shared solution trustworthy enough to surface as a match, not just discoverable) and an independent scoring/review loop, structured the same way. This document is written so that extension doesn't require re-deciding the model, only applying it.

### Disagreement

Two kinds of disagreement are possible, and they're handled the same way: surfaced, not silently resolved by whichever system produced the disputed result.

- **Automated certification is wrong** (a record passed that shouldn't have, or failed that should have passed). This is exactly what commons-keeper's `autonomous/review` PR queue exists for — a human reviewer with commit rights on the relevant repo makes the final call, informed by the automated result but not bound by it.
- **A human disputes a published, already-certified result** (in practice: it produced bad advice, or turned out to rest on a stale or incorrect source). This is not a certification-pipeline bug to route through the same queue as an automated disagreement — it's a report against something already trusted, and it starts the revocation path below rather than waiting for the next scheduled sweep.

Either way, the standing to decide sits with a human maintainer of the certifying repo (labor-commons-curator + commons-keeper today), not with the platform that consumed the specialist and hit the problem. commons-board and commons-crew consume certification status; they don't adjudicate it.

### Revocation

This section describes the target state model: a record should move from `certified`/`published` to `flagged` or `retired`. As of this writing, labor-commons' real schema (`infra/schema/specialist-record.schema.json`) only has `published` as an actual status value — `certified`, `flagged`, and `retired` don't exist in the schema yet. Adding them is part of the same open work as the certification gate itself (see status note above), not a separate task. Once they exist, *when* a transition is allowed to happen and what a consumer owes it once it does is settled here so that part of the design doesn't need re-deciding later:

1. **Triggered by:** a failed periodic re-certification sweep, an explicit "flagged stale by use" signal from a consumer, or a human-initiated dispute per the section above. Any of the three is sufficient — none requires the other two to concur first.
2. **Effective immediately, not eventually.** A record moved to `flagged` or `retired` stops being surfaced as a trusted match the moment that state change lands — commons-board and commons-crew read status on every resolution, not from a cache that catches up later. There is no grace period during which a revoked record continues to be treated as certified.
3. **`flagged` is not `retired`.** `flagged` means under review with certification in question; the record isn't deleted and isn't necessarily wrong, but it's no longer presumptively trustworthy until re-certified. `retired` is the terminal state — a consumer that still holds a reference to a retired record must treat it as a gap, the same as if no record had ever existed.

The same model applies to artifact-commons once its certification gate exists: revocation is triggered the same three ways, takes effect immediately, and a retired artifact is a gap to close, not a degraded-but-usable match.

## What this document doesn't do

It doesn't build anything. **Correction (2026-07-13):** an earlier version of this document claimed the mechanisms it ratifies — labor-commons-curator's gate, commons-keeper's review loop, the `flagged`/`retired` states — already existed in code. An audit found that wasn't true: labor-commons-curator's gate isn't built, commons-keeper's review loop runs but scores against files the catalog no longer has, and `flagged`/`retired` aren't in the real schema. This document ratifies the model those mechanisms should follow once they're built; treat the "Current implementation status" note under "Who certifies" as the actual state, not this section. It doesn't cover artifact-commons' own certification gate in detail, since that repo doesn't have one yet; when it's built, its design should extend this document's model rather than write a new one. And it doesn't change per-org autonomy tiers, which were already policy in practice — this is the first place that's written down rather than left implicit.
