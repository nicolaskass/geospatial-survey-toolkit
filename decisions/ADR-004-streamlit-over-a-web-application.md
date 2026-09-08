# ADR-004 — Streamlit over a web application

**Date:** 2026-01 · **Status:** Accepted

## Context

The toolkit needed a user interface: file upload, a guided multi-step flow, tables of
alerts, an interactive map, charts, and downloads.

My default stack for this is React with a REST API, and it is what the other systems I
build use. The honest question was whether it was right here.

The constraints that made it not right:

- **Four users**, inside one firm, using it occasionally rather than daily.
- **Local execution**, on the user's own machine, no server
  ([ADR-005](ADR-005-docker-as-a-windows-installer.md)).
- **The value is entirely in the processing.** Geometry, validation, DXF and PDF
  generation are the product. The interface is a way to reach them.
- **One developer**, and the alternative to shipping something was the firm continuing to
  do this by hand in spreadsheets for months longer.

A React frontend plus an API would have meant two codebases, a serialisation layer
between Python geometry objects and JSON, a build step, and a deployment story — to give
four occasional users a nicer-looking wizard.

## Decision

**Streamlit**, running locally, with a multi-page suite: a home page and four
applications sharing one sidebar.

The processing stays strictly separate from the interface — geometry and analysis in
their own modules, chart and PDF generation in another, with the Streamlit layer only
orchestrating. That separation is what keeps the decision reversible: replacing the
interface later does not touch the part that matters.

## Consequences

**What was gained**

- One language, one codebase, no serialisation boundary. Geometry objects pass directly
  from analysis to display.
- The tool shipped in weeks rather than months, which is the difference between it
  existing and not.
- The interactive map, dataframes, file upload and download widgets are one line each.
- The multi-page pattern gave four applications a shared home for free.
- Local execution is trivial: no server, no build.

**What it cost, and it is not small**

- **The rerun execution model.** Streamlit re-runs the whole script on every interaction,
  so state lives in `session_state` and expensive work must be explicitly guarded. This
  is a real constraint that shapes the code, and it is the main source of awkwardness in
  the wizard.
- **Limited layout control.** The interface looks like a Streamlit app. Fine internally,
  not something to put in front of a client.
- **No component reuse** with my other frontends.
- **Concurrency is not really a thing.** Correct for a local single-user tool, disqualifying
  the moment it needs to be shared.
- **A ceiling that is genuinely close.** Any significant increase in interaction
  complexity means fighting the framework rather than using it.

**When this stops being right:** if the tool goes multi-user, is exposed to clients, or
needs interaction beyond a linear wizard, Streamlit should be replaced rather than
extended. Because the processing is already isolated, that replacement is an interface
rewrite and not a rebuild — which is the property that makes accepting the ceiling
reasonable rather than reckless.

The general point: **the right stack is a function of the audience, the lifespan and the
developer count, not of what is technically best in the abstract.** Choosing the heavier
stack here would have been a worse engineering decision made for a better-sounding reason.
