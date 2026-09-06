*🇰🇷 Korean version: [0001-use-adrs-for-career-decisions_kr.md](0001-use-adrs-for-career-decisions_kr.md)*

# ADR-0001: Document career-transition decisions as ADRs, the same way `sun-moon-python-platform` documents architecture decisions

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

The owner is preparing for a job transition and building a technical
Platform project (`sun-moon-python-platform`, born out of the `Network`
work) as the primary vehicle for that transition. Two kinds of decisions
are happening at once: technical ones (what to build, which language for
which component) and career ones (which skills to prioritize, which roles
to target, how much time to spend where). The technical side already has a
documentation practice — ADRs in `sun-moon-python-platform/docs/adr/`. The
career side had none, and the reasoning behind career choices is exactly
the kind of thing that's easy to lose track of or re-litigate under
pressure (e.g., right before an interview, six months from now, wondering
"why did I choose Go over Rust for this").

## Decision

Use the same ADR format (Context/Decision/Alternatives
considered/Consequences/References — see `0000-template.md`) for
career/portfolio-strategy decisions in this repository, kept separate from
`sun-moon-python-platform`'s technical ADRs but explicitly
cross-referencing them where relevant. This repository is not a codebase —
it exists to hold the reasoning behind the career strategy, the same way
that repository holds the reasoning behind the system's architecture.

One addition to the format not used in the technical ADRs: a `Status` value
of **Ongoing** for decisions that are deliberately kept open and revisited
over time rather than resolved to a single answer at one point — this
matches how the owner already prefers to treat some technical direction
questions (see `sun-moon-python-platform`'s ADR-0009 and the "direction-setting,
not confirmed-vs-exploratory" framing established there).

## Alternatives considered

- **One running notes document instead of numbered ADRs** — simpler to
  start, but as ADR-0001 in the other repo already noted, a single document
  makes it hard to tell "the reasoning that's still valid" from "the
  reasoning that's since been superseded." Career direction is likely to
  shift at least as much as technical direction over a multi-month
  timeline, so the same problem applies here, possibly worse.
- **No documentation at all, just build and see** — the owner explicitly
  asked for this process to be documented, so this wasn't a live option;
  noted here anyway because it's a legitimate choice for someone who
  prefers to move fast and reflect only at natural checkpoints.

## Consequences

- Every ADR here doubles as raw material for later interview answers and
  resume/portfolio write-ups — "why did you build it this way" questions
  get answered by pointing at a dated, honest record instead of
  reconstructing the reasoning from memory under interview pressure.
- Adds writing overhead on top of an already tight timeline (see ADR-0002)
  — worth watching that this doesn't become a way to feel productive
  without actually building anything. A light rule of thumb: write the ADR
  when a decision actually changes direction, not for every small task.
- This repo will likely contain personal/timeline-sensitive information
  (see ADR-0002) — its GitHub visibility needs a deliberate decision, not a
  default (tracked separately, not yet made as of this ADR).

## References

- `sun-moon-python-platform/docs/adr/0001-use-adrs-for-decisions.md` — the
  ADR practice this one is modeled on.
- Michael Nygard, *Documenting Architecture Decisions* (2011) — same
  original source cited there; the format generalizes beyond software
  architecture to any decision worth a dated rationale.
