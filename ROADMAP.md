*🇰🇷 Korean version: [ROADMAP_kr.md](ROADMAP_kr.md)*

# Roadmap — draft, revised additively

This is a first draft, not a commitment set in stone. Per this project's
working style (build a piece, reflect, add to it — see
`sun-moon-python-platform`'s ADR referencing this), expect this document to
change as phases complete and priorities shift. When a phase's scope
changes materially, write an ADR about *why* rather than silently editing
this file's history away.

**Baseline this is built on**: ADR-0002 (C/C++/C#/Java + server-comms/POS
background, no cloud-native/AI/ML experience yet, ~3.5 months from
2026-09-06 to a December 2026 target, market-agnostic, no existing
portfolio). **Strategy this follows**: ADR-0003 (one Platform, deliberately
spanning Backend/Platform, AI/ML, and DevOps/SRE signal).

## Phase 0 — done (as of 2026-09-06)

`sun-moon-python-platform`: a working, tested, end-to-end-verified Python
monorepo — `order-service` + `ai-agent-service`, DDD-structured, async
HTTP/TCP, Redis pub/sub between services, 9 passing tests, ADR-0000 through
0009 documenting the reasoning. This already stands on its own as partial
evidence for **Backend Engineering** and **AI/ML Engineering** (the AI
Agent loop exists and works, currently against a deterministic offline
provider — see Phase 1).

## Phase 1 — deepen the Python side (target: ~6 weeks, through late October)

Goal: turn "the plumbing works" into "this is a real, defensible piece of
engineering," without adding a new language yet.

- Wire up the real Anthropic provider (currently `echo` only) — needs an
  API key and an actual verification pass, not just code that's never been
  run against a live model.
- Add Alembic migrations to `order-service` (currently `create_all` on
  startup — noted as a gap in that repo's README).
- Add a second bounded-context service (e.g. `delivery-service`) to prove
  the DDD/monorepo pattern generalizes past one example — this was already
  flagged as unproven in that repo's ADR-0003/ADR-0006.
- Basic CI (GitHub Actions): run both services' pytest suites on every
  push. A green check-badge in the README is disproportionately persuasive
  to a reviewer skimming a portfolio repo.

**Why this comes first**: it's the lowest-risk, highest-leverage phase —
no new language to learn, and it converts an already-built system from
"demo" to "defensible." Skipping straight to Go/TypeScript with a shaky
Python foundation would be a mistake.

## Phase 2 — one Go component (target: ~4 weeks, through late November)

Goal: prove polyglot Platform capability with **one** well-built Go piece,
not three half-built ones — and, per ADR-0004, deliberately use it to
resolve a specific real pain point (completion-callback-fragmented domain
logic from past IOCP-based C++ work) rather than just "trying Go."

- Optional 1-2 day warm-up first: revisit C#'s `async`/`await` (already
  known) purely to re-confirm the IOCP → linear-async-code lesson from
  ADR-0004 before tackling a genuinely new language — low risk, fast confidence check.
- Candidate: a Go-based Gateway/Transport service in front of
  `order-service`/`ai-agent-service` — conceptually similar to the
  "combined-server" idea from the original C project, and a direct bridge
  from the owner's strongest existing skill (C/C++ systems programming —
  Go's mental model is close) into a new, in-demand language. Its ADR (in
  `sun-moon-python-platform`) should explicitly compare goroutines/channels
  against the completion-callback style from the department-store system —
  see ADR-0004.
- Rust deprioritized behind Go for now (ADR-0004) — Go's CSP model is the
  higher-contrast lesson to prove out first; Rust's async/await can follow later.
- Scope deliberately small: one Go binary, one clear job (route/proxy
  requests, maybe collect basic metrics), well-tested, with its own ADR in
  `sun-moon-python-platform` explaining the choice.
- This is where `sun-moon-python-platform`'s ADR-0009 open question #1
  (separate Go services vs. one Go process with internal modules) gets its
  first real, concrete answer — through building, not just discussing.

**Why one component, not the full Core Runtime/Transport/Event Bus
split**: ADR-0003 already flags execution risk from spanning too much.
One finished, well-documented Go service is a stronger interview artifact
than three unfinished ones.

## Phase 3 — DevOps polish (target: ~2-3 weeks, through mid-December)

Goal: make the **AI/ML + Backend** work in Phases 1-2 look and run like a
real, operable system — this is where the DevOps/SRE signal mostly comes from.

- Verify `docker-compose.yml` actually builds and runs (flagged as
  untested in that repo — this environment had no Docker to test with).
- A real CI pipeline covering the Go component too (lint + build + test).
- A live deployment somewhere free/cheap (Fly.io, Render, a small VPS) —
  a working URL beats a thousand words in an interview.

## Phase 4 — optional / stretch (only if time remains)

- A minimal TypeScript Dashboard hitting `/health` and recent
  orders/agent notes. Explicitly the lowest-priority component — fine to
  defer past the Q4 target, or past the transition window entirely,
  without treating it as a failure. Not every box in the original Platform
  diagram needs to be checked before this is a legitimate, presentable portfolio.

## Explicit scope-cutting rule

If time runs short, cut from the bottom of this list (Phase 4, then late
Phase 3) before cutting depth from Phase 1/2 — a smaller, finished,
well-documented system beats a sprawling, half-finished one in every
interview conversation this is likely to come up in.

## Not yet decided

- Whether "Q4 2026" means "portfolio must be interview-ready before the
  transition" or "keep building immediately after it" (ADR-0002 flagged
  this as open) — affects how aggressively Phase 3/4 get compressed.
- GitHub visibility for this repo and for `sun-moon-python-platform` going
  forward, given this repo's timeline/company-departure content is more
  sensitive than pure architecture reasoning.
