# Architecture

This document describes what the Open Labor Foundation stack is actually building toward, and where each repository fits. It is the canonical reference — other repos should draw from this rather than each maintaining their own account of the ecosystem.

## Mission

The founding question behind this project: if AI can genuinely replace or augment labor at the level of a professional role, that claim should be proven, not asserted — and it should be proven in a controlled, governed environment, not a chaotic one.

The proof starts with a single, comprehensive catalog of what every profession actually does, and builds upward from there into a governed system that can put that catalog to work — from the top of an organization's decision-making structure all the way down to the individual worker-level task, without collapsing into either an unaccountable autonomous agent or a system too shallow to matter.

The end goal is to provide this system as open source, removing the operating burden — software costs, staffing overhead, administrative load — for a business owner in any industry, without requiring them to understand or manage the AI underneath it.

## The core loop

Three repositories form the actual product. Everything else exists to build toward them, support them, or maintain them over time.

### labor-commons — the catalog

A single catalog of specialist definitions, one per profession, seeded from the NAICS classification of essentially every industry and role in the US economy. Each definition specifies what an agent standing in for that profession is authorized to decide, what it must escalate or refuse, and the boundary that keeps it faithfully imitating the role rather than improvising past it.

The reuse principle is central: a definition isn't scoped to whichever business or workflow needed it first. Any part of the platform can invoke any specialist whose boundary fits, regardless of what originally motivated writing it.

labor-commons is deliberately just a catalog — no runtime code. Generation, maintenance, and certification are each pulled into their own repositories (see Maintenance, below), keeping the catalog itself thin and authoritative.

**The catalog has two axes, not one.** `catalog/naics-overlays/{industry}/{specialist-slug}/spec.yaml` answers "what does a company operating *in* this industry need" — the axis that exists today. It has no answer for the orthogonal question, "what does *any* organization's internal Finance, HR, Legal, Marketing, Sales, or Product function need, regardless of what industry the organization itself is in" — a generic corporate-function axis, not an industry-vertical one. That gap is why commons-board's chair-matching produces specialists a domain expert wouldn't have picked (see commons-board, below): the catalog was never searched over the right axis, because the right axis didn't exist.

**Migrated.** The second axis now exists: `catalog/function-overlays/{domain}/{specialist-slug}/spec.yaml`, 260 specialist definitions across 22 functional domains (finance, HR, legal-and-compliance, marketing, sales-and-revenue, operations, product, and others), converted from mother-board's legacy `manifest.yaml` shape to the current `spec.yaml` schema, domain folder names stripped of the source's `director-of-` prefix to match how industry folders are named. The three downstream consumers that hardcoded `naics-overlays` specifically have all been widened to scan both axes: foundation-site's live catalog-count script, commons-crew's `LocalCatalogService` catalog scanner, and commons-board's labor-commons readers (`labor-commons-client.ts`, `specialist-resolver.ts`). `data/catalogs/agent_catalog.sqlite` — the fast pre-scoping index commons-board's readers query before loading full spec.yaml — is reconciled to match.

**Current gap:** the catalog currently grows one way — AI inference against the NAICS backlog. The other intended growth path, corrections and additions from actual practitioners in a given profession, has no working mechanism yet. commons-board and commons-crew are expected to provide user-guided spec updates; that isn't built.

### commons-board — governed authority

The proof that AI-run governance can work at the top of an organization. Structurally, commons-board is a collection of commons-crew instances — chairs, and everything beneath them — wrapped in governance: a chain of authority rather than a flat pool of agents, autonomy that only ever increases through explicit human promotion, never self-promotion, and an immutable audit trail before anything executes.

Chairs are not a separate concept commons-board staffs on its own. A chair is simply the top-level commons-crew instance in that collection, governed and audited the same way every layer beneath it will be. commons-board's own job is the governance wrapper — authority chain, autonomy tier, audit trail, human oversight — not specialist resolution.

**Both original gaps are fixed.** The missing-axis problem (commons-board's resolver could only search `naics-overlays`) is resolved by the two-axis catalog migration described under labor-commons, above. The duplication problem — commons-board resolving specialists on its own instead of going through commons-crew — is resolved at the identity level: every chair is now registered as a real commons-crew run at onboarding (`pa.createChairRun`, called via commons-crew's `POST /api/chairs`), giving it an audit trail, autonomy tiers, and `delegate_to_child` capability from the moment it's created.

This is deliberately not a full replacement. commons-board still resolves *which specialist to preview and pin* for a chair using its own axis-aware labor-commons search — a legitimate, human-reviewable onboarding behavior, independent of the governance question.

