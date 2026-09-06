*🇰🇷 Korean version: [0003-platform-as-primary-portfolio-vehicle_kr.md](0003-platform-as-primary-portfolio-vehicle_kr.md)*

# ADR-0003: Build one Platform project as the primary portfolio vehicle, deliberately spanning all three target role families

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

`sun-moon-python-platform`'s ADR-0009 records a *technical* reason for
preferring a Platform shape (Core Runtime/Agent Runtime/Domain SDK/
Transport Layer/Event Bus/Dashboard/CLI) over a plain Framework: it's a
better fit for what the project is actually becoming. Separately, given
ADR-0002's baseline (no existing portfolio, targeting three different role
families — Backend/Platform Engineering, AI/ML Engineering, DevOps/SRE —
with no preference between them), a career-strategy question needed
answering too: build one thing that spans all three, or three separate,
smaller, role-specific projects?

## Decision

Build **one Platform**, not three separate portfolio projects, and treat
its component breakdown as deliberately mapping onto the three target role
families:

- **Core Runtime / Transport Layer / Event Bus / CLI** (Go) → Backend/Platform Engineering signal
- **Agent Runtime / Domain SDK** (Python) → AI/ML Engineering signal
- **Dashboard** (TypeScript), plus Docker/CI/deployment work across all
  components → DevOps/SRE signal

A single coherent Platform, built by one person, that credibly spans all
three areas is a stronger portfolio story than three disconnected toy
projects — it demonstrates the ability to design a system where the parts
fit together, which is itself a signal senior/platform-level roles
specifically look for. It also means work done for one role's story isn't
wasted if the final application ends up going toward a different one of
the three.

## Alternatives considered

- **Three separate smaller projects, one per role family** — lower risk
  per project (easier to finish something small), and lets each project be
  laser-targeted at one role's expectations. Rejected for now because (a)
  three finished small projects read as "dabbling," where one integrated
  system reads as "can architect," and (b) ADR-0002's timeline (~3.5
  months, no existing portfolio) doesn't comfortably fit three
  independent finish lines — likely to end with three unfinished projects
  instead of one finished one.
- **Pick just one role family and go deep** — lowest risk, clearest focus,
  and probably the fastest path to a polished single artifact. Rejected
  because the owner explicitly wants to keep options open across all three
  (ADR-0002) — this stays a live alternative to fall back to if the
  timeline gets tight (see `ROADMAP.md`'s scope-cutting notes).

## Consequences

- Raises execution risk: an unfinished multi-language Platform is a weaker
  portfolio than one finished single-language project. `ROADMAP.md` needs
  explicit, ruthless prioritization and scope-cutting checkpoints to manage this.
- Every component added should be justifiable against a specific role
  family's expectations, not added just because it's technically
  interesting — otherwise this drifts back into "Framework built for its
  own sake" rather than a deliberate portfolio artifact.
- Ties this repo's roadmap directly to `sun-moon-python-platform`'s
  technical roadmap — a change to one likely needs a check against the
  other (e.g., if that repo's ADR-0009 open questions resolve toward "3
  separate Go services" instead of "1 Go process," this ADR's timeline
  assumptions should be revisited too).

## References

- `sun-moon-python-platform/docs/adr/0009-pivot-framework-to-platform.md` —
  the technical-side reasoning this career-side decision builds on.
- ADR-0002 (this repo) — the baseline this decision is made against.
