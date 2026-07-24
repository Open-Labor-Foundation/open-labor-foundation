# Governance goldens — the acceptance oracle

This directory is the answer to the question the whole stack was missing: **how
does a session decide it's done without round-tripping through a human's head?**
It does it by checking the change against frozen goldens — human-authored
(input → expected) pairs, one per governance-critical behavior, derived from the
invariants in `VISION.md`.

## The promote rule

A change is correct **iff every golden still passes**. That's the whole gate.

- A failing golden is never "fixed" by editing the golden. A failing golden means
  the change is wrong — or the invariant genuinely moved, which is a `VISION.md`
  decision made deliberately in its own PR, not a test tweak buried in a feature.
- The runner (`run.ts`) exits nonzero on any failure and is wired as a required
  status check (`.github/workflows/evals.yml`). "Promote when correct" now has an
  external condition, exactly like a UAT ticket at a day job — the thing that
  made the session-per-task workflow work there and not here.

## Sampling frame (why these goldens, and which ones next)

You cannot write a golden per catalog entry — that's thousands of specialists and
an eval bill you don't want on every PR. You don't need to. **Goldens track
stakes, not coverage.** Author them where a wrong answer is irreversible,
adversarial, or load-bearing for the governance claim, and sample the rest.

Two tiers, by cost:

- **Pure-function goldens (PR path).** Deterministic, sub-second, zero token cost.
  Every invariant reducible to a pure function lives here and runs on every PR.
  The delegation gate is the first (`delegation-gate.goldens.ts`).
- **Model-in-the-loop goldens (nightly).** Behaviors that require a real model
  call (e.g. "the model chooses `search_artifacts` unprompted") are flaky and
  cost tokens. Keep them a smaller, separate suite on a schedule — never on the
  PR path, or you couple merge latency and cost to model nondeterminism.

Next goldens to author, each tagged with its invariant and the seam it tests:

| Invariant | Behavior to pin | Seam | Tier |
|---|---|---|---|
| #2 no-ungated-execution | dispatch-to-commons-crew `decision` has no default; nothing auto-approves | `dispatch-to-commons-crew/decision` handler | pure |
| #2 no-ungated-execution | unsynced org fails closed to `advisor` before the gate | tier resolution in commons-crew | pure |
| #1 no-self-promotion | no tier transition raises a run's own authority | tier-transition validator | pure |
| #2 / authority | `propose_spec_correction` proposes only; never auto-executes a labor-commons PR | commons-crew allowlist (`AUTONOMOUS_PROPOSE_ONLY_*`) | pure |
| #7 independent-certification | certification runs independently of the catalog it certifies | labor-commons-curator entry | pure |
| #8 audit-precedes-execution | audit record is written before, not after, execution | commons-board execution path | pure |
| #5 reuse-before-build | `search_artifacts` is called before build capability | commons-crew tool loop | model, nightly |

That table is the backlog. Each row is one `[golden]` issue and one small file.

## The review inversion

When a session reaches a judgment **no golden covers**, its deliverable is a
*proposed golden* (`.github/ISSUE_TEMPLATE/proposed-golden.md`), not merged code.
A human ratifies the expected outcome once; from then on the loop checks it
forever. This is what moves you from live reviewer to acceptance-criteria author
— the only step in the pipeline that structurally cannot be handed to the model,
done once per behavior instead of once per implementation.

## Correction rate — the autonomy-readiness metric

The old `ARCHITECTURE.md` was dense with "an earlier version said X — actually
not-X." Every one of those was a session asserting done and being wrong later.
That rate is the single number that tells you whether autonomy is safe to widen:

**correction rate = (promotes later reopened or reverted) / (total promotes)**

Cheap to measure without new infrastructure:

1. Label every promote PR `promote`.
2. When a later session finds a promoted change was wrong, label the reopening
   issue/PR `regression` and reference the original.
3. Rate = `count(regression) / count(promote)` over a trailing window.

Drive it down before turning any org up a tier. If it isn't falling as goldens
accumulate, the goldens aren't covering the behaviors that actually break — fix
the frame, not the models.

## The freeze (scope)

Every new production-autonomy loop (devloop, keeper, curator, autonomous tool
selection) generates more output that the oracle must verify. Until the
correction rate is low and falling, **new production-autonomy work is frozen**;
the budget goes to goldens. `commons-devloop` retires or is absorbed once
commons-board has internal build capacity via commons-crew — as `ARCHITECTURE.md`
already states. Amplification of your judgment, not replacement of it.
