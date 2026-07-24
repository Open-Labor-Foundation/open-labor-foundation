---
# Target path in the repo: .github/ISSUE_TEMPLATE/proposed-golden.md
name: Proposed golden
about: A session hit a judgment no golden covers. Propose the acceptance criterion instead of shipping unverified behavior.
title: "[golden] "
labels: proposed-golden
---

<!--
When a session reaches a decision no existing golden covers, its deliverable is
THIS issue, not code. A human ratifies the expected outcome; only then does the
behavior have an oracle and the work promote. This is the inversion: you author
acceptance criteria once, instead of reviewing every implementation forever.
-->

**Invariant this protects** (from VISION.md):

**Stakes**: governance-critical / high / medium

**Intent** (one line — the behavior being pinned):

**Input** (the frozen scenario):

**Expected outcome**:

**Why the current goldens don't cover it**:

---

**Ratification** (a maintainer checks these — this is the only human review the
change needs):

- [ ] The expected outcome is what the system *should* do, not merely what it
      does today
- [ ] Added to the golden set and passing in CI
