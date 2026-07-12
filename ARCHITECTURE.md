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

**Current gap:** the catalog currently grows one way — AI inference against the NAICS backlog. The other intended growth path, corrections and additions from actual practitioners in a given profession, has no working mechanism yet. commons-board and commons-crew are expected to provide user-guided spec updates; that isn't built.

### commons-board — governed authority

The proof that AI-run governance can work at the top of an organization. Structurally, commons-board is a collection of commons-crew instances — chairs, and everything beneath them — wrapped in governance: a chain of authority rather than a flat pool of agents, autonomy that only ever increases through explicit human promotion, never self-promotion, and an immutable audit trail before anything executes.

Chairs are not a separate concept commons-board staffs on its own. A chair is simply the top-level commons-crew instance in that collection, governed and audited the same way every layer beneath it will be. commons-board's own job is the governance wrapper — authority chain, autonomy tier, audit trail, human oversight — not specialist resolution. There is exactly one mechanism for pulling a specialist out of labor-commons, and commons-crew is it, used uniformly from the chair down to the individual worker.

**Current gap:** commons-board's existing implementation duplicates this — it has its own direct SQL-based resolver against labor-commons, built before commons-crew's recursive shape existed. That resolver should retire in favor of treating chairs as commons-crew instances like every other layer, once commons-crew supports recursive delegation. Until then, commons-board's reach into labor-commons is also limited to whatever its own resolver maps to the chair layer — the hundreds of line-level specialists the catalog actually contains are unreachable through commons-board today.

### commons-crew — recursive delegation

The single mechanism used at every layer of an organization — including the chair itself. A chair is a commons-crew instance; it hands a request to a director, itself another commons-crew instance; the director hands it to a department, the department to a worker — each hop scoped one level down, independently assembling whatever specialist its task needs from labor-commons. This is also what makes the full breadth of labor-commons load-bearing rather than dormant: instead of only a chair-level slice of the catalog ever being reached, every layer, including the chair layer itself, resolves through the same recursive mechanism. commons-board is what you get when this collection of instances is wrapped in governance.

commons-crew already has the right engine for this — a governed tool-execution loop (propose, approve, execute) with explicit gating on anything with real-world side effects. What it does not yet have is the recursive shape: a way for one instance to delegate to a child instance, a notion of which layer of an organization it's operating at, or a connection back to commons-board to sit in as a chair in the first place. It is currently built and documented as a single flat personal assistant for one individual, not a composable delegation primitive.

**Current gap:** not implemented as envisioned, in two respects — there is no delegation between instances at all yet, and commons-board does not yet treat its own chairs as commons-crew instances rather than a separately staffed concept. This is the single most consequential piece of unbuilt architecture in the stack — it's what commons-board is missing, and what makes labor-commons' full catalog usable.

### artifact-commons — the second commons (new repository, not yet created)

A peer catalog to labor-commons, holding not professional boundaries but previously built solutions. The same reuse principle applies one layer up: a solution built by one business isn't scoped to that business. Any other operator can use it as-is or adapt it, without needing to know the catalog exists — that responsibility sits with the platform (commons-board / commons-crew), not the user.

The intended sequence when a gap is identified: check artifact-commons for an exact or close match first, adapt an existing artifact if one is close, and only build from scratch if nothing usable exists — then publish the result back so it's never built twice. This is what completes commons-board's "determines but can't close" gap: the actuation arm is check-then-adapt-then-build, with build-from-scratch as the exception, not the default.

This is a new repository, not a repurposing of `commons-artifacts` (see Relics, below) — the clean separation avoids carrying forward a repo whose schema and history were shaped by an earlier, narrower idea.

## Repository classification

### Core (the product itself)
- **labor-commons** — the catalog
- **commons-board** — governed authority
- **commons-crew** — recursive delegation (not yet implemented as envisioned)
- **artifact-commons** — reusable solutions (not yet created)

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

**Per-organization authority is already answered by a mechanism that exists today.** commons-board's autonomy tiers are the answer: Advisor tier means every determination is advisory until a human approves it; higher tiers mean progressively more is presumptively binding within a pre-approved scope, with anything outside that scope or above a defined threshold still escalating to a human. Autonomy never self-promotes — a deployment's tier, and therefore the weight its determinations carry, is set by that organization's own governance process (owner approval in business mode, member vote in collective mode). Nothing new needs to be built for this. It needs to be stated as policy, plainly, rather than left implicit in the autonomy-tier code.

**Platform-level authority — what it takes for something to be trusted enough to be treated as binding across the whole commons, not just within one org's deployment — is a different question, and isn't answered yet.** The mechanism for it already exists in outline: certification. labor-commons-curator's independently-graded certification pipeline is what's supposed to answer this for specialists in labor-commons; artifact-commons will need an equivalent certification gate before an artifact is trusted enough to be surfaced as a match rather than merely discovered. What's still missing is the ratified statement underneath the mechanism — who has standing to certify, what happens on disagreement, how a certification gets revoked. That's a founding document this repository should eventually hold, not a technical build.

## artifact-commons matching

Resolved: matching is a commons-crew tool, not a separate service. Before commons-crew's tool-execution loop reaches for build capability to close a gap, its first call is a search against artifact-commons — the same governed loop (propose, approve, execute) that already exists, just with one more tool type added to it. There is no separate matching engine to design or build.

## Modification path

Resolved: adapting an existing artifact is not architecturally distinct from building one from scratch. Both run through the same commons-crew tool-execution loop (edit_file, write_file, and so on) — the only difference is whether that loop starts from an existing artifact's content or a blank slate. An exact match needs no execution at all; a close match runs the loop seeded with the existing file; no match runs the same loop from nothing.
