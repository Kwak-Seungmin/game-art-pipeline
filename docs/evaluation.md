# Evaluating Generated Assets

How the scoring was designed, and why it took the shape it did.

---

## The problem

"Is this good enough?" answered differently by different people is not a usable signal. Without a
fixed standard you cannot decide what to fix, what to regenerate, and what to pass downstream.

Worse, you cannot tell whether a change to the pipeline improved anything — because the measurement
moves along with the thing being measured.

So the standard came first, and was revised repeatedly against real output.

---

## Several axes, no combined total

Assets are scored on **multiple independent axes** rather than one number. The axes separate
concerns that fail for unrelated reasons — the finish of the asset itself, conformance to what was
specified, consistency across a set, and whether animation actually animates.

**Each axis stands alone. There is no total.**

A single number hides which axis collapsed. Two assets scoring the same can mean entirely different
things — one uniformly mediocre, one excellent with one broken property — and only one of them tells
you what to do next.

The objection is that a total is convenient for ranking. But the rubric was never built to rank
assets against each other. It was built to answer *what do I fix first*, and a total actively
obscures that.

---

## Not every item should be added the same way

Some conditions must hold regardless of anything else. Others accumulate.

Treating both additively lets an asset that fails something fundamental still score respectably by
being polished elsewhere — which is backwards. A sprite with the wrong dimensions is not partially
usable.

So the two kinds of item combine differently: **conditions that must hold act as a factor**, while
finish quality accumulates on top. A collapsed fundamental drives the axis toward zero no matter
what else is true. The arithmetic enforces the priority rather than relying on a reviewer to weigh
it correctly.

Conformance items grade proportionally rather than pass/fail, so a near-miss and a total miss are
not recorded as the same event.

---

## Paperwork became gates, not scores

Items about records and provenance were removed from the scored axes entirely and turned into
**gates** — pass, fail, or not-measurable. They carry no points.

Two failures were being conflated. *The work is bad* and *the work could not be evaluated* are
different states requiring different responses, but both were landing as a low score.

Separated, a missing generation record returns **not-measurable** rather than zero. The asset is not
condemned; it is set aside until it can be judged. And a genuine quality failure can no longer hide
behind a documentation failure.

---

## Scores carry their instrument

A revised rubric is **a different instrument**, and scores from different versions are not
comparable. Every score records the version it was measured under, and cross-version comparison is
treated as invalid rather than approximate.

This matters more than it sounds. Without the rule, revising the rubric and re-scoring looks
identical to the output improving — the most dangerous kind of false signal, because it confirms
whatever change was just made.

---

## What the rubric is for

Not grading. The purpose is to make three questions answerable:

1. **What do I fix first?** — the collapsed axis, not the low total
2. **Did that change help?** — comparable only within one instrument version
3. **Can I trust this number?** — provenance decides whether it was measurable at all
