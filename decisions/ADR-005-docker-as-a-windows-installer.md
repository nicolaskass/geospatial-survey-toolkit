# ADR-005 — Docker as a Windows installer

**Date:** 2026-01 · **Status:** Accepted

## Context

The users are surveyors. Their laptops run Windows, have no Python, no terminal habit,
often no administrator rights, and on site frequently no internet. They are highly
skilled — at surveying.

The tool depends on a scientific Python stack — Shapely, pandas, NumPy, Matplotlib,
ReportLab, ezdxf, Streamlit — which on Windows historically means compiled dependencies
and version conflicts.

The available options all failed:

- **`pip install` into a virtualenv** — requires Python, a terminal, and troubleshooting
  compiled packages over the phone.
- **A frozen executable** (PyInstaller and similar) — brittle with scientific libraries,
  produces enormous binaries, and antivirus software quarantines unsigned executables.
- **Host it in the cloud** — needs connectivity that is absent on site, adds hosting cost
  for four users, and moves the firm's field data onto someone else's server.

Getting the software to run at all was a harder problem than writing it. That is normal
for internal tooling and routinely underestimated.

## Decision

**Docker, wrapped in double-clickable Windows batch files.** The user never sees a
command line.

| Launcher | Purpose |
|---|---|
| start | bring the suite up, open the browser |
| stop | shut down cleanly |
| restart | the honest answer to most support calls |
| update | pull the current version and restart |
| install Docker | for a machine that does not have it yet |
| run without Docker | documented fallback for locked-down machines |

Three parts of this are the actual decision, as opposed to the tool choice:

**The fallback path exists and is documented.** Corporate IT sometimes will not permit
Docker Desktop. A delivery story with one path fails totally on that machine; with a
documented second path it degrades. The same instinct as designing a graceful fallback in
a server system — the dependency you do not control needs an alternative *before* you
need it.

**The restart script is a feature.** For a four-user tool with no operations staff, a
reliable "turn it off and on again" the user can run themselves resolves most incidents
without a phone call at 9pm. Building the recovery path the user can operate is cheaper
than being the recovery path.

**Installing the dependency is part of the deliverable.** Shipping a script that installs
Docker treats "the user does not have the runtime" as the tool's problem, not the user's.

## Consequences

**What was gained**

- One environment definition works on every machine. The scientific stack is pinned
  inside the image and never fights the host.
- No `pip` troubleshooting over the phone.
- Updates are a double-click.
- Data stays on the firm's machines, and there is no hosting cost or connectivity
  requirement.
- Adoption, which was the point. A tool that is hard to launch does not get used
  regardless of how good it is.

**What it cost**

- **Docker Desktop is a heavy dependency** on a field laptop — memory, disk, and a
  licensing regime that has changed before and can change again.
- **The batch scripts are their own small codebase**, unversioned relative to the app and
  Windows-specific.
- **The fallback path is a second path to maintain**, and it drifts unless exercised.
- **Diagnosing a failure inside a container, remotely, with a non-technical user** is
  genuinely hard — the restart script exists partly because deeper diagnosis is
  impractical.
- **First run needs internet** to pull the image, which is exactly the constraint being
  worked around, just displaced to setup time.

The lesson generalises past this project: **the deployment story is part of the design,
not a step after it.** Choosing a stack before knowing how it reaches the user is how
internal tools end up technically finished and never adopted.
