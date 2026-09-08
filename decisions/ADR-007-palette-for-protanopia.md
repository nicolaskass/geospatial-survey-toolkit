# ADR-007 — A palette chosen for protanopia

**Date:** 2026-01 · **Status:** Accepted

## Context

The nomenclature editor shows survey points on an interactive map, with colour carrying
the meaning: which contractor, which layer, which zone. Colour is not decoration here —
it *is* the encoding. If two categories are not distinguishable, the map does not work.

The default approach is a stock palette, or red-amber-green because it reads as
good-warning-bad.

**Protanopia and deuteranopia** — red-green colour deficiency — affect roughly 8% of men
of Northern European descent. A field crew is a small, overwhelmingly male population.
The probability that somebody using this map cannot cleanly separate red from green is
not marginal.

The failure mode is what makes this worth a decision record. Nobody files a bug saying
"I cannot distinguish your categories." They squint, they guess, they cross-check against
the table, and they quietly trust the tool less. The defect never surfaces as a
complaint — it surfaces as the map not being used.

Nothing forced the issue. No compliance requirement applies to an internal tool, no
procurement checklist, no auditor.

## Decision

**Pick the palette for protanopia, and say so in the interface.**

Layer and category colours are selected to remain distinguishable under red-green colour
deficiency. The choice is stated in the application's own help text and in its
documentation, so a user can tell it was deliberate — which matters, because a user who
knows the palette was chosen for them will report a case where it still fails.

Colour is also not the only channel: the underlying data remains visible in tables and
exports, so the map is a faster path to the answer rather than the sole path to it.

## Consequences

**What was gained**

- The map works for the whole crew, including the members who would never have mentioned
  that it did not.
- Stating it in the help text turns an invisible property into a visible commitment.
- Constraining the palette forced the categories to be genuinely separable rather than
  merely different — a discipline that improved the map for everyone.
- Redundant encoding means colour perception is never the only route to the information.

**What it cost**

- **A smaller usable palette.** Protanopia-safe colours are a restricted set, so the
  number of categories distinguishable at once is lower than with a free choice.
- **It is less pretty** than an unconstrained palette. Accepted without hesitation for a
  working tool.
- **Deuteranopia and tritanopia are not separately verified.** The palette targets
  protanopia specifically. Overlap with deuteranopia is substantial but the claim being
  made is the narrow one, not a general accessibility claim.
- **No automated check.** Nothing prevents a future contributor adding a colour that
  breaks the property. A palette-contrast test would fix that and does not exist — this
  is the honest gap.

The reasoning generalises. **The users who suffer from an accessibility failure are
usually the least likely to report it**, so the absence of complaints is not evidence
that a tool works. In a visualisation whose entire function is distinguishing categories
by colour, the palette is not styling — it is correctness, and it belongs in the same
category of concern as getting the geometry right.
