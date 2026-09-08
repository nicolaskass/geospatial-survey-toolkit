![Geospatial Survey Toolkit — Architecture and Decision Record. Python, Shapely, Streamlit, DXF, potrace. Field data to CAD, in production with surveyors.](assets/banner.png)

# Geospatial Survey Toolkit — Architecture & Decision Record

Field-data tooling for a land-surveying and civil-engineering practice: road-works
quality control, survey-code validation, and a raster-to-CAD vectoriser. Built for
surveyors who are not developers, running on their own laptops, in the field.

**This repository contains no source code.** It documents how the tools are built and
why. The interesting part is not the Python — it is the decisions a data problem forces
when the output feeds a legal document and the user is holding a total station.

---

## The problem

A surveyor walks a road under construction and records points with a GNSS receiver or a
total station. Each point carries a code encoding which contractor laid it, which layer
it belongs to, and where in the cross-section it sits. A day's work is a CSV with
thousands of rows.

Large road works are executed by **more than one contractor at once**, each responsible
for a different stretch or a different layer, and a central purpose of the survey is to
establish exactly where one company's scope ends and the next one's begins. The contractor
is therefore part of the point's identity, not metadata about it.

Somebody then has to answer: is the layer thickness within tolerance, is the slope within
spec, does one contractor's work leave a gap against the next, and can we hand the client
a signed report and a CAD drawing tomorrow morning.

That was being done by hand, in spreadsheets and AutoCAD, and it took days.

Three constraints shaped everything:

1. **The data is dirty and the errors are meaningful.** A miscoded point may be a typo,
   or it may be the one real defect in the job. Software that silently normalises field
   data destroys evidence.
2. **The output is a deliverable, not a screen.** It ends up in a CAD file a civil
   engineer opens and a PDF somebody signs. Both formats are non-negotiable.
3. **The users are surveyors on Windows laptops.** No Python, no terminal, no admin
   rights, sometimes no internet. A tool that needs any of those is a tool that does not
   get used.

## Scale

Measured from the repository.

| | |
|---|---|
| Processing and apps | ~11,100 LOC Python |
| Vectoriser service | ~715 LOC Node.js |
| Field applications | 4, in one Streamlit suite |
| Validation engine | 9 alert types across 3 severity levels, ~1,230 LOC |
| Geometry | Shapely — projection, chainage, polygon generation |
| Outputs | DXF (ezdxf), signed PDF reports (ReportLab), CSV, PNG, SVG |
| Delivery | Docker, wrapped in Windows batch launchers |

## What's in this repository

| File | Contents |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) | The pipelines, the geometry, the data model, delivery topology |
| [ENGINEERING.md](ENGINEERING.md) | Shipping Python to non-technical Windows users, validation strategy, accessibility |
| [decisions/](decisions/) | Seven architecture decision records |

---

## Seven things worth a look

**The tool advises; the surveyor decides.**
The validator's own docstring states it: *it detects alerts and problems — the user
decides whether they are errors.* Nine typed alert kinds across three severity levels,
and not one of them auto-corrects anything. This is the central decision of the whole
toolkit, and it is the opposite of what most data-cleaning code does.
→ [ADR-001](decisions/ADR-001-the-tool-advises-the-surveyor-decides.md)

**Chainage is computed, not entered.**
Station along a road is derived by projecting each surveyed point onto the road axis as
a Shapely `LineString`. One `project()` call replaces a column people used to fill in by
hand and get wrong. The geometry library is doing the work the domain actually needs.
→ [ADR-002](decisions/ADR-002-chainage-by-projection.md)

**A colour palette chosen for protanopia, not for looks.**
The interactive map's layer colours are picked to remain distinguishable under red-green
colour deficiency, which affects roughly 8% of men — a meaningful slice of a field
workforce. It is documented in the app's own help text. Accessibility in an internal
industrial tool, where nobody would have complained, is vanishingly rare.
→ [ADR-007](decisions/ADR-007-palette-for-protanopia.md)

**Raster to CAD, end to end.**
Scanned plans become editable drawings through an explicit pipeline: PDF → ImageMagick →
raster preprocessing with sharp → potrace vectorisation → SVG parsed with xml2js → DXF
emitted for CAD. Each stage is a separate module with a single job, so a failure names
its own stage instead of producing a mysteriously empty drawing.
→ [ADR-006](decisions/ADR-006-raster-to-cad-pipeline.md)

**DXF is the contract.**
Output targets the format the client's civil engineer already opens, rather than a
viewer of my own. The toolkit ends where the industry's tools begin — which is why it
got adopted instead of admired.
→ [ADR-003](decisions/ADR-003-dxf-as-the-delivery-contract.md)

**Docker as a Windows installer.**
Python tooling reaches non-technical field users through Docker, wrapped in plain
double-clickable batch files — start, stop, restart, update, and a script that installs
Docker itself. There is also a documented no-Docker fallback for the machine where IT
says no. Choosing the deployment story before the framework is what made adoption
possible.
→ [ADR-005](decisions/ADR-005-docker-as-a-windows-installer.md)

**Streamlit, deliberately, and with an exit plan.**
Chosen because a four-step guided wizard for four users is not worth a React frontend
and a REST API, and because the alternative was the tool never shipping. The decision
record states what it costs and the conditions under which it stops being right.
→ [ADR-004](decisions/ADR-004-streamlit-over-a-web-application.md)

---

## On AI-assisted development

Built with [Claude Code](https://claude.com/claude-code) under the same
version-controlled agent setup described in my
[other architecture repository](https://github.com/nicolaskass/specialty-retail-erp):
role-scoped agent definitions, an explicit model-cost tier per role, and a blackboard
coordination protocol.

The domain reasoning here is mine and had to be: what counts as a station, when two
points belong to the same cross-section, which discrepancies are alerts and which are
merely informative. Those are judgements about how a road is actually built, and getting
them wrong produces software that is confidently incorrect — the most expensive kind in a
context where the output gets signed.

---

## Author

Nicolás Kass — biologist, ISO 9001 consultant, and software architect, in that
historical order. I build operational and analytical systems for small businesses at
[T³](https://t3.com.ar).

Client identifiers, site data and deployment hostnames are omitted throughout.

**Licence:** the writing in this repository is published under
[CC BY 4.0](LICENSE) — reuse it, quote it, build on it, with attribution.

*Every figure above is a real measurement taken from the project — lines counted from the
source tree, alert types and applications counted from the modules.*
