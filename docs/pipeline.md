# Pipeline Structure

What each stage owns, and what verification is there to catch.

---

## Overview

```
design document
   └─▶ spec check ──── spec blank ──▶ halt, return the document
        └─▶ concept art
             └─▶ 2D assets and frames
                  └─▶ pixel conversion   (only when the document asks for it)
                       └─▶ runtime assets
```

Each stage is a separate agent. They are split not by convenience but by **failure mode** — concept
art fails on style coherence, 2D assets on conformance to what was specified, pixel conversion on
palette and seams. Splitting them means each failure is caught by the stage equipped to recognize
it, and each stage's verification is narrow enough to be strict.

---

## Spec check

Confirms every asset has the values it needs before anything is generated — dimensions, kind,
animation properties, palette, naming.

**A blank value halts the pipeline.** The asset kind is not inferred from the filename, and the
palette is not inferred from the concept art. The document goes back to be filled in.

This is the most load-bearing rule in the pipeline. An inferred value is indistinguishable from a
specified one at every later stage, so nothing downstream can catch it being wrong. The error never
surfaces as an error — it surfaces as an asset that quietly does not match the game.

---

## Routing

Before generation, each asset takes one of two paths: **generate large and downsample**, or
**author directly at target resolution**.

The decision is made in the unit that stays meaningful across projects rather than in raw pixels,
because the relationship between the two varies per project. A threshold written in the wrong unit
stops applying the moment a project changes its configuration.

**Why two paths.** Downsampling averages detail away, and below a certain size there is nothing left
to average. Small assets came back either empty or as a single flat color — and the flat-color case
is the dangerous one, because an asset that fills its canvas with the correct color **passes a color
check while carrying no shape at all.** No quantization setting recovers detail that was never
sampled; only a different path does.

---

## Concept art

Establishes the look before any production asset exists, and becomes the reference every later stage
measures against.

Color here is **reference only** — the design document makes the final call, so a concept image is
read for form and silhouette rather than for its exact colors.

One rule proved necessary in practice: an asset's identity color is *what it reads as*, not *what
its body is painted*. An asset whose identity lives in a rim light or a glow collapses into a
featureless dark shape if that color is applied to the body instead.

---

## 2D assets and frames

Produces the assets themselves and, for animated ones, their frames. Motion assets pass through
generation, frame extraction, and selection — with a deterministic filter rejecting frames that are
mis-scaled or clipped before a human ever looks at them.

**The gate here:** frames must differ from one another. An asset ordered as animated whose adjacent
frames are identical has not been animated — it has been duplicated.

---

## Pixel conversion *(conditional)*

Runs **only when the document declares that style.** Otherwise the 2D assets pass through unchanged.

This stage uses **no generative model.** Color and form are separated and the declared palette is
assigned directly, so the output palette is exactly the specified one rather than approximately it.

Tiling gets an additional check: a tile that looks correct in isolation can still show a visible
join when repeated, so continuity is verified separately from appearance.

---

## What verification targets

Not obvious breakage — someone will notice that. Verification earns its cost only where the failure
is **invisible to review**:

- animation whose frames are identical, while the frame count is correct
- a required length met by duplicating a frame rather than animating
- seams that appear only once tiles repeat
- foreground that reads as background, using colors that are all legitimately from the palette
- quality dropping against a previously accepted version

Every one of these describes an asset a reviewer would plausibly approve. That is precisely why the
check has to be mechanical.

---

## Output

Assets arrive in the form the engine loads directly — correct dimensions, declared palette,
animation as specified, named as the document said they would be. No conversion step sits between
the pipeline and the build.