**A chair's registered run can now actually be used, via a raw API call — not yet through the product.** `POST /api/v1/board/requests/:id/dispatch-to-commons-crew` proposes a `delegate_to_child` dispatch of a board request to its target chair's commons-crew run — safe to call automatically, since proposing has no real-world effect. A **separate**, explicitly admin/operator-gated `POST .../dispatch-to-commons-crew/decision` is the only thing that can approve and execute it; `decision` is a required input with no default, so nothing auto-approves a real-world-impact action on a human's behalf. This split exists because an earlier draft of this feature auto-approved the delegation as part of the same call — a real governance-model violation, caught before merging, not after. Verified end to end against real running servers: propose → explicit approve → real delegated child run, and propose → explicit deny → no execution. **Correction:** an earlier version of this section implied "admin/operator has to trigger it explicitly" meant a UI action. It doesn't — an audit found zero references to this route anywhere in commons-board's actual UI (`apps/web`). Today this is reachable only by someone calling the HTTP API directly; no person using the shipped product can trigger it.

**An actor-identity bridge is built and in review, not yet merged (commons-board#5).** commons-crew's approval endpoint checks workspace membership, and today only recognizes its own seeded identity (`user_primary`) — every commons-board decision is still attributed to that placeholder regardless of who actually decided, until this PR merges. `ensureBoardMemberIdentity` closes this using commons-crew's own existing user/membership system (`POST /api/users`, `POST /api/workspaces/:id/memberships` with an `approval_decision` permission) — no new commons-crew capability needed, just a commons-board-side bridge: a real commons-crew user + "supporting" membership per commons-board admin, namespaced by org so two orgs' same user id can't collide, created once and reused. Falls back to `user_primary` only if the bridge itself can't run. Live-verified on the PR branch: the bridged identity actually deciding a real approval, not just existing as a record.

Two things still genuinely open: first, the dispatch mechanism above has no UI path yet, only a raw API — see the correction above. Second, nothing in commons-board's normal request lifecycle calls the dispatch mechanism automatically either way — an admin/operator would have to trigger it explicitly per request even with a UI, and reconciling it with the existing direct-LLM `chair-reasoning.ts` path (which this doesn't touch) is a deliberate, separate product decision, not made here. Until both are addressed, the hundreds of line-level specialists the catalog contains are technically reachable through a real, verified mechanism, but not exercised — or even reachable — in day-to-day use of the actual product.

### commons-crew — recursive delegation

The single mechanism used at every layer of an organization — including the chair itself. A chair is a commons-crew instance; it hands a request to a director, itself another commons-crew instance; the director hands it to a department, the department to a worker — each hop scoped one level down, independently assembling whatever specialist its task needs from labor-commons. This is also what makes the full breadth of labor-commons load-bearing rather than dormant: instead of only a chair-level slice of the catalog ever being reached, every layer, including the chair layer itself, resolves through the same recursive mechanism. commons-board is what you get when this collection of instances is wrapped in governance.

**Built and merged to commons-crew's `main`.** `delegate_to_child` is a real action tool, going through the same governed propose/approve/execute loop as any other side-effecting action. Single-hop and multi-hop delegation both work end to end (chair → director → department → worker), verified by real tests against the actual public API, not mocked. `pa.createChairRun` registers a root run as a specific chair — a fixed v1 role set, expanded from six to eight to match commons-board's real onboarding domains (finance, legal, HR, marketing, operations, product, IT, security) — for a specific org, pre-authorized to delegate immediately, with that org context propagating down everything it spawns. Reachable externally via `POST /api/chairs`. `LocalCatalogService` scans both catalog axes. Full detail in commons-crew's own `docs/architecture.md`.

**Current gap:** commons-board now calls `pa.createChairRun` once per chair at onboarding, and can dispatch real work to that run via a proposal it explicitly approves (see commons-board, above — identity bridging for that approval is built and in review, not yet merged; the dispatch route itself has no UI yet either). One primitive this uncovered and fixed: a chair's seeded delegation approval is one-shot by design (an approval authorizes one specific act, not a standing blank check), so a long-lived chair had no way to delegate a *second* time — `pa.requestDelegationApproval` (`POST /api/runs/:runId/delegation-approvals`) seeds a fresh one on request, verified live across multiple sequential dispatches to the same chair. `search_artifacts` (`class_a`, read-only, no approval needed) is the commons-crew side of artifact-commons matching, built and verified live against the real artifact-commons repo on its own PR branch (commons-crew#11, not yet merged). A real autonomous tool-selection loop that lets a task call `search_artifacts` on its own initiative — the actual missing piece flagged below — is also built and in review (commons-crew#12, not yet merged). What's still open: commons-board's normal request lifecycle doesn't call the dispatch mechanism automatically, and search_artifacts' autonomous invocation hasn't been verified against a real model yet, only a deterministic test double — see commons-crew's own `docs/architecture.md` for the current, precise status of both.

### artifact-commons — the second commons

A peer catalog to labor-commons, holding not professional boundaries but previously built solutions. The same reuse principle applies one layer up: a solution built by one business isn't scoped to that business. Any other operator can use it as-is or adapt it, without needing to know the catalog exists — that responsibility sits with the platform (commons-board / commons-crew), not the user.

