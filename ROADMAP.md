*🇰🇷 Korean version: [ROADMAP_kr.md](ROADMAP_kr.md)*

# Roadmap — a living document

This document is a draft as of now, not a fixed plan. Per this project's
working style (build a piece, reflect, add to it), expect this document to
change as each phase completes and priorities shift. When a phase's scope
changes materially, write an ADR about *why* rather than silently editing
this file's history away.

**Baseline this is built on**: ADR-0002/ADR-0005 (16 years total — MFC
years 1-4, C++ server years 5-8, C# POS Client years 5-16, Java/Spring
Boot REST+Batch years 9-16 — no cloud-native/AI/ML experience built yet,
~3.5 months from 2026-09 to a Q4 2026 target, market-agnostic, no existing
portfolio). **Strategy this follows**: ADR-0003 (one Platform, deliberately
spanning Backend/Platform, AI/ML, and DevOps/SRE signal), extended by
ADR-0004 (a technical throughline across every new language) and ADR-0006
(Batch built twice, in C and Python, against a shared contract).

## Phase 0 — done (as of 2026-09-06)

`sun-moon-python-platform`: a working, tested, end-to-end-verified Python
monorepo — `order-service` + `ai-agent-service`, DDD-structured, async
HTTP/TCP, Redis pub/sub between services, 9 passing tests, ADR-0000 through
0009 documenting the reasoning. This already stands on its own as partial
evidence for **Backend Engineering** and **AI/ML Engineering** (the AI
Agent loop exists and works, currently against a deterministic offline
provider — see Phase 1).

