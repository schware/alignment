*🇰🇷 Korean version: [README_kr.md](README_kr.md)*

# Alignment — a career-transition preparation log

This repository is not a codebase. It records the reasoning behind a
career transition, the same way [`sun-moon-python-platform`](https://github.com/schware/sun-moon-python-platform)
records the reasoning behind a system's architecture — as dated ADRs
(Architecture Decision Records) instead of a single running notes document.

## Why this exists

The owner is preparing for a career transition and is building
`sun-moon-python-platform` (a Platform project) as the primary vehicle for
that transition. Two kinds of decisions happen along the way: technical
ones (what to build, which language for which piece) and career ones
(which skills to prioritize, which roles to target, how much time goes
where). The technical decisions already get ADRs in that repository; this
repository does the same for the career decisions, and cross-references
the technical ones where they're entangled.

## Relationship to `sun-moon-python-platform`

- That repo is the *system* — code, tests, its own ADRs about architecture.
- This repo is the *person building it* — why that system, why these
  components, why this timeline, why these target roles.
- They're kept separate (not merged) so the technical repo stays legible
  as pure engineering work, and this one stays legible as career strategy
  — but ADRs on either side cross-reference the other when a decision
  spans both (see ADR-0003 here, and ADR-0009 there).

## Structure

```
docs/adr/       ADRs — one per career/portfolio-strategy decision
ROADMAP.md      A living, phase-by-phase plan (revised as phases complete)
```

Start with `docs/adr/0001-use-adrs-for-career-decisions.md` for why this
exists at all, then `0002` (the honest starting baseline) and `0003` (the
strategy built on it), then `ROADMAP.md` for the concrete plan.

## A note on what's in here

This repo intentionally contains personal, time-sensitive information
(current-skill honesty, a target timeline for a career transition). Its GitHub
visibility should be a deliberate choice, not a default — see
`ROADMAP.md`'s "Not yet decided" section.
