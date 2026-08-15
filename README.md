# Game Art Pipeline — Design Notes

Design notes for an asset generation pipeline: a design document goes in, and sprites, tilesets, and
UI come out in a form an engine loads without conversion.

This repository documents **how the pipeline was designed and why** — the structure, the failure
modes it was built around, and the reasoning behind each decision. It contains no implementation.

```
design document
   └─▶ spec check ──── spec blank ──▶ halt, return the document
        └─▶ concept art
             └─▶ 2D assets and frames
                  └─▶ pixel conversion   (only when the document asks for it)
                       └─▶ runtime assets
```

## The problem

Generating a good-looking asset is not the hard part. Getting **the same quality every time, in the
exact form the engine expects**, is. An asset that looks right but has the wrong dimensions, an
off-palette color, or frames that do not actually animate will fail at integration — and it will
fail quietly, after someone has already approved it.

So the pipeline is built less around generation than around **the points where generation goes wrong
without announcing it.**

## Contents

| Document | What it covers |
|---|---|
| [Pipeline](docs/pipeline.md) | The stages, what each one owns, and what verification catches |
| [Evaluation](docs/evaluation.md) | How generated assets are scored, and why the scoring is shaped that way |
| [Design decisions](docs/decisions.md) | Ten judgment calls and the failures behind them |

## Principles

**One source of truth.** Every asset property comes from the design document. Nothing is inferred
from a filename or a prose description. When a required value is blank the pipeline stops and sends
the document back rather than guessing — a guess here propagates silently through every later stage.

**Verify between stages.** Each stage validates its own output before the next begins, so a defect
is caught by the stage that understands it instead of arriving downstream disguised as valid input.

**Don't generate what you can determine.** Where the output is fully determined by the input,
generation adds variance and nothing else.

**Measure in a unit that survives a configuration change.** A threshold expressed in the wrong unit
does not fail when the configuration changes — it quietly starts covering different things.

## Scope

Implementation, agent definitions, evaluation results, and generated assets are not included. What
is documented is the design reasoning, written to stand on its own.
