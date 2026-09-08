# ADR-002 — Chainage by projection onto the road axis

**Date:** 2026-01 · **Status:** Accepted

## Context

Road-works control is organised by **chainage** — distance along the road from its
origin. Every analysis depends on it: grouping points into cross-sections, ordering
stations, detecting gaps, plotting slope and width along the job.

Field data does not contain chainage. It contains northing, easting and elevation, plus
a code. Chainage has to come from somewhere.

Three options:

1. **Have the surveyor record it.** An extra column, entered by hand in the field,
   transcribed later. It was how the work was already being done.
2. **Derive it from point order.** Assume points were taken sequentially along the road.
3. **Compute it geometrically**, by projecting each point onto the road's axis.

Option 1 is a manual step that is tedious and error-prone, and its errors are silent: a
mistyped chainage puts a point in the wrong cross-section and nothing looks wrong
afterwards. Option 2 is seductive and false — crews revisit sections, work in both
directions, and take reference points out of order. Any assumption about ordering breaks
on a real job.

## Decision

**Compute chainage geometrically.** The road axis is loaded as a Shapely `LineString`,
and each surveyed point's chainage is its projection onto that line.

The whole operation is one call: `axis.project(Point(easting, northing))`. Shapely
returns the distance along the line to the nearest point on it — which is precisely the
definition of chainage.

Everything downstream falls out of this. Cross-sections become groups of points at
similar chainage. Gaps become chainage jumps — one of the nine alert types. Control
charts plot naturally against it.

The road axis becomes an explicit **input** to the process rather than an assumption
buried in the data, which is the deeper part of the decision: the reference geometry is
now a named thing that can be inspected, swapped and versioned.

## Consequences

**What was gained**

- A manual, error-prone column disappears from the workflow.
- Chainage is reproducible: the same points and the same axis always give the same
  answer, which matters when a report is questioned months later.
- Point ordering stops mattering. Crews work however they work.
- Errors become visible instead of silent — a point far off the axis produces a strange
  projection, which surfaces as an alert rather than quietly landing in the wrong
  cross-section.
- Cross-section grouping, gap detection and charting all inherit one consistent
  definition.

**What it cost**

- **The road axis is now a required input**, and a wrong or misaligned axis corrupts
  everything downstream. The dependency is real; it is mitigated by the axis being
  explicit and reviewable rather than implied.
- **Projection is nearest-point**, so on a tight curve a point can project ambiguously,
  and on a road that doubles back it can project to the wrong side entirely. Not
  encountered on these jobs, but it is a genuine limitation of the approach rather than
  a hypothetical.
- **A dependency on Shapely** and on its planar assumptions. Fine for local survey
  coordinates over a road; it would need care over long distances or across projections.
- Points far from the axis still get a chainage — a plausible-looking number for a bad
  input, which is why the distance from the axis has to be watched as its own signal.

The general point: **prefer computing a value from geometry over asking a human to
maintain it.** A derived value is reproducible and its failures are inspectable; a typed
value is neither.
