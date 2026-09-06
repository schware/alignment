*🇰🇷 Korean version: [0006-dual-language-batch-via-shared-contract_kr.md](0006-dual-language-batch-via-shared-contract_kr.md)*

# ADR-0006: Build the Batch component twice — C (`sun-moon-c-server`) and Python (`sun-moon-python-platform`) — sharing one design and, where possible, one live contract

- **Status**: Accepted — C side implemented 2026-09-06 (see
  [`sun-moon-c-server`'s ADR-0001](https://github.com/schware/sun-moon-c-server/blob/master/docs/adr/0001-batch-job-design.md)
  and this repo's `ROADMAP.md` Phase 1.5 for verification details); Python
  side still pending
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

ADR-0005 identified 8 years of Spring Boot Batch work as the single
deepest, most-provable technical asset available for this transition, and
flagged that neither `sun-moon-python-platform` nor `sun-moon-c-server`
currently has any Batch-shaped component. The owner then confirmed the
intent directly: not "pick one language for Batch," but **add this
direction to both GitHub repos** — `sun-moon-c-server` (yesterday's C
server framework, now pushed to GitHub at
https://github.com/schware/sun-moon-c-server) and
`sun-moon-python-platform`.

## Decision

Build the same conceptual design — Spring Batch's **Job → Step → Chunk
(Reader / Processor / Writer)** model — twice, once per repo, and use the
existing `order-service` REST contract (from `sun-moon-python-platform`'s
ADR-0006) as the shared data source for both, rather than building two
disconnected toy batch jobs:

- **Python side (`sun-moon-python-platform`)**: a new `batch-service` in
  the monorepo. Reads order data through `order-service`'s existing public
  REST API (the same seam `ai-agent-service` already uses — see that
  repo's README "The two seams between services"), processes it in
  chunks, and writes a periodic aggregate (e.g. a daily order summary).
  Detailed design (reader/processor/writer interfaces, execution-history
  tracking analogous to Spring Batch's job repository) is deferred to that
  repo's own ADR when this phase starts, per the precedent set for the Go
  component in ADR-0004.
- **C side (`sun-moon-c-server`)**: adds the real DB client that repo's
  own README already flags as a gap ("A real Oracle client behind
  `core_submit_work()`, replacing the `db_demo_service.c` simulation") —
  starting with SQLite rather than Oracle to keep scope realistic for a
  portfolio timeline — **and** fetches order data from the same
  `order-service` REST API using a C HTTP client (e.g. libcurl), building
  the Job/Step/Chunk pattern by hand in C idioms (a struct of function
  pointers standing in for the Reader/Processor/Writer interfaces, since C
  has no built-in abstraction mechanism for this).

**Why route the C implementation through the same REST API instead of
giving it its own independent toy dataset**: it turns two disconnected
demos into one integrated proof that `sun-moon-python-platform`'s
"contract, not shared code" architecture (that repo's ADR-0006) actually
holds across a completely different, older, lower-level technology stack —
not just between two modern Python-ecosystem services. This is a
materially stronger portfolio claim than three isolated single-language demos.

## Alternatives considered

- **Pick one language for Batch (Python only, or C only)** — the owner
  explicitly rejected this; both directions are wanted, not a choice
  between them.
- **Build the C batch job against its own independent, unrelated
  dataset** — simpler (no cross-repo HTTP dependency, no need to keep
  order-service's API contract stable while building against it), but
  gives up the strongest available claim (a working polyglot integration
  across a genuinely different tech stack, not just a language-syntax demo).
- **Merge `sun-moon-c-server` into the `sun-moon-python-platform` monorepo**
  — rejected: the two are different enough in tooling (CMake/vcpkg vs.
  uv workspace) that a shared repo would only add friction; two repos
  integrated by a live HTTP contract demonstrates the same "contract over
  shared code" principle at the repo level, not just the service level.

## Consequences

- Combined estimated effort (discussed in chat before this ADR): roughly
  2.5-3 weeks of part-time work for both implementations together — less
  than doing them fully independently, since the Job/Step/Chunk design
  only needs to be *thought through* once, even though it's *coded* twice.
  Still a real addition to `ROADMAP.md`'s timeline; see that file's revised
  phase order and its updated scope-cutting note.
- `sun-moon-c-server`'s new C HTTP client dependency (libcurl or
  equivalent) needs its own build-system wiring (CMake) — the first
  external network-client dependency that repo will have, beyond what it
  already vendors (cJSON, log.c).
- `sun-moon-python-platform`'s `order-service` REST API becomes a contract
  now depended on by *two* other things (`ai-agent-service` and, soon,
  both the Python `batch-service` and the C batch job) — raises the bar on
  not breaking that API's shape without updating all three, which is
  itself a realistic, demonstrable "why contracts need versioning
  discipline" story for later.
- Reinforces, rather than dilutes, ADR-0003's "one Platform, not scattered
  projects" strategy — both Batch implementations are additions to
  already-planned repos around an already-existing contract, not a third
  unrelated project.

## References

- ADR-0004, ADR-0005 (this repo) — the technical thesis and the
  professional-timeline basis for prioritizing Batch at all.
- `sun-moon-python-platform/docs/adr/0006-inter-service-communication-http-and-redis.md` —
  the REST contract being extended to a third (and fourth, cross-repo) consumer.
- `sun-moon-c-server/README.md`'s "Planned next steps" — the pre-existing,
  independently-flagged real-DB-client gap this ADR resolves.
