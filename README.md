*🇰🇷 Korean version: [README_kr.md](README_kr.md)*

# Alignment — a career-transition preparation log

- This repository isn't just a set of study notes — it records the decisions
  made during career-transition preparation and the reasoning behind them,
  the same way system architecture decisions get recorded, as ADRs
  (Architecture Decision Records).

## Why keep this

- Preparing for a career transition surfaces two broad kinds of decisions:

  - Technical: what to build, which technologies and languages to use.
  - Career: which skills to prioritize, which roles to target, how to invest time.
- Technical decisions get recorded as ADRs in each technical repository; this
  repository exists to record career decisions and the reasoning behind them.

In the end, this repository is where the technical-growth and
career-growth processes are organized and tracked together.

## Relationship to the technical repositories

- Each technical repository is a system — code, tests, and the technical
  decisions around them, managed as its own ADRs.
  This repository is the direction behind building those systems — why each
  system was chosen, why these particular technologies are being learned, why
  this role is the target at this point in time.
  The two kinds of repositories are kept separate because they serve different
  purposes: the technical repositories explain the engineering output, and
  this one explains the career strategy and the direction behind it.

- Whenever a single decision affects both technology and career, the ADRs on
  each side cross-reference each other.

Technical repositories this direction currently points to:

- [`sun-moon-python-platform`](https://github.com/schware/sun-moon-python-platform)
- [`sun-moon-c-server`](https://github.com/schware/sun-moon-c-server)

Other parallel tracks (e.g. the mobile app, and the ongoing
[`sun-moon-java-platform`](https://github.com/schware/sun-moon-java-platform)
modernization — maintaining the pre-existing Java/Spring Boot baseline
rather than building new portfolio evidence) are tracked in `ROADMAP.md`.

## Structure

```
docs/adr/       ADRs — one per career/portfolio-strategy decision
ROADMAP.md      A living, phase-by-phase plan (revised as phases complete)
```

Start with `docs/adr/0001-use-adrs-for-career-decisions.md` for why this
exists at all, then `0002` (the honest starting baseline) and `0003` (the
strategy built on it), then `ROADMAP.md` for the concrete plan.

## A note on what's in here

- This repo may contain personal, time-sensitive information — an honest
  assessment of current skills, learning plans, a target timeline for a
  career transition, and similar.

Its GitHub visibility should therefore be a deliberate choice rather than
whatever the default happens to be; that decision is tracked in
`ROADMAP.md`'s "Not yet decided" section.
