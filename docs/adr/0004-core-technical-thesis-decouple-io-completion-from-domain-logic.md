*🇰🇷 Korean version: [0004-core-technical-thesis-decouple-io-completion-from-domain-logic_kr.md](0004-core-technical-thesis-decouple-io-completion-from-domain-logic_kr.md)*

# ADR-0004: Core technical thesis — decouple completion-based I/O from domain logic (the IOCP lesson)

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

The owner's clearest source of dissatisfaction with their own past C/C++
work (a multi-tenant server letting several department stores/malls
connect) wasn't lack of skill with async I/O — they used IOCP
(I/O Completion Ports), a genuinely correct, high-performance choice for
that problem. The dissatisfaction was structural: business logic ended up
fragmented across separate completion-callback functions
(connect/read-complete/write-complete/disconnect-style handlers), which
made proper object decomposition hard and pushed too much code into one
oversized source file. The performance benefit IOCP gave (precise
knowledge of when I/O completes) didn't offset the structural cost of
organizing code around I/O events instead of around business concepts.

## Decision

Adopt this as the project's core technical thesis, and deliberately use it
as the throughline connecting every new language in scope
(ADR-0002/ROADMAP: Go, Rust, TypeScript, JavaScript) rather than treating
each language's concurrency model as an unrelated thing to memorize:

- **Python (already applied, 2026-09-06)**: `asyncio`'s `async`/`await`
  gives linear-reading code over the same completion-based mechanism
  (epoll on Linux). More importantly, `sun-moon-python-platform`'s
  `AggregateRoot` (e.g. `Order`) is kept deliberately ignorant of I/O —
  the interfaces/infrastructure layers own all completion/networking
  mechanics, the application layer orchestrates, and the domain layer
  holds every business rule as one cohesive object regardless of how many
  I/O round-trips it took to assemble. This is the direct architectural
  answer to "business logic fragmented across I/O handlers."
- **C# (already known)**: `async`/`await` is the same model, in a language
  already used professionally — a fast, low-risk checkpoint to confirm the
  lesson before tackling a genuinely new language.
- **Go (Phase 2 target)**: a different, arguably more elegant answer —
  goroutines + channels (CSP). Go's runtime network poller literally wraps
  IOCP on Windows / epoll on Linux; a goroutine can call a
  blocking-looking read and the runtime parks it without blocking an OS
  thread, so business logic never needs an "async-colored" function at
  all. Likely the most intuitive concurrency model for someone with
  hands-on IOCP experience, since it's mechanically the closest cousin.
- **Rust / TypeScript / JavaScript**: async/await again, each on its own
  state-machine/event-loop implementation of the same underlying idea.
- **C/C++ itself**: the direct, same-language fix to the original problem
  is C++20 coroutines (`co_await`) plus a modern async I/O library (e.g.
  Boost.Asio's coroutine support, or cppcoro) — meaning the original
  system's root cause wasn't IOCP, it was the pre-coroutine callback style
  the tooling of the time forced.

## Alternatives considered

- **Treat each language's concurrency model as unrelated trivia to
  memorize** — rejected: the owner's own diagnosis already found the
  unifying thread. Ignoring it would waste the clearest, most hard-won
  piece of intuition available for this whole plan.
- **Rewrite the old department-store C++ system with coroutines as a
  portfolio project** — considered, but that codebase is very likely
  employer-owned/proprietary and inaccessible for a public portfolio.
  Demonstrating the same lesson fresh, in the Platform's Go component
  (Phase 2), is fully valid and avoids that problem.

## Consequences

- Phase 2 (the Go gateway/transport component in `sun-moon-python-platform`)
  now has a sharper justification than "prove polyglot capability" — it's
  the deliberate, direct rebuttal to a specific real professional
  frustration, which reads as a far more memorable story in an interview
  than a generic "I also know Go."
- That Go component's own ADR (to be written in `sun-moon-python-platform`
  when Phase 2 starts) should include a short comparison note — goroutines
  vs. the completion-callback style used in the original department-store
  system — turning a private frustration into demonstrable growth without
  needing to expose any proprietary code.
- Slightly re-weights ADR-0002's language list: revisiting C#'s
  async/await is nearly free (already known) and worth doing briefly
  before Go as a warm-up; Rust's async/await can wait, since Go's CSP
  model is the higher-contrast, higher-priority lesson to prove out first.

## References

- This conversation, 2026-09-06 — the department-store/mall IOCP system as the originating case.
- The general pattern this names: business logic fragmentation under
  callback-based / continuation-passing-style async I/O (sometimes
  informally called "callback hell" or "stack ripping" in async I/O literature).
- Tony Hoare, *Communicating Sequential Processes* (1978) — the
  theoretical root of Go's goroutine + channel model.
- `sun-moon-python-platform/docs/adr/0003-domain-driven-design-for-business-logic.md` —
  the concrete implementation of "domain logic stays ignorant of I/O" referenced above.
