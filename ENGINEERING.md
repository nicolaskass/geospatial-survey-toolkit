# Engineering practices

How these tools are built, shipped and kept working, and where the practices here differ
from a server-side system — because the constraints are genuinely different.

## Contents

- [Shipping Python to people who do not have Python](#shipping-python-to-people-who-do-not-have-python)
- [Accessibility in an internal tool](#accessibility-in-an-internal-tool)
- [Validation strategy](#validation-strategy)
- [Working with external binaries](#working-with-external-binaries)
- [Testing, and an honest account of it](#testing-and-an-honest-account-of-it)
- [Documentation aimed at the actual reader](#documentation-aimed-at-the-actual-reader)
- [AI-assisted development](#ai-assisted-development)

---

## Shipping Python to people who do not have Python

The users are surveyors on Windows laptops. No Python, no terminal, frequently no admin
rights, and sometimes no internet on site. Every technically elegant delivery mechanism —
`pip install`, a virtualenv, a cloud deployment — fails at least one of those.

What shipped instead:

| Launcher | Purpose |
|---|---|
| start | bring the suite up and open the browser |
| stop | shut it down cleanly |
| restart | the honest answer to most support calls |
| update | pull the current version and restart |
| install Docker | for a machine that does not have it yet |
| run without Docker | documented fallback for locked-down machines |

All of them are double-clickable batch files. The user never sees a command line.

Two decisions inside that are worth naming.

**The fallback path exists and is documented.** Corporate IT sometimes will not allow
Docker, and a delivery story with a single path fails completely at that machine rather
than degrading. Same instinct as designing a graceful fallback in a server system: the
dependency you cannot control needs an alternative before you need it.

**A restart script is a deliberate feature, not laziness.** For a four-user internal tool
with no operations staff, a reliable "turn it off and on again" that the user can run
themselves resolves most incidents at 9pm without a phone call. Building the recovery
path the user can operate is cheaper than being the recovery path.

The application runs locally rather than on a server: field data never leaves the firm
that owns it, the tool works with no connectivity, and four users cost nothing to host.

## Accessibility in an internal tool

The interactive map's layer colours are chosen to stay distinguishable under **protanopia**
— red-green colour deficiency, affecting roughly 8% of men, which in a field crew is not a
hypothetical. The choice is stated in the app's own help text, so a user can tell it was
deliberate rather than accidental.

This is worth calling out precisely because nothing forced it. There is no compliance
requirement on an internal tool, no procurement checklist, and no user who would have
filed a complaint — they would simply have found the map hard to read and quietly trusted
it less. Accessibility work that survives without external pressure is rare, and a map
whose whole function is distinguishing categories by colour is exactly where it counts.

## Validation strategy

The rule the entire toolkit is built on: **never silently modify field data.**

Field measurements are evidence. They end up in documents somebody signs, and in disputes
about who laid which layer and whether it met spec. Software that quietly normalises a
miscoded point may be erasing the one real finding in the job.

So the validator produces typed alerts and stops:

- **Three severities**, worded to avoid false confidence — even the most severe reads as
  *very probably* an error.
- **Nine alert types** as enum values, not strings, so the interface can group and count
  them and so no new alert can exist without a declared severity.
- **Normalisation is itself an alert type.** When the system does adjust something, that
  adjustment is surfaced as a reviewable event rather than hidden.
- **The human decides.** A dedicated review step precedes any output.

The consequence is a slower workflow than full automation, and that is the trade being
made on purpose. See [ADR-001](decisions/ADR-001-the-tool-advises-the-surveyor-decides.md).

## Working with external binaries

The vectoriser shells out to ImageMagick and potrace. Calling external binaries is where
pipelines rot, so three rules:

1. **Verify the output, not the exit code.** External tools fail by producing nothing far
   more often than by returning an error. Every stage checks that its output file exists
   before continuing.
2. **Wrap each stage's failure in a message naming that stage.** "Vectorisation with
   potrace failed" is actionable; a stack trace from a downstream parser is not.
3. **Cap the buffers and isolate the jobs.** Command output is bounded, and every job gets
   a unique id and its own filenames so concurrent runs cannot collide.

## Testing, and an honest account of it

Test coverage here is **thinner than in my server-side work**, and it is worth being
straight about that rather than presenting a uniform story.

What exists: a test module for the correction logic — the piece where an error would
propagate silently into a deliverable.

Why it stops there: the bulk of the remaining code is Streamlit interface and chart or PDF
composition, where the failure mode is visible immediately to the person clicking through
a four-step wizard, and where a test asserts mostly that the code still does what it does.

**What it costs, honestly.** The geometry — projection, cross-section grouping, polygon
generation — deserves property-based tests and does not have them. It is the code most
worth testing in the whole toolkit: a subtle chainage error would not look wrong on a
chart, and it would reach a signed document. That is the first thing I would add, and the
correct criticism to make of this repository.

## Documentation aimed at the actual reader

Documentation is written for the person who will read it, which here is not a developer:
quick-start, how to run it, Windows-specific setup notes, a Docker guide, and a current
project-state document. Plain instructions, screenshots of the actual buttons, and no
assumed vocabulary.

An internal tool's documentation competes with sending a WhatsApp message asking how to
do it. If reading is slower than asking, nobody reads.

## AI-assisted development

Built with [Claude Code](https://claude.com/claude-code) under the version-controlled
agent setup described in the
[ERP repository](https://github.com/nicolaskass/specialty-retail-erp#on-ai-assisted-development):
role-scoped agents, an explicit model-cost tier per role, blackboard coordination.

The division of labour was sharper here than on a web system. Pipeline wiring, chart
composition, PDF layout and the Windows launcher scripts are pattern work. The domain
model was not: what constitutes a station, when two points belong to the same
cross-section, which discrepancies are alerts and which are noise. Those are claims about
how a road is built, they cannot be derived from the data alone, and getting one wrong
produces software that is confidently incorrect — the worst possible outcome when the
output is a document somebody signs.
