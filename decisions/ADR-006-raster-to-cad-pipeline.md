# ADR-006 — Compose existing tools for raster-to-CAD

**Date:** 2026-06 · **Status:** Accepted

## Context

Surveying practices hold decades of plans as paper scans and PDFs. Reusing one means
re-drawing it in CAD by hand — hours of skilled work to recover geometry that is already
there, just in the wrong representation.

The task is raster-to-vector conversion followed by CAD emission. Three ways to get there:

1. **Write the vectoriser.** Edge detection, thinning, path tracing, curve fitting. It is
   a genuinely interesting problem and a well-studied one.
2. **Pay for a commercial converter.** They exist, they work, and they are licensed
   per seat and per machine.
3. **Compose existing open-source tools**, each already best-in-class at its stage.

Option 1 is the trap for someone who enjoys algorithms. potrace has been refined since
1999. A first implementation would be months of work to produce something measurably
worse, and the value being delivered here is not tracing quality — it is the *end-to-end
path* from a PDF somebody found in a drawer to geometry in a CAD file.

## Decision

**Compose, and own the wiring.**

```
PDF/image → ImageMagick → PNG → sharp → potrace → SVG → xml2js → DXF
```

| Stage | Tool | Job |
|---|---|---|
| Rasterise | ImageMagick | PDF to PNG, 150 dpi, trimmed, first page |
| Preprocess | sharp | prepare the raster for tracing |
| Vectorise | potrace | raster to SVG paths |
| Parse | xml2js | SVG to geometry structures |
| Emit | custom | geometry to DXF |

The engineering is in the wiring, and three rules govern it:

**Each stage is its own module.** Conversion, vectorisation and DXF emission are separate
files with a single responsibility each. When a plan fails, the error names the stage
that failed.

**Verify the output, not the exit code.** External binaries fail by producing nothing far
more often than by returning an error status. Every stage checks its output file exists
before continuing, and raises a message naming its own stage when it does not.

**Jobs are identified and isolated.** Every upload gets an id and its own filenames, so
concurrent jobs cannot collide over a shared temporary path — a bug that is invisible in
testing and constant in production.

Command output buffers are capped, because a pathological input that produces unbounded
output should fail rather than exhaust memory.

## Consequences

**What was gained**

- Tracing quality equal to tools refined over decades, for none of the effort.
- Weeks instead of months, and effort spent on the part nobody else had solved — the
  end-to-end path.
- Failures are attributable to a stage, which makes support tractable.
- Stages are independently replaceable: a better vectoriser is one module swap.

**What it cost**

- **External binary dependencies.** ImageMagick and potrace must exist on the host, at
  compatible versions. This is a real operational constraint and a reason the service is
  containerised.
- **Shelling out to commands** brings escaping, buffering and platform-difference
  concerns that an in-process library would not have.
- **Little control over tracing behaviour.** potrace's parameters are the parameters
  available; a plan it traces badly cannot be fixed from this side.
- **Quality depends heavily on scan quality**, and the pipeline cannot rescue a bad
  original. There is no automatic assessment of how well a trace went, which is the most
  useful missing feature.
- **Five stages is five things that can break**, and the pipeline is only as reliable as
  its least reliable stage.

The generalisable judgement: **the interesting problem and the valuable problem are
usually different problems.** Writing a vectoriser was interesting. Delivering a path
from a drawer full of paper to editable CAD geometry was valuable. Recognising which one
you are being paid for is most of the job.
