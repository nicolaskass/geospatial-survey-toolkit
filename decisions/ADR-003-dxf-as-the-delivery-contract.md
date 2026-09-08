# ADR-003 — DXF as the delivery contract

**Date:** 2026-01 · **Status:** Accepted

## Context

The analysis produces geometry: cleaned points, cross-sections, per-contractor polygons.
That geometry has to reach a civil engineer who will check it, annotate it, and fold it
into project documentation.

Three ways to deliver it:

1. **A viewer of my own** — render the geometry in the app, let the user pan and zoom.
2. **An image export** — PNG or PDF of the plan.
3. **A CAD file** the recipient opens in the software they already use.

Option 1 is the tempting one, and it is a trap. It is the most work, it never matches
the CAD tools the industry has spent decades refining, and it produces a dead end: the
engineer can look but cannot work. Option 2 has the same dead end without the work.

The question that settled it was not "what can I build" but **"what does the recipient do
next?"** They open it in CAD and keep working. Anything that does not land there makes
the recipient re-draw by hand what the tool already computed.

## Decision

**DXF is the primary output**, generated with `ezdxf`, alongside a PDF report for the
signature path and CSV/PNG/SVG for the tabular and presentational ones.

DXF specifically, rather than a native CAD format: it is the interchange format every CAD
package reads, it is documented, and it does not tie the toolkit to one vendor's release
cycle.

The generated drawing is structured for the recipient — clean geometry organised so the
contractor separation survives the handoff — rather than being a dump of everything the
analysis knows.

**The toolkit ends where the industry's tools begin.** It does the computation nobody
else was doing, and hands off in the format everybody already uses.

## Consequences

**What was gained**

- The output is usable, not just viewable. The engineer keeps working instead of
  re-drawing.
- No viewer to build, and no viewer to maintain against the expectation that it behave
  like AutoCAD.
- Adoption. The tool fit an existing workflow instead of asking anyone to change one,
  which is the single biggest reason it got used.
- Vendor independence via an interchange format.
- One PDF path for the signature workflow, one DXF path for the technical workflow — the
  two audiences get the artefact each actually needs.

**What it cost**

- **No control over presentation.** How the drawing looks depends on the recipient's CAD
  settings — layers, colours, line weights render differently in different setups.
- **DXF is a large and quirky specification**, and `ezdxf` is now a dependency in the
  critical output path.
- **No interactive exploration in the tool itself.** Reviewing results means opening
  another application, which is friction during the review step.
- **Round-tripping is not supported.** Edits made in CAD do not come back; the flow is
  one-way by design, and a future need for reconciliation would be a genuine redesign.

The generalisable lesson is about where a tool should stop. **Integrating into an
existing workflow beats replacing it**, especially against mature domain software. The
value added here was the analysis, not the drawing environment — and building the
drawing environment would have consumed the effort that made the analysis good.
