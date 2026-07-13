# Open Labor Foundation

A governed AI orchestration platform: a catalog of agent definitions, each
with an explicit authority boundary and a tested evaluation suite, running
under an engine that executes them inside a real hierarchy with human
approval gating. It isn't tied to one industry — it's built to close the
same AI-adoption gap that exists everywhere, at whatever depth a given field
needs it:

- A bookkeeper wants tax-code guidance that stays current without hiring a
  research team — an individual-scale problem.
- A home-care worker collective wants scheduling and billing it owns
  outright, governed by its own members — a collective infrastructure
  problem.
- A small manufacturer wants to stop renting a proprietary quoting and
  inventory system it can never fully customize — a business rewriting a
  tool it currently licenses.

Same platform underneath all three, just different depths of the same gap.

## Why it's a commons and not another vendor

Every one of those problems could be solved by paying a SaaS vendor, and
each vendor would take a permanent cut for solving it. OLF's bet is that if
the governance layer and the orchestration engine are both open and
self-hostable, whatever a business builds to solve its own version of the
problem can be handed to the next business for free — and the next one
after that. Enough of that, function by function, and the software stack an
industry currently rents becomes optional.

The realistic path there is small businesses first — operators opting out
of fees they already resent paying (delivery cuts, POS subscriptions,
scheduling tools). The further destination is workers organizing regionally
into collectives that own this layer collectively instead of individually.
Gig-economy platforms are the sharpest illustration of what happens when
that layer is owned by an extractive third party instead: the customer pays
more, the platform takes its cut, and the worker keeps what's left.

## The stack

Several repos, each with one job, that only add up to a platform together:

- **[labor-commons](https://github.com/Open-Labor-Foundation/labor-commons)** — the governance layer. Agent definitions, one per field, each with an authority boundary and a scenario-based test suite. Seeded from the NAICS industry classification; coverage is tracked against a target and grows continuously.
- **[commons-keeper](https://github.com/Open-Labor-Foundation/commons-keeper)** — runs continuously against labor-commons, validating definitions and flagging decay before it becomes drift.
- **[commons-board](https://github.com/Open-Labor-Foundation/commons-board)** — the orchestration engine. Runs a business or collective end to end under a governed hierarchy (board → chairs → orchestrating agents → worker agents), in business or collective mode. Addins — the mechanism described above — are developed and installed here.
- **[commons-crew](https://github.com/Open-Labor-Foundation/commons-crew)** — the same governed specialists, scoped to one person instead of an organization. In development.
- **[commons-idea](https://github.com/Open-Labor-Foundation/commons-idea)** — turns a plain-language conversation into a spec, a working deliverable, or a labor-commons contribution. No install: it's a conversation you have with any LLM.
- **[commons-specs](https://github.com/Open-Labor-Foundation/commons-specs)** and **[commons-artifacts](https://github.com/Open-Labor-Foundation/commons-artifacts)** — where commons-idea output gets shared back: written records in one, runnable deliverables and commons-board addins in the other.

## What keeps it growing

No central team writes every definition or every addin. The catalog and the
addin library grow because operators use the platform and share back what
they built — from that point on, it's available to everyone else running
the same stack.

## Who this is for today

The honest state: running any of this requires Docker and a terminal, which
rules out most of the people it's ultimately meant for. Inference access
itself isn't the blocker — a provider API key or self-hosted inference
covers that — the remaining barrier is packaging. A no-terminal deployment
(desktop and mobile apps) is a near-term commitment, not built yet. See
[ROADMAP.md](ROADMAP.md) for what's built versus in progress.

## Governance

Two questions, both answered in [GOVERNANCE.md](GOVERNANCE.md): what makes a
determination binding inside one deployment (commons-board's autonomy
tiers — never self-promoted, set by that org's own process), and what makes
something trusted enough to be treated as binding across the whole commons
(certification — who runs it, what happens on disagreement, how it gets
revoked). See [ARCHITECTURE.md](ARCHITECTURE.md) for how the repos fit
together.

## Contributing

Three ways in: domain knowledge (via commons-idea, no code required), an
idea for a tool (also via commons-idea), or infrastructure work (for
engineers). See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Specialist definitions and specs: [CC-BY-SA 4.0](LICENSE) — share alike.
Software: [AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html) —
modifications shared back. OLF never charges or meters use of the
self-hosted platform itself.