The intended sequence when a gap is identified: check artifact-commons for an exact or close match first, adapt an existing artifact if one is close, and only build from scratch if nothing usable exists — then publish the result back so it's never built twice. This is what completes commons-board's "determines but can't close" gap: the actuation arm is check-then-adapt-then-build, with build-from-scratch as the exception, not the default.

**Created, not a repurposing of `commons-artifacts`** (see Relics, below) — the clean separation avoided carrying forward a repo whose schema and history were shaped by an earlier, narrower idea. Holds the real content that had that role: `board-addins/gig-cooperative` and `board-addins/startup-launch`, migrated with a real pre-existing bug fixed in the process (a stale seed-file reference in `gig-cooperative`'s manifest). commons-artifacts' other category — standalone deliverables, unzip-and-run with no board required — was checked directly and never had a single real submission; there was nothing to migrate, and an empty category isn't worth forking across two repos.

Both consumer sides are real and load-bearing: commons-board's `addin-registry.ts` reads this repo's `catalog.json` by default (not `commons-artifacts`' anymore), and commons-crew's `search_artifacts` action tool searches it as a governed, read-only tool — see commons-crew, below. What's still open is the check-then-adapt-then-build *sequencing*: `search_artifacts` exists and works, but nothing calls it automatically before a task reaches for build capability yet, and there's no certification gate (see Governance authority, below) to trust a match before surfacing it.

## Repository classification

### Core (the product itself)

- **labor-commons** — the catalog
- **commons-board** — governed authority
- **commons-crew** — recursive delegation (built and merged; commons-board registers every chair as a real instance at onboarding and can dispatch real work to it via an explicit-approval flow, but nothing in commons-board's normal request lifecycle calls that automatically yet)
- **artifact-commons** — reusable solutions (created; real content, both consumer sides wired up; matching tool isn't invoked automatically yet and there's no certification gate)

### Relics (early bootstrapping, superseded once the core works as intended)

- **commons-idea** — intake via a hand-pasted LLM prompt; superseded once commons-crew provides native, in-product intake
- **commons-specs** — community archive of written idea records; superseded by the same
- **commons-artifacts** — the original deliverable/addin-sharing repo; retained only for its narrower original use case (sharing a standalone generated deliverable), with its addin-economy role moving to artifact-commons

### Maintenance (keep the product honest over time; no end user touches these)

- **commons-keeper** — independent catalog health and cross-repo security review. Deliberately external and independent by design — the same reason labor-commons-curator's certification has to be independent of the thing it certifies.
- **labor-commons-curator** — the specialist "contract" and an independently graded certification pipeline for trusting what's in the catalog. Independent for the same reason as commons-keeper.
- **commons-devloop** — self-hosted autonomous development engine (issue in, reviewed PR out). Actively in use today, but transitional: its job — build capability — is exactly what commons-board is missing internally. Once commons-crew's recursive delegation gives commons-board real build capacity of its own, an external automation engine doing the same job from outside becomes redundant. Expected to be retired or absorbed into artifact-commons/commons-board, not maintained indefinitely as separate infrastructure.

### Delivery (how the product reaches people; not the product)

- **foundation-site** — the public-facing website. Live, deployed, intentionally honest about how much of the stack is actually usable today. Mostly disconnected from the rest of the stack by design — its job is presence, not integration.
- **open-labor-foundation** — this repository. The org-level home for documentation like this one; other repos should draw from it rather than each carrying their own account of the vision.

## Governance authority

This splits into two distinct questions that were previously conflated as one.

**Both questions have a ratified model in [GOVERNANCE.md](GOVERNANCE.md); only one is actually built.** Per-organization authority is already answered by a mechanism that exists today — commons-board's autonomy tiers — and is now stated as policy rather than left implicit in the autonomy-tier code. Platform-level authority — what it takes for something to be trusted enough to be treated as binding across the whole commons, not just within one org's deployment — was the open question; `GOVERNANCE.md` states the target model (who has standing to certify, how disagreement is handled, how a certification gets revoked), grounded in mechanisms that are meant to exist for labor-commons-curator's certification gate and commons-keeper's independent review loop. Neither is actually built yet — see `GOVERNANCE.md`'s own "Current implementation status" note for specifics. artifact-commons will need the equivalent certification gate before an artifact is trusted enough to be surfaced as a match rather than merely discovered — `GOVERNANCE.md`'s model is written to extend to it directly once both the labor-commons and artifact-commons gates exist, not to be re-decided.

## artifact-commons matching

Resolved: matching is a commons-crew tool, not a separate service. Before commons-crew's tool-execution loop reaches for build capability to close a gap, its first call is a search against artifact-commons — the same governed loop (propose, approve, execute) that already exists, just with one more tool type added to it. There is no separate matching engine to design or build.

## Modification path

Resolved: adapting an existing artifact is not architecturally distinct from building one from scratch. Both run through the same commons-crew tool-execution loop (edit_file, write_file, and so on) — the only difference is whether that loop starts from an existing artifact's content or a blank slate. An exact match needs no execution at all; a close match runs the loop seeded with the existing file; no match runs the same loop from nothing.
