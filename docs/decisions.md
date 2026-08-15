# Design Decisions

Each of these was a judgment call, and most were made after something failed in a way that was hard
to notice.

---

## 1. Halt on a blank spec instead of inferring

**Decision.** When a required field is missing, stop and send the document back rather than guessing
from context.

**Why.** An inferred value is indistinguishable from a specified one at every later stage. Nothing
downstream can tell that a guess happened, so nothing downstream can catch it being wrong. The error
does not surface as an error — it surfaces as an asset that quietly does not match the game.

Halting is expensive and unpopular. It stops work and puts it back on whoever wrote the document.
That cost is the point: the alternative is discovering the mismatch after integration, when it costs
more.

---

## 2. Measure in the unit that survives a configuration change

**Decision.** Routing thresholds are expressed in the unit that stays meaningful across projects,
not in raw pixels.

**Why.** The relationship between the two varies per project, so a threshold written in the wrong
unit means one thing under one configuration and something else under another. The rule silently
stops applying — it does not fail, it just quietly covers different assets than intended.

**How it went wrong first.** The documentation and the implementation ended up expressed in
different units, and drifted apart. A set of assets the documentation excluded was in fact being
handled by the other path the whole time, and nobody noticed, because both paths produce plausible
output.

The fix was not just correcting the number but removing the duplicate. The threshold lives in one
place, and every routing decision records which rule decided it.

---

## 3. Author small assets directly rather than downsampling

**Decision.** Below a size threshold, do not generate large and shrink. Author at the target
resolution.

**Why.** Downsampling averages detail away, and below a certain size there is nothing left to
average. Small assets came back either empty or as a single flat color.

The flat-color case is why this needed a structural fix rather than a parameter change. An asset
that fills its canvas with the correct color **passes the color check** — the verification designed
to catch wrong colors sees exactly the right one. The asset is wrong in a way that particular check
was not looking for.

No quantization setting recovers detail that was never sampled. Only a different path does.

---

## 4. No generative model where the output is determined

**Decision.** Pixel conversion assigns the declared palette directly rather than generating.

**Why.** A generative model asked to produce a constrained style will produce *something that looks
like* it — approximately the right palette, approximately on-grid. Approximately is the problem. The
palette was specified for a reason, and an asset off by a few values will not read as part of the
same set.

Where the output is fully determined by the input, determinism is not a compromise. It is the
correct implementation, and it makes the result reproducible.

---

## 5. Split stages by failure mode

**Decision.** Each production stage is a separate agent rather than one process.

**Why.** Each fails differently — style coherence, conformance, palette and seams. A single process
would need to hold every failure mode at once, and in practice would check whichever was most
recently a problem.

Separating them keeps each stage's verification narrow enough to be strict, and means a defect is
caught by the stage that can explain it rather than arriving downstream disguised as valid input.

---

## 6. Verify what looks fine

**Decision.** Verification targets defects that pass visual review.

**Why.** An obviously broken asset needs no check; someone will notice. Verification earns its cost
only where the failure is invisible — animation whose frames are identical while the count is
correct, a length met by duplication, seams that appear only once tiles repeat, foreground that
reads as background using entirely legitimate colors.

Each of these describes an asset a reviewer would plausibly approve. That is precisely why the check
has to be mechanical.

---

## 7. Score axes separately, refuse a total

**Decision.** Multiple independent axes, no combined number.

**Why.** A total hides which axis collapsed. Two assets scoring the same can mean entirely different
things, and only one of them tells you what to do next.

The objection is that a total is convenient for ranking. But the rubric was never built to rank
assets against each other — it was built to answer *what do I fix first*, and a total actively
obscures that.

---

## 8. Let the arithmetic enforce the priority

**Decision.** Conditions that must hold act as a factor; finish quality accumulates on top.

**Why.** Treated additively, an asset that fails something fundamental can still score respectably
by being polished elsewhere. That is backwards — a sprite with the wrong dimensions is not partially
usable.

Making the fundamentals a factor means a collapsed one drives the score toward zero regardless of
what else is true, instead of relying on a reviewer to weigh it correctly.

---

## 9. Separate "bad" from "unmeasurable"

**Decision.** Record and provenance items were removed from the scored axes and became gates.

**Why.** Two failures were being conflated. *The work is bad* and *the work could not be evaluated*
are different states requiring different responses, but both were landing as a low score.

Separated, a missing record marks the asset not-measurable rather than zero — set aside until it can
be judged. And a genuine quality failure can no longer hide behind a documentation failure.

---

## 10. Label every score with its instrument

**Decision.** Scores record the rubric version they were measured under, and cross-version
comparison is treated as invalid.

**Why.** The rubric was revised repeatedly, and each revision changed what the numbers mean. Without
this rule, revising the rubric and re-scoring looks identical to the output improving — the most
dangerous kind of false signal, because it confirms whatever change was just made.

Refusing the comparison is less useful than an approximate one would be, and more honest than a
wrong one.
