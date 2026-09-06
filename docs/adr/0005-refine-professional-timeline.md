*🇰🇷 Korean version: [0005-refine-professional-timeline_kr.md](0005-refine-professional-timeline_kr.md)*

# ADR-0005: Refine the professional experience timeline (Supersedes ADR-0002's skills baseline)

- **Status**: Accepted — Supersedes ADR-0002 (only the skills/timeline
  portion; ADR-0002's target-transition-window, market, and
  no-existing-portfolio facts still stand)
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

ADR-0002 recorded the skills baseline as a flat list (C, C++, C#, Java —
server comms + POS client), which flattened four languages/tracks into an
undifferentiated blob. Follow-up conversation clarified the actual
structure, and it turns out to matter: this isn't four disconnected
experiences, it's ~12 years of continuous, overlapping work in one coherent
domain, with a clear center of gravity in the most recent 8 years.

## Decision

Record the corrected timeline as the baseline going forward:

- **Total professional experience: ~12 years**, entirely within a
  retail/commerce technology domain (POS client software, server
  communication protocols, backend REST/Batch services for multi-store
  systems like the department-store/mall case in ADR-0004).
- **Years 1-4**: C++ server-side development, as a standalone/dedicated track.
- **Years 1-12 (the entire career so far)**: C# POS Client development —
  continuous throughout, running concurrently with both the early C++
  track and the later Java track. This is the single longest-running
  thread across the whole career.
- **Years 5-12 (the most recent 8 years)**: Java + Spring Boot — REST API
  development and Batch processing. This is the most recent and,
  by sustained depth, the primary current specialization.

This reframes the transition's narrative: not "an engineer starting a
portfolio from scratch," but **a 12-year senior engineer generalizing deep,
consistently-applied enterprise experience across new languages and
paradigms** — directly in service of ADR-0004's thesis (the same
architectural judgment transfers across languages) and ADR-0003's strategy
(one Platform spanning Backend/Platform, AI/ML, DevOps/SRE).

## Alternatives considered

Not applicable in the usual sense — like ADR-0002, this records a
corrected fact pattern rather than choosing between options. Written as a
superseding ADR (not an edit to ADR-0002) per this repo's own ADR-0001 rule.

## Consequences

- Every later ADR/README/portfolio write-up should frame this as a senior
  engineer's cross-language generalization project, not a
  starting-from-zero career change — this materially affects how a
  reviewer should read the whole repository.
- The Batch gap flagged in conversation just before this ADR is now more
  significant than it first looked: Batch work spans the *entire* most
  recent 8 years, not a side skill — it's arguably the single most
  provable, deepest specific technical asset available to demonstrate in
  a new stack. Still open (not yet decided as of this ADR): which
  language to build it in, and where it slots into `ROADMAP.md`'s phases.
- ADR-0002's blanket "no existing portfolio" is worth re-checking now that
  the full timeline is visible — 12 years may have produced other
  professional artifacts (certifications, internal tools, conference talks,
  etc.) not yet mentioned. Not assumed either way here; flagged as open.

## References

- This conversation, 2026-09-06.
- ADR-0002 (this repo) — the baseline this supersedes.
- ADR-0004 (this repo) — the department-store/mall system this timeline
  contextualizes.
