# Architecture

This document describes how the Open Labor Foundation repositories fit
together — the structural wiring of the stack.

- **The target** — what the stack is building toward, and the invariants any
  correct version satisfies — lives in `VISION.md`. This document does not
  restate it.
- **Status** — what is built, in progress, broken, or planned — lives in issues
  and the project board. This document contains none of it.
- **Describing a mechanism here does not assert it exists.** This is the designed
  structure; whether any given piece is built and working is an issue, not a line
  in this file. If the wiring itself changes, update the description — never
  record progress against it.

## The core loop

Three repositories form the product; everything else builds toward, supports, or
maintains them.

### labor-commons — the catalog

Specialist definitions, one per profession, with no runtime code. Laid out along
two axes:

- `catalog/naics-overlays/{industry}/{specialist-slug}/spec.yaml` — what a
  company operating *in* an industry needs.
- `catalog/function-overlays/{domain}/{specialist-slug}/spec.yaml` — what any
  organization's internal functions need (finance, HR, legal-and-compliance,
  marketing, sales-and-revenue, operations, product, and others), independent of
  the organization's own industry.

A fast pre-scoping index (`data/catalogs/agent_catalog.sqlite`) is queried to
narrow candidates before full `spec.yaml` files load. Consumers scan both axes:
foundation-site's catalog-count script, commons-crew's catalog scanner, and
commons-board's labor-commons readers.

Practitioner corrections flow back through one shape: a correction (which spec,
which field, proposed value, rationale) opens a PR against labor-commons from an
isolated worktree and never self-merges — it goes through the same review as any
other catalog change. Two entry points use this shape: commons-board's org-roster
UI (a human editing a field) and commons-crew's `propose_spec_correction` (a task
proposing one mid-conversation, propose-only, gated by its own narrow allowlist,
with a human approving execution separately).

### commons-board — governed authority

The governance wrapper over a collection of commons-crew instances. Its concerns
are the authority chain (board → chairs → directors → workers), the per-org
autonomy tier, the immutable audit trail, and human oversight — not specialist
resolution.

- A chair is the top-level commons-crew instance in the collection, registered as
  a real commons-crew run at onboarding (`pa.createChairRun` via
  `POST /api/chairs`), giving it an audit trail, autonomy tier, and
  `delegate_to_child` capability.
- commons-board runs its own axis-aware labor-commons search to decide *which
  specialist to preview and pin* for a chair at onboarding — a human-reviewable
  step, distinct from governed execution.
- A board request reaches a chair's run through a two-call split:
  `POST /api/v1/board/requests/:id/dispatch-to-commons-crew` proposes the
  delegation (no real-world effect), and a separate operator-gated
  `.../dispatch-to-commons-crew/decision` approves and executes it. The decision
  input has no default, so nothing auto-approves a real-world action.
- Approvals are attributed to a real bridged identity per admin
  (`ensureBoardMemberIdentity`), namespaced by org, not to a placeholder.
- `chair-reasoning.ts` is the advisory/conversational surface (talk to a chair,
  get an answer now); commons-crew dispatch is the governed-execution surface
  (the chair's org actually does the work, audited). A request may use both —
  they are complementary, not alternatives.

### commons-crew — recursive delegation

The single mechanism at every layer, including the chair. Each hop is scoped one
level down and assembles its own specialist from labor-commons.

- `delegate_to_child` is a governed action tool, running through the same
  propose → approve → execute loop as any side-effecting action; single- and
  multi-hop chains (chair → director → department → worker) delegate this way.
- `computeDelegationRequiresApproval` gates each hop by the org's autonomy tier:
  `advisor` gates every hop; `orchestrator` auto-approves every hop except the
  final one into `worker`; `autopilot` auto-approves all hops, capped by a hard
  numeric backstop independent of tier. The tier is synced down from
  commons-board (`pa.setOrgAutonomyTier`), never decided by commons-crew; an
  unsynced org fails closed to `advisor`.
- `search_artifacts` is a read-only governed tool that searches artifact-commons
  before build capability is used, invocable by a task through the autonomous
  tool-selection loop across execution paths (`shared_runner`,
  `isolated_subprocess`).

### artifact-commons — the second commons

A peer catalog holding previously built solutions (`board-addins/`), governed by
the same reuse principle one layer up. commons-board reads its `catalog.json`
through its addin registry; commons-crew reaches it through `search_artifacts`.
The sequence when a gap is found: check for a match, adapt a close one, build from
scratch only if nothing fits — then publish back.

## Repository classification

### Core (the product itself)
- **labor-commons** — the catalog
- **commons-board** — governed authority
- **commons-crew** — recursive delegation
- **artifact-commons** — reusable solutions

### Relics (early bootstrapping, superseded once the core works as intended)
- **commons-idea** — intake via a hand-pasted LLM prompt; superseded once
  commons-crew provides native in-product intake.
- **commons-specs** — community archive of written idea records; superseded by the
  same.
- **commons-artifacts** — the original deliverable/addin-sharing repo; retained
  only for its narrower original use case (sharing a standalone generated
  deliverable), its addin-economy role having moved to artifact-commons.

### Maintenance (keep the product honest over time; no end user touches these)
- **commons-keeper** — independent catalog health and cross-repo security review.
  External and independent by design.
- **labor-commons-curator** — the specialist contract and an independently graded
  certification pipeline. Independent for the same reason.
- **commons-devloop** — self-hosted autonomous development engine (issue in,
  reviewed PR out). Transitional: its job is build capability, which is what
  commons-board is meant to grow internally via commons-crew's recursive
  delegation. Slated to be retired or absorbed once that capacity exists, not
  maintained indefinitely.

### Delivery (how the product reaches people; not the product)
- **foundation-site** — the public website; presence, not integration.
- **open-labor-foundation** — this repository; the org-level home for `VISION.md`,
  `ARCHITECTURE.md`, and `GOVERNANCE.md`. Other repos draw from it rather than
  each carrying their own account.

## Governance wiring

The two governance questions (defined in `VISION.md`, modeled in `GOVERNANCE.md`)
are realized by distinct mechanisms:

- **Per-organization authority** — commons-board's autonomy tiers, synced down
  into commons-crew's delegation gate.
- **Platform-level authority (certification)** — labor-commons-curator's
  certification gate plus commons-keeper's scoring loop, run independently of the
  catalog they check, invoked by labor-commons CI on records marked for
  publication. artifact-commons requires its own equivalent gate before an
  artifact is surfaced as a trusted match rather than merely discovered;
  `GOVERNANCE.md`'s model extends to it directly.

## Cross-cutting structure

- **artifact-commons matching is a commons-crew tool, not a separate service.**
  Before build capability is used, commons-crew's tool loop calls a search against
  artifact-commons through the same propose/approve/execute loop — one more tool
  type, no separate matching engine.
- **Adapting an artifact is not distinct from building one.** Both run through the
  same commons-crew tool loop (`edit_file`, `write_file`, …); the only difference
  is whether the loop starts from an existing artifact's content or a blank slate.
  An exact match needs no execution; a close match seeds the loop; no match runs
  it from nothing.
