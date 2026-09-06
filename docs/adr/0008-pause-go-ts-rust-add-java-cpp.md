*🇰🇷 Korean version: [0008-pause-go-ts-rust-add-java-cpp_kr.md](0008-pause-go-ts-rust-add-java-cpp_kr.md)*

# ADR-0008: Defer Go/TypeScript/Rust re-planning to next session; add Java and C++ as first-class roadmap items

- **Status**: Partially Accepted — the deferral (item 1 below) is accepted
  per explicit owner instruction; the Java/C++ items (2 and 3) are
  **Proposed**, pending owner confirmation
- **Date**: 2026-09-06
- **Deciders**: Project owner (item 1); Claude proposal awaiting owner
  confirmation (items 2-3)

## Context

In one message, the owner did two things: (a) asked to revisit Go/
TypeScript/Rust roadmap planning next session (tomorrow) rather than now,
so they can reorganize their own thinking first — **not** an indefinite
pause, just deferred one session, the same timing as the already-scheduled
Phase 1.6 (`sun-moon-c-server` refactor); and (b) pointed out that Java
and C++ — by their own account "가장 중요한 언어" (the most important
languages) — had never gotten their own roadmap treatment. Both languages
had only appeared as *background justification* for other decisions:
Java/Spring Boot in ADR-0005 (the 8-year timeline that made Batch the
priority) and C++ in ADR-0004 (the IOCP/completion-callback thesis).
Neither had a concrete "what do we actually build to demonstrate this"
item the way Go (Phase 2) and TypeScript (Phase 4) did.

The owner opened this with an apology ("변덕쟁이 모드"/"acting fickle"),
and separately corrected an early draft of this ADR that had
over-characterized the Go/TS/Rust deferral as an open-ended pause. Worth
noting explicitly: neither is something to apologize for or overstate —
reworking a plan mid-stream as understanding deepens matches this
project's own documented working style
([[user-additive-collaboration-style]]), and "revisit tomorrow" is a
concrete, scheduled point, not an indefinite hold.

## Decision

1. **(Accepted) Revisit Go/TypeScript/Rust next session.** The existing
   Phase 2 (Go) and Phase 4 (TypeScript Dashboard) entries in
   `ROADMAP.md` are marked "revisit next session" — their content stays
   as today's starting point, not deleted or judged wrong, but not to be
   treated as current/settled either until that session happens. Rust
   (never phased, only deprioritized behind Go in ADR-0004) is deferred
   the same way. This lines up with Phase 1.6, already scheduled for the
   same next session.

2. **(Proposed) Java: complete a three-way Batch comparison.** Build the
   same Job/Step/Chunk design a third time, in actual Java/Spring Boot
   (likely real Spring Batch, not a hand-rolled engine — unlike Python/C,
   Java already has the canonical framework for this). Python
   (`batch-service`) and C (`batch_runner`) already implement it; Java
   would be the **reference implementation** — not a third demo among
   equals, but the "known-good" baseline the other two get checked
   against, since Spring Batch is what the owner has 8 years of real
   production judgment in (ADR-0005). This is the most direct way to make
   the portfolio *show* that expertise rather than only asserting it in prose.

3. **(Proposed) C++: a focused C++20-coroutine demo resolving the
   original IOCP pain point directly.** ADR-0004's thesis names C++20
   coroutines (`co_await` + a library like Boost.Asio or cppcoro) as the
   same-language fix to the department-store system's completion-callback
   fragmentation — distinct from `sun-moon-c-server`'s plain-C
   `batch_runner` (no coroutines, no classes). A small, scoped demo
   showing linear-looking async code over the same IOCP mechanism, with
   clean object decomposition, would be a direct "here's the fix, in the
   actual language, to the actual problem" artifact — sharper than the Go
   CSP analogy alone.

## Alternatives considered

- **Treat Java/C++ like Go/Rust — new-language acquisition items** —
  rejected: Java and C++ aren't skills to *learn*, they're 16 years of
  existing depth to *demonstrate*. That's a different kind of roadmap
  item (showcase vs. skill-acquisition) and shouldn't be planned the same way.
- **Leave Java/C++ as background-only narrative** (their role in ADR-0004/
  ADR-0005) **and add nothing further** — this was the status quo the
  owner explicitly asked to change; rejected by direct instruction.
- **Decide Java/C++ scope unilaterally and start building now** — not
  taken. Unlike item 1 (explicitly instructed), items 2-3 are Claude's
  proposal; recommending a plan and building it immediately would skip
  the owner's confirmation this kind of scope addition deserves (per this
  project's own established pattern of proposing, then confirming, before
  writing further ADRs/code).
- **Characterize item 1 as an open-ended pause** — this was this ADR's
  own first draft, corrected by the owner mid-turn: it's a one-session
  deferral with a concrete next step (tomorrow), not a shelved topic.

## Consequences

- If items 2-3 are confirmed: `alignment`'s ADR-0006 (dual-language Batch)
  likely needs a follow-up noting the Batch comparison became three-way,
  with Java specifically as the reference rather than a peer implementation.
- A C++20-coroutine demo is new, separate work from `sun-moon-c-server`'s
  existing plain-C `batch_runner` — doesn't replace it, adds to total
  portfolio surface area. ADR-0003's scope-cutting discipline still applies.
- Not yet decided: exact scope and target timing for the Java and C++
  items relative to the reorganized Go/TS/Rust plan and the already-scheduled
  Phase 1.6 (C service-based refactor) — all next-session topics, needing
  the owner's confirmation before this ADR's Proposed items become
  Accepted and get dates in `ROADMAP.md`.
- Next session has three things on its plate already: Phase 1.6 (C
  service refactor), the reorganized Go/TypeScript/Rust plan, and
  confirming (or not) the Java/C++ proposals above — worth sequencing
  deliberately rather than tackling all three at once.

## References

- ADR-0004, ADR-0005, ADR-0006 (this repo).
- [[user-additive-collaboration-style]] (memory) — why a re-plan mid-stream
  isn't treated as a problem here.
