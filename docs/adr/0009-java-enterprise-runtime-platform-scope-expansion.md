*🇰🇷 Korean version: [0009-java-enterprise-runtime-platform-scope-expansion_kr.md](0009-java-enterprise-runtime-platform-scope-expansion_kr.md)*

# ADR-0009: Java — expand from a Batch-only reference implementation toward a full Netty-based Enterprise Runtime Platform

- **Status**: Ongoing (direction-setting) — the shape and candidate stack
  are recorded today; the architecture itself is deliberately still open,
  the same way ADR-0001 uses "Ongoing" for topics revisited over time
  rather than closed to one answer
- **Date**: 2026-09-06
- **Deciders**: Project owner (proposed the stack and the platform shape);
  Claude (suggested additions and flagged the scope change below)

## Context

ADR-0008 scoped Java narrowly: item 2 there proposed Java only as the
**Spring Batch reference implementation** in a three-way Job/Step/Chunk
comparison against the already-built Python (`batch-service`) and C
(`batch_runner`) versions — not a new Platform, just a third Batch demo.

Today the owner brought a substantially larger shape: a Java, DDD-based
**Enterprise Runtime Platform** that runs Socket, REST API, WebSocket, and
Batch Job together in a single runtime, open-source-first, targeting
1,000-10,000 concurrent connections. Structurally, this is the same kind
of ambition as `sun-moon-c-server` (one runtime, multiple transports) —
not the narrow Batch-only comparison ADR-0008 scoped. The owner proposed a
candidate stack and asked for further open-source recommendations and help
shaping direction, explicitly framing it as still being worked out
("고민하다가 정리한 것").

## Decision

Record today's proposed stack, Claude's suggested additions, and the scope
observation below — without freezing an architecture. This ADR is
deliberately left "Ongoing," matching this project's own established
pattern (see ADR-0001, and [[feedback-avoid-confirmed-exploratory-framing]]):
the repositioning itself (Java gets more than a Batch-only demo) is worth
recording now, but the concrete component boundaries are a continuing
direction-setting process, not a one-time gate.

**Owner's proposed stack, as given today:**

- Netty — Core Runtime / Transport Engine
- Quartz — Batch Scheduler
- MyBatis + Oracle — Persistence Layer
- Redis — Cache / Session / Lock
- Kafka — Event Bus
- Logback — per-service Logging
- Micrometer / Prometheus / Grafana — Monitoring
- OpenTelemetry — Distributed Tracing

**Claude's suggested additions, with rationale:**

- **Spring (Core DI + Spring Batch — not Spring Boot's embedded Tomcat).**
  This reconnects directly to ADR-0008's original point: Quartz by itself
  has no Job/Step/Chunk/Reader/Processor/Writer model — it is a cron-style
  trigger, nothing more. Spring Batch is what actually supplies that model,
  and is the "known-good baseline" ADR-0008 wanted precisely because it's
  where the owner has 8 real production years (ADR-0005). The natural
  pairing is Quartz *triggering* Spring Batch jobs, not Quartz replacing
  Spring Batch.
- **Reactor Netty / Spring WebFlux**, if REST and WebSocket should share
  Netty's event loop with Spring-managed services rather than a hand-rolled
  router sitting directly on raw Netty — an open question, see below.
- **HikariCP** — connection pooling; close to mandatory alongside
  MyBatis + Oracle in any real deployment.
- **Jackson** — JSON serialization, the de facto standard in this stack.
- **Jakarta Bean Validation (Hibernate Validator)** — request/DTO validation
  at the transport boundary.
- **Resilience4j** — circuit breaker / retry / rate limiter, directly
  relevant to a 1,000-10,000 concurrent-connection target.
- **Redisson** — a maintained Redis client with real distributed-lock
  semantics, instead of a hand-rolled Redis lock.
- **Flyway** — Oracle schema migrations; mirrors the Alembic gap already
  flagged on the Python side (`ROADMAP.md` Phase 1).
- **JUnit5 + Mockito + Testcontainers** — matches the "verified against a
  live instance, not just unit-tested" pattern already used for the
  Python/C Batch work (ROADMAP.md Phase 1/1.5).

## Scope observation (the main open question)

What the owner described today is structurally bigger than ADR-0008 item
2. That item scoped Java narrowly as a Batch-only reference implementation
sitting alongside the existing Python/C Batch work. What's on the table
now — a Netty Core Runtime hosting Socket + REST + WebSocket + Batch
together — is the same shape of ambition as `sun-moon-c-server` itself: a
third full Platform reimplementation, not a fourth Batch demo layered onto
the third.

That's not necessarily a wrong direction — Java is 8 years of real
production depth (ADR-0005), and per ADR-0008's own reasoning it's the
lowest new-skill-acquisition-risk language on this whole roadmap. But it
does reopen ADR-0003's scope-cutting discipline: Phase 3 is already
spilling past the original Q4 2026 target, and Phase 4 (TypeScript
Dashboard) is already flagged as the first thing to cut if time runs
short. Adding a full Java Platform is a larger addition to total portfolio
surface area than the Batch-only version ADR-0008 proposed.

## Alternatives considered

- **Keep Java narrowly scoped to Batch only** (ADR-0008's original item 2)
  — faster, lower-risk, and already-proposed; not taken because the
  owner's own message today moved past this scope. Recording that shift
  here rather than silently letting ADR-0008's narrower framing stand
  unchallenged.
- **Decide the full architecture in this same turn** (DI approach, routing
  layer, repo name, sequencing against Phase 1.6 and the reorganized
  Go/TS/Rust plan) — not taken. Matches this project's established
  pattern of proposing, then getting the owner's confirmation, before an
  ADR moves from "Ongoing"/"Proposed" to "Accepted."

## Consequences

- `ROADMAP.md`'s "Proposed (unscheduled) — Java" section is updated
  alongside this ADR to reflect the expanded stack and this scope
  observation — still unscheduled, no target date yet.
- If the owner confirms the full-Platform scope, a follow-up ADR (or this
  one moving to Accepted) should record: the repo name (candidate:
  `sun-moon-java-platform`, matching the `sun-moon-python-platform` /
  `sun-moon-c-server` naming convention), the DI/routing architecture
  decision, and where this sequences against Phase 1.6 and the reorganized
  Go/TS/Rust plan.
- **Not yet decided**: DI container approach (Spring Core vs. none, given
  Netty is already the transport); whether REST/WS routing sits on raw
  Netty or on Reactor Netty/Spring WebFlux; build tool (Gradle vs. Maven);
  the repo name; and how this sequences against everything else already
  queued for "next session" (Phase 1.6, the reorganized Go/TS/Rust plan).

## References

- ADR-0008 (this repo) — the original, narrower Java proposal this one
  expands on.
- ADR-0003 (scope-cutting discipline), ADR-0004 (cross-language technical
  thesis), ADR-0005 (the 8-year Java/Spring Boot baseline) — all this
  repo.
- `sun-moon-c-server`'s architecture (one runtime, multiple transports) as
  the structural precedent for what's being proposed here.