Also already on GitHub: `sun-moon-c-server` — the pre-existing C server
framework (libuv-based TCP/HTTP/WS/TLS, verified to 10k concurrent
connections), now the home for the C-language direction (see the new
Phase 1.5 and ADR-0006).

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
- **Done, 2026-09-06**: a `batch-service` implementing Spring Batch's
  Job → Step → Chunk (Reader/Processor/Writer) model, reading order data
  through `order-service`'s existing public REST API (now with real
  `limit`/`offset` pagination, added alongside this work) and writing a
  periodic aggregate (a daily order summary). This is the highest-signal
  addition to this phase — it directly reproduces the deepest, most recent
  8 years of professional specialization (ADR-0005). Design details in
  that repo's own
  [ADR-0010](https://github.com/schware/sun-moon-python-platform/blob/master/docs/adr/0010-batch-service-design.md),
  including the direct comparison against `sun-moon-c-server`'s C
  implementation. 17 tests passing across the affected packages; verified
  end-to-end against a live `order-service` (10 orders, revenue matched exactly).

**Why this comes first**: it's the lowest-risk, highest-leverage phase —
no new language to learn, and it converts an already-built system from
"demo" to "defensible." Skipping straight to Go/TypeScript with a shaky
Python foundation would be a mistake.

## Phase 1.5 — C Batch + cross-repo integration — done, 2026-09-06

**Both sides done.** C side 2026-09-06 (below); Python side (in Phase 1
above) same day, once `order-service` gained real pagination.

Goal, per ADR-0006: prove the same Job/Step/Chunk design in C — a language
with no built-in abstraction mechanism, making this the hardest and
therefore most convincing proof of ADR-0004's thesis — and prove
`sun-moon-python-platform`'s "contract, not shared code" architecture
holds even across a completely different, older tech stack.

- ~~In `sun-moon-c-server`: add a real DB client (SQLite)~~ — **deliberately
  not done as part of this phase.** `sun-moon-c-server`'s own
  [ADR-0001](https://github.com/schware/sun-moon-c-server/blob/master/docs/adr/0001-batch-job-design.md)
  scoped this out explicitly: the batch job's storage is files
  (NDJSON + JSON), not a database, so the `db_demo_service.c` simulated-DB
  gap stays open as its own separate, still-unscheduled item on that
  repo's "Planned next steps."
- **Done**: a C HTTP client (libcurl, wired into CMake) that calls
  `order-service`'s public `GET /orders` — the same contract
  `ai-agent-service` already consumes, now proven to work from C too.
- **Done**: the Job/Step/Chunk pattern implemented by hand in C idioms (a
  struct of function pointers standing in for Reader/Processor/Writer) —
  `include/batch.h`/`src/batch.c` (generic engine) +
  `src/services/order_summary_batch_job.c` (the concrete job: sums
  revenue per chunk, streams NDJSON, writes a JSON summary + JSON
  execution report). `config/batch-job.json` drives it, matching the
  "manage batch via JSON" instruction this phase was built to. Verified
  end-to-end against a live `order-service`: single-chunk
  (`chunk_size: 5`, 3 orders) and multi-chunk (`chunk_size: 2`, 2+1)
  both produced correct revenue totals; `scripts/run_batch.sh` correctly
  propagates `batch_runner`'s exit code (0 success / 1 on failure) for
  cron/systemd-timer scheduling. See
  [`sun-moon-c-server`'s ADR-0001](https://github.com/schware/sun-moon-c-server/blob/master/docs/adr/0001-batch-job-design.md)
  for the full design and its own "Alternatives considered"/"Consequences."
- **Done**: direct comparison against the Python `batch-service` — same
  job semantics, same output shape, same fault-tolerance model, but the
  Python reader is genuinely chunk-bounded over HTTP while this C reader
  still fetches everything in one call (see the "Known gap" note this
  repo's own README now carries, and its updated "Planned next steps").

**Why right after Phase 1, before Go**: C is already-known territory (low
risk) — sequencing it before the genuinely new language (Go) builds a
finished comparison pair (Python batch vs. C batch, same design) while
confidence is high, and gives Phase 2 a second data point to build on
rather than starting Go cold.

**Schedule impact, actual vs. estimated**: this phase was estimated at
~2-3 weeks; both the C and Python sides actually shipped the same day
(2026-09-06) they were scoped, working with Claude. Worth treating as a
calibration point rather than assuming every future phase compresses this
much — this particular phase reused an already-designed pattern (Job/
Step/Chunk) twice rather than inventing something new, which is a big
part of why it went faster than estimated. Phase 2/3's target dates below
are left as originally estimated rather than pulled in, until Phase 2
itself provides a second data point.

## Phase 1.6 — `sun-moon-c-server`: unify ports, refactor toward service-based architecture (target: next session)

Goal, per [ADR-0007](docs/adr/0007-prefer-process-isolation-over-port-splitting.md):
bring `sun-moon-c-server` in line with `sun-moon-python-platform`'s
service-per-process model — a second concrete proof (alongside Job/Step/
Chunk) that ADR-0004's "principles transfer across languages" thesis
extends to service decomposition itself, not just one design pattern.

- Reconsider `server`'s current shape: one CMake target, one process,
  config-driven mode selection (tcp/http/udp) via threads, spread across
  several special-purpose ports (8081 HTTP, 8090 TCP, 8082 UDP, 8443
  HTTPS, a 9010-9020 TCP-range demo) — plus `batch_runner`, already a
  separate one-shot binary.
- Move toward independent, separately-runnable services/binaries per
  business capability (mirroring `order-service`/`ai-agent-service`/
  `batch-service`), each with one consistent port, instead of one process
  juggling multiple modes via config + threads.
- Not yet decided: exact service boundaries for the C side (which
  existing modes become which services), and whether the
  `combined-server.json`/multi-thread convenience mode is removed or kept
  as an option. Needs its own ADR in `sun-moon-c-server` once this starts.
- No code changes yet — this phase starts next session (owner's own
  scheduling note: "내일 토큰이 초기화되면").

## Phase 2 — one Go component — revisit next session (per ADR-0008)

Not paused indefinitely — scheduled for next session (tomorrow), same as
Phase 1.6. The owner wants to reorganize their own thinking on
Go/TypeScript/Rust first; everything below is today's content, kept as a
starting point for that next session, not a settled plan.

**Content as of today (2026-09-06), to be revisited next session:**

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

## Phase 3 — DevOps polish (target: ~2-3 weeks, likely spilling past Q4 2026 into the transition window itself — see schedule note above)

Goal: make the **AI/ML + Backend** work in Phases 1-2 look and run like a
real, operable system — this is where the DevOps/SRE signal mostly comes from.

- Verify `docker-compose.yml` actually builds and runs (flagged as
  untested in that repo — this environment had no Docker to test with).
- A real CI pipeline covering the Go component too (lint + build + test).
- A live deployment somewhere free/cheap (Fly.io, Render, a small VPS) —
  a working URL beats a thousand words in an interview.

## Phase 4 — optional / stretch — revisit next session (per ADR-0008)

Same as Phase 2: not paused indefinitely, just revisited next session
alongside the reorganized Go/TypeScript/Rust plan (TypeScript specifically
is this phase's language).

- A minimal TypeScript Dashboard hitting `/health` and recent
  orders/agent notes. Explicitly the lowest-priority component — fine to
  defer past the Q4 target, or past the transition window entirely,
  without treating it as a failure. Not every box in the original Platform
  diagram needs to be checked before this is a legitimate, presentable portfolio.

## Parallel track (outside this roadmap's time budget) — Mobile App

Per [ADR-0011](docs/adr/0011-mobile-app-parallel-portfolio-track.md),
**not a Phase** — deliberately kept outside this roadmap's Q4 2026
sequencing and scope-cutting rules, so it never competes with Phase
1.6/2/3/4 for time. Noted here only so its existence is visible.

- **What**: cross-platform (iOS/Android) app at `D:\Claude_Code\App`
  (separate repo), Expo/React Native/TypeScript. Two features: paste a
  meeting transcript → Claude API Markdown summary → GitHub commit; and a
  morning-plan/evening-review daily schedule → GitHub commit as Markdown.
- **Why it's recorded at all**: the owner confirmed this is intended as a
  real portfolio signal (mobile + third-party AI/API integration), not a
  career-irrelevant side tool — but explicitly parallel, not sequenced
  against this repo's Q4 target.
- **Status, 2026-09-06**: MVP scaffolded and smoke-tested (web preview) —
  4 screens, GitHub PAT-based sync, on-device secure storage for
  credentials, paste-only meeting input (no recording/STT yet).
- If this track ever starts drawing on time budgeted for Phase 1.6/2/3,
  that drift needs its own ADR (see ADR-0011's Consequences) rather than
  silently eating into the Q4 timeline.

## Proposed (unscheduled) — Java: complete the three-way Batch comparison

Per [ADR-0008](docs/adr/0008-pause-go-ts-rust-add-java-cpp.md), **not yet
confirmed** — proposed by Claude, awaiting the owner's go-ahead before
this gets a target date.

- Build the same Job/Step/Chunk design a third time in actual Java/Spring
  Boot (likely real Spring Batch, not hand-rolled — Java already has the
  canonical framework). Python's `batch-service` and C's `batch_runner`
  already exist.
- Java would be the **reference implementation**, not a peer demo — the
  "known-good" baseline the other two get checked against, since Spring
  Batch is where the owner has 8 real production years (ADR-0005). This
  is the most direct way to make the portfolio *show* that expertise.

## Proposed (unscheduled) — C++: a focused C++20-coroutine IOCP fix

Also per ADR-0008, **not yet confirmed**.

- ADR-0004 named C++20 coroutines (`co_await` + Boost.Asio/cppcoro) as the
  same-language fix to the original department-store system's
  completion-callback fragmentation — distinct from `sun-moon-c-server`'s
  plain-C `batch_runner` (no coroutines, no classes).
- A small, scoped demo: linear-looking async code over the same IOCP
  mechanism, with clean object decomposition — a sharper "here's the
  actual fix, in the actual language" artifact than the Go CSP analogy alone.

## Explicit scope-cutting rule

If time runs short, cut from the bottom of this list (Phase 4, then late
Phase 3) before cutting depth from Phase 1/1.5/2 — a smaller, finished,
well-documented system beats a sprawling, half-finished one in every
interview conversation this is likely to come up in. Adding Phase 1.5
(ADR-0006) makes Phase 4 (TypeScript Dashboard) more likely to be cut
entirely, not less — that tradeoff was made deliberately: Batch directly
proves 8 years of specialization (ADR-0005), while the Dashboard proves a
skill with no prior exposure at all. If forced to choose, Phase 1.5 wins.

## Not yet decided

- Whether "Q4 2026" means "portfolio must be interview-ready before the
  transition" or "keep building immediately after it" (ADR-0002 flagged
  this as open) — now more pressing, since Phase 1.5 already pushes Phase
  3 past the original Q4 target (see that phase's note above).
- Go/TypeScript/Rust's reorganized shape (Phase 2/4, revisit next
  session) — the owner is rethinking this themselves before the next
  conversation; do not assume today's Phase 2/4 content is still current
  without checking.
- Whether the Java and C++ proposals above get confirmed, and if so, how
  they sequence against Phase 1.6 and the reorganized Go/TS/Rust plan —
  all scheduled for next session's discussion.

## Already decided (for the record)

- GitHub visibility: both `sun-moon-python-platform` and this repo are
  Public, deliberately, including the personal/timeline content here — the
  owner's own current employer is already aware, and public visibility is
  intentionally being used as a commitment device.
