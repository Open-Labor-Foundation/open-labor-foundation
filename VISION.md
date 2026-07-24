# Vision

This document states what the Open Labor Foundation stack **is** and what must
**always be true** of it. It is the fixed target the whole ecosystem is built
toward.

**This document does not move.** It contains no implementation status, no PR
references, no "built / merged / verified," no "current gap," no corrections. If
a sentence here could become false by writing code, it does not belong in this
file — it belongs in the issue tracker. Agents and contributors **read** this
file as the target; they do not **write** to it. Changing it is a deliberate
human act about *intent*, reviewed as a change of direction — never as a status
update.

Structural detail about how the repos wire together lives in `ARCHITECTURE.md`.
What is currently built lives in issues and the project board. Neither belongs
here.

---

## Mission

If AI can genuinely replace or augment labor at the level of a professional
role, that claim should be **proven, not asserted** — and proven in a
controlled, governed environment, not a chaotic one.

The proof starts from a single, comprehensive catalog of what every profession
actually does, and builds upward into a governed system that can put that
catalog to work — from the top of an organization's decision-making structure
all the way down to the individual worker-level task.

It must never collapse into either failure mode:

- an **unaccountable autonomous agent** (too much autonomy, no governance), or
- a system **too shallow to matter** (governance with nothing real underneath).

The end goal is to provide this system as open source, removing the operating
burden — software cost, staffing overhead, administrative load — for a business
owner in any industry, **without requiring them to understand or manage the AI
underneath it.**

It is not tied to one industry. It closes the same AI-adoption gap wherever it
exists, at whatever depth a given field needs: an individual bookkeeper, a
worker collective that wants to own its own scheduling and billing, a small
manufacturer rewriting a tool it currently rents. Same platform underneath,
different depths of the same gap.

## Why a commons, not a vendor

Each of those problems could be solved by a SaaS vendor taking a permanent cut.
The bet is that if the governance layer and the orchestration engine are both
open and self-hostable, whatever one business builds to solve its version of the
problem can be handed to the next business for free — and the next after that.
Function by function, the stack an industry rents becomes optional. OLF never
charges or meters use of the self-hosted platform.

---

## What the system is

Three components form the actual product. Everything else exists to build toward
them, support them, or keep them honest.

### labor-commons — the catalog

A single catalog of specialist definitions, one per profession, seeded from the
NAICS classification of the economy. Each definition specifies **what an agent
standing in for that profession is authorized to decide, what it must escalate
or refuse, and the boundary that keeps it faithful to the role rather than
improvising past it.**

- **Reuse over ownership.** A definition is never scoped to whichever business or
  workflow needed it first. Any part of the platform may invoke any specialist
  whose boundary fits.
- **The catalog is data, not runtime.** It stays thin and authoritative;
  generation, maintenance, and certification live in separate repositories.
- **Two axes.** An industry-vertical axis (what a company operating *in* an
  industry needs) and an orthogonal corporate-function axis (what *any*
  organization's internal Finance, HR, Legal, Marketing, Sales, Operations, or
  Product function needs, regardless of the organization's industry).

### commons-board — governed authority

The proof that AI-run governance works at the top of an organization. A **chain
of authority** — board → chairs → directors → workers — not a flat pool of
agents. Its own job is the governance wrapper: the authority chain, the autonomy
tier, the audit trail, human oversight. It is not the mechanism that resolves
which specialist does the work.

### commons-crew — recursive delegation

The single mechanism used at **every** layer of an organization, including the
chair. Each hop is scoped one level down and independently assembles whatever
specialist its task needs from labor-commons. This is what makes the full
breadth of the catalog load-bearing rather than dormant: every layer resolves
through the same recursion. commons-board is what you get when a collection of
these instances is wrapped in governance.

### artifact-commons — the second commons

A peer catalog to labor-commons, holding **previously built solutions** rather
than professional boundaries. The same reuse principle, one layer up. When a gap
is identified, the order is fixed: **check for a match first, adapt an existing
solution if one is close, build from scratch only if nothing usable exists —
then publish the result back so it is never built twice.**

---

## Invariants — what must always be true

These hold regardless of implementation state. They are the properties any
correct version of the system satisfies, and the seeds from which acceptance
tests are written.

1. **Autonomy never self-promotes.** An agent's authority increases only through
   explicit human promotion. No component raises its own tier.
2. **Nothing with real-world effect executes un-gated.** Every side-effecting
   action passes through propose → approve → execute, with the approval
   attributable to a real, identified human.
3. **Every specialist has an authority boundary.** Each definition states what it
   may decide, what it must escalate, and what it must refuse. A definition
   without all three is incomplete.
4. **An agent stays faithful to its role.** It does not improvise past the
   boundary of the profession it stands in for.
5. **Reuse precedes building.** Both commons are searched before new work is
   generated; new work is published back.
6. **The catalog is thin and authoritative.** No runtime logic lives in
   labor-commons.
7. **Whatever certifies or checks is independent of the thing it certifies.**
8. **An immutable audit trail precedes execution**, not follows it.
9. **The operator never has to understand or manage the AI underneath** to run
   the platform.

## Governance — two distinct questions

1. **Within one deployment:** what makes a determination binding is that
   deployment's own autonomy tier — set by that organization's own process,
   never self-assigned.
2. **Across the whole commons:** what makes something trusted enough to be
   treated as binding everywhere is certification — who has standing to certify,
   how disagreement is resolved, and how a certification is revoked.

The ratified model for both lives in `GOVERNANCE.md`. This document only fixes
*that the two questions are distinct and both must be answered* — not their
current implementation.

---

## Out of scope for this document

Progress, status, what is built, what is broken, what is next, and any
correction to a past claim. All of it belongs in issues and the project board.
If you came here to record that something now works, you are editing the wrong
file.
