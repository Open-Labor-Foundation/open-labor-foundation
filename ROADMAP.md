# Roadmap

The Open Labor Foundation stack is built and test-functional. The path from here is making it genuinely accessible to any worker — not just workers who know how to run a Docker container.

---

## Where we are

The full stack exists and has been tested:

- **labor-commons** — seeded with NAICS-grounded specialist definitions covering virtually any task a worker needs help with and every function an organization needs to operate
- **commons-keeper** — catalog quality loop, autonomous validation and issue filing, ready to deploy against labor-commons
- **commons-specs** and **commons-artifacts** — community libraries, ready to receive contributions
- **foundation-site** — source ready, awaiting deployment to openlabor.foundation
- **commons-idea** — base complete: SKILL.md, governance docs, and templates in place
- **commons-board** — design complete; implementation in progress
- **commons-crew** — in development; substantial work remaining

---

## The primary challenge: packaging, not inference

commons-crew, commons-board, and commons-idea require LLM inference to function, and that part is resolved: a Featherless.ai API key covers it by default, with self-hosted inference as the alternative for an operator who wants to run their own models.

What's still unsolved is packaging. The stack runs in Docker on a host the user controls — which works, but creates a significant barrier for anyone who isn't comfortable with containers and a terminal. Mobile adds a further wrinkle: current phone hardware can run small local models but not at the quality level genuine multi-agent specialist work needs, so mobile users need to be thin clients connecting to an inference backend rather than running it locally.

**This is the near-term problem to solve.** Desktop and mobile apps that replace the Docker/terminal requirement are the prerequisite for real adoption by the small business owners and worker collectives this is built for.

---

## Near term

- [ ] Desktop and mobile apps — native apps replacing the Docker/terminal requirement
- [ ] Launch openlabor.foundation
- [ ] Contribution workflow — make it easy for any worker to contribute to labor-commons through commons-idea without touching a terminal
- [ ] In-product addin publishing — commons-board operators publish addins back to commons-artifacts from inside the platform, no git required
- [ ] Public release of all repos

---

## What communities can build

Once the accessibility barrier is cleared, the stack enables communities of workers to build and run their own platforms without a commercial intermediary.

**Local delivery and logistics** — dispatch, route optimization, earnings tracking, collective governance. Workers running their own platform instead of paying a platform fee.

**Trades and skilled labor** — job estimating, code compliance, scheduling, materials, licensing, invoicing. Collectives eliminating dependency on staffing agencies and franchise models.

**Legal services** — contract review, document drafting, compliance tracking, case management, billing. Independent attorneys and legal aid collectives.

**Accounting and finance** — bookkeeping, tax preparation, payroll, compliance, financial planning. Independent practitioners and bookkeeping collectives.

**Healthcare** — documentation, scheduling, billing, compliance, care plan management. Independent practitioners reducing administrative burden.

**Creative and media** — project management, contracts, licensing, accounting, client communication. Creative professionals and studios.

**Home and community care** — scheduling, documentation, billing, compliance, care plan management. Independent care worker collectives.

---

## Long term

- Community governance — formal collective membership, contribution recognition, sustainability model
- Community-governed catalog — specialist acceptance decisions made by domain experts, not maintainers
- Cross-collective coordination — specialists and workflows shared between different industry communities
- Regional deployment models — communities running their own infrastructure with local governance

---

*The fastest path to any of the above is workers contributing their expertise to labor-commons. Every contribution compounds.*
