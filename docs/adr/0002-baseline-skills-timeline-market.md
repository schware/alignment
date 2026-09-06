*🇰🇷 Korean version: [0002-baseline-skills-timeline-market_kr.md](0002-baseline-skills-timeline-market_kr.md)*

# ADR-0002: Record the starting baseline — current skills, timeline, target market, existing portfolio

- **Status**: Accepted (this is a snapshot, not a strategy — see ADR-0003 for the strategy built on it)
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

Every plan that follows (ADR-0003 onward, `ROADMAP.md`) depends on an
honest starting point. Rather than let that baseline live only in this
conversation's transcript, it's recorded here once, plainly, so future
ADRs can reference a fixed point instead of restating it — and so it's
easy to look back later and measure actual movement against it.

## Decision

As of 2026-09-06, the baseline is:

- **Existing language/domain experience**: C, C++, C#, Java. Primary work
  has been server communication and POS (point-of-sale) client
  development — meaning real, hands-on experience with network protocols,
  client-server architecture, and production client software, but not with
  cloud-native tooling, container orchestration, or AI/ML.
- **What today's work already adds on top of that baseline**: a working,
  tested, ADR-documented Python monorepo (`sun-moon-python-platform`) with
  DDD-structured services, async I/O (HTTP/TCP/Redis), and a functioning
  (if not yet LLM-backed) AI Agent tool-calling loop — built today, for the
  first time, with Claude. This is new territory, not yet a demonstrated
  independent skill.
- **Timeline**: targeting a career transition around **Q4 2026** —
  roughly 3.5 months from this ADR's date. Not yet decided (see
  ADR-0003/ROADMAP) whether the target is "portfolio interview-ready before
  the transition" or "keep building immediately after it" — this ADR only
  fixes the endpoint, not what must be true by it.
- **Target market**: no constraint — domestic (Korea) or international/remote,
  startup or large company, all open.
- **Existing portfolio**: none. `sun-moon-python-platform` (and whatever
  else gets built following this plan) is effectively the entire portfolio
  being built from scratch for this transition.

## Alternatives considered

Not applicable in the usual sense — this ADR records a fact pattern rather
than choosing between options. Recorded as an ADR anyway (rather than left
implicit) because every later strategic ADR depends on it being explicit
and dated, not assumed.

## Consequences

- Every roadmap item should be checked against "does this build on the
  C/C++/C#/Java + network-protocol strength, or does it require learning
  something with zero prior exposure" — both are fine to include, but the
  mix matters for realistic time-boxing given ~3.5 months and (presumably)
  still-full-time-employed availability.
- No existing portfolio means there's no "just polish what's already
  there" shortcut — everything in the final portfolio narrative gets built
  in this window.
- Because target market is unconstrained, the roadmap shouldn't over-index
  on one region/company-size's specific hiring quirks; general engineering
  fundamentals and clear documentation (which travel well across markets)
  should be weighted over market-specific trend-chasing.

## References

- This conversation, 2026-09-06 — direct source for all of the above.
