# ADR-001 — The tool advises, the surveyor decides

**Date:** 2026-01 · **Status:** Accepted

## Context

Field survey data arrives dirty. Codes are mistyped, points are recorded in the wrong
zone, a station is missed, the same point is taken twice, chainage jumps where the crew
skipped a section.

The obvious move is to clean it automatically. Normalise the codes, drop the duplicates,
interpolate the gap, and hand back a tidy dataset. Every data-cleaning tutorial does
this, and for most analytics work it is correct.

It is wrong here, and the reason is what the data *is*.

These measurements are **evidence**. They end up in a report somebody signs and in
disputes about which contractor laid which layer and whether the thickness met spec. A
miscoded point is sometimes a typo — and sometimes it is the single real defect in the
job, the anomaly the whole inspection exists to find. Software that silently normalises
it has destroyed the finding, and nobody will ever know it did.

The failure is invisible by construction. A cleaned dataset looks better than a dirty
one. There is no error, no warning, and no way to notice afterwards that the interesting
row is gone.

## Decision

**The system never silently modifies field data.** It detects, classifies, and presents.
A human decides.

The validator's own docstring commits to this: it detects alerts and problems, and *the
user decides whether they are errors*.

The mechanics that hold the line:

**Three severity levels, worded to avoid false confidence.** `INFO` is probably fine,
`WARNING` is worth a look, and `ERROR` reads as *very probably* an error — even the top
level does not claim certainty it cannot have.

**Nine typed alert kinds**, as enum values rather than formatted strings: unrecognised
code, contractor in the wrong zone, orphan point, incomplete code, overlap, chainage
jump, duplicate, normalisation, reference issue. Typed values mean the interface can
group, filter and count them, and a new alert type cannot be added without declaring its
severity.

**Normalisation is itself an alert type.** On the occasions the system does adjust
something, that adjustment is surfaced as a reviewable event rather than folded silently
into the data.

**A review step sits between analysis and output.** No deliverable is generated until
the surveyor has seen the alerts and decided.

## Consequences

**What was gained**

- Anomalies reach the person qualified to interpret them. The tool never overrules
  domain knowledge it does not have.
- Outputs are defensible: every accepted deviation was seen and accepted by a person.
- Trust, which is the whole ballgame for an inspection tool. A tool caught silently
  altering measurements once is never used again.
- Alerts document the dataset's condition, which is useful evidence in its own right.

**What it cost**

- **The workflow is slower than full automation**, and on a clean dataset the review step
  is pure overhead. This is the trade, made deliberately.
- **Alert fatigue is a real risk.** Too many low-value alerts and the surveyor clicks
  through without reading, which restores the original problem with extra steps. The
  severity levels are the mitigation; whether the thresholds are well tuned is a question
  only field use answers, and it needs revisiting.
- **More code than a cleaning script**, since presenting a decision costs more than
  making one.
- **It assumes a competent user.** The tool is worse than automation in the hands of
  somebody who does not understand the domain. Acceptable here — the users are surveyors
  — but it is an assumption, not a universal truth.

The generalisable rule: **automate the judgement only when being wrong is cheap.** Here
being wrong is a signed document with a defect removed from it, so the software's job
ends at the point where domain judgement begins.
