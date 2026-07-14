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

**Current implementation status (2026-07-14):** labor-commons-curator's certification gate is now real, merged, and independently verified — not scaffolding. `certifyForPublish` runs the full three independent passes (scenario-generation, then manifest-conversion + materialization + execution/grading) against a real materialized specialist, driving commons-crew's actual manifest parser and `materials.create` rather than a mock of either (verified end to end against a real vendored commons-crew checkout, including the real generated system prompt). `sweepBackfillCertification` retroactively certifies pre-gate records the same way, skipping records already certified through the real gate and reporting — not silently swallowing — any record whose pipeline errors. All thirteen tracked issues (#3–#15) are closed.

commons-keeper's scoring loop is also fixed: it now checks the real single-file `spec.yaml` content (`authority_sources`, `adjacent_specialties`, `specialty_boundary` length, `metadata.status`) instead of the six-file evaluation/readiness paths the catalog's schema migration removed, which had put every record's score at or near zero regardless of actual quality. Verified live against the real catalog: scoring now surfaces specific, real issues instead of a uniform automatic failure.

**What's still open, precisely:** these two fixes are not yet wired to each other or to a real publish flow.

- Nothing calls `certifyForPublish` from an actual publish action yet — a record's `metadata.status` does not currently get blocked from reaching `published` by a failed certification. Issue #14 scoped that integration out explicitly ("separate, later work not in this train"); it remains genuinely separate, later work.
- Nothing schedules `sweepBackfillCertification` against the real catalog or writes its results back to any file — issue #15 scoped it as report-only by design. A real backfill run, and deciding what happens to a `newly_failing` record's status, is still undecided/unbuilt.
- commons-keeper's (now-fixed) scoring loop and labor-commons-curator's certification gate are two separate mechanisms, not one — commons-keeper does not call labor-commons-curator's functions. The "commons-keeper runs that gate on a schedule" framing below is still the target model, not the current wiring.

No specialist record certifies itself, and no automated pass has final say — it has *first* say, gated by an independent process, with a human as the actual authority of record.

artifact-commons will need the equivalent pairing once it exists: an independent certification gate (what makes a shared solution trustworthy enough to surface as a match, not just discoverable) and an independent scoring/review loop, structured the same way. This document is written so that extension doesn't require re-deciding the model, only applying it.

### Disagreement

Two kinds of disagreement are possible, and they're handled the same way: surfaced, not silently resolved by whichever system produced the disputed result.

- **Automated certification is wrong** (a record passed that shouldn't have, or failed that should have passed). This is exactly what commons-keeper's `autonomous/review` PR queue exists for — a human reviewer with commit rights on the relevant repo makes the final call, informed by the automated result but not bound by it.
- **A human disputes a published, already-certified result** (in practice: it produced bad advice, or turned out to rest on a stale or incorrect source). This is not a certification-pipeline bug to route through the same queue as an automated disagreement — it's a report against something already trusted, and it starts the revocation path below rather than waiting for the next scheduled sweep.

Either way, the standing to decide sits with a human maintainer of the certifying repo (labor-commons-curator + commons-keeper today), not with the platform that consumed the specialist and hit the problem. commons-board and commons-crew consume certification status; they don't adjudicate it.

### Revocation

This section describes the target state model: a record should move from `certified`/`published` to `flagged` or `retired`. **Correction (2026-07-13):** an earlier version of this paragraph cited a specific schema file and status enum that don't exist — there is no `infra/schema/specialist-record.schema.json` in labor-commons, and no schema anywhere enforces a status enum at all. In practice `spec.yaml` files carry a range of `metadata.status` values today (`validated`, `published`, `draft` among them), unconstrained by any validator. `certified`, `flagged`, and `retired` don't exist as recognized values in that field, formally or informally. **Update (2026-07-14):** `infra/schema/specialist-record.schema.json` now exists (merged to labor-commons's main) and does define this enum — `metadata.status: "draft" | "validated" | "published" | "flagged" | "stale" | "retired"`. That closes half the original gap: the target vocabulary is now real and canonical, not just this document's proposal. It's still not *enforced*, though — `infra/scripts/validate-spec-yaml.mjs`, the script that actually gates catalog PRs, does not check `metadata.status` against the schema at all; a record with an out-of-enum status value would pass validation today. Enforcing it is part of the same open work as wiring the certification gate to a real publish flow (see status note above), not a separate task. Once enforcement exists, *when* a transition is allowed to happen and what a consumer owes it once it does is settled here so that part of the design doesn't need re-deciding later:

1. **Triggered by:** a failed periodic re-certification sweep, an explicit "flagged stale by use" signal from a consumer, or a human-initiated dispute per the section above. Any of the three is sufficient — none requires the other two to concur first.
2. **Effective immediately, not eventually.** A record moved to `flagged` or `retired` stops being surfaced as a trusted match the moment that state change lands — commons-board and commons-crew read status on every resolution, not from a cache that catches up later. There is no grace period during which a revoked record continues to be treated as certified.
3. **`flagged` is not `retired`.** `flagged` means under review with certification in question; the record isn't deleted and isn't necessarily wrong, but it's no longer presumptively trustworthy until re-certified. `retired` is the terminal state — a consumer that still holds a reference to a retired record must treat it as a gap, the same as if no record had ever existed.

The same model applies to artifact-commons once its certification gate exists: revocation is triggered the same three ways, takes effect immediately, and a retired artifact is a gap to close, not a degraded-but-usable match.

## What this document doesn't do

It doesn't build anything. **Correction (2026-07-13):** an earlier version of this document claimed the mechanisms it ratifies — labor-commons-curator's gate, commons-keeper's review loop, the `flagged`/`retired` states — already existed in code. An audit found that wasn't true: labor-commons-curator's gate isn't built, commons-keeper's review loop runs but scores against files the catalog no longer has, and `flagged`/`retired` aren't in the real schema. **Update (2026-07-14):** labor-commons-curator's gate and commons-keeper's scoring loop are now both real and independently verified (see "Current implementation status" above), and `flagged`/`retired` are now real enum values in the canonical schema, though not yet enforced by the validator. What's still true of this correction: those two fixed mechanisms aren't wired to each other or to a real publish flow yet, so this document still describes the target model more than it describes a fully assembled, running pipeline — treat the "Current implementation status" note under "Who certifies" as the actual state, not this section. It doesn't cover artifact-commons' own certification gate in detail, since that repo doesn't have one yet; when it's built, its design should extend this document's model rather than write a new one. And it doesn't change per-org autonomy tiers, which were already policy in practice — this is the first place that's written down rather than left implicit.
