# ADR-0010: Java — drop Spring entirely; keep the originally-proposed stack, hand-roll Batch

- **Status**: Accepted (the Spring-drop itself); routing/DI-replacement
  details remain open, same as ADR-0009's other unresolved items
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

[ADR-0009](0009-java-enterprise-runtime-platform-scope-expansion.md),
written earlier the same day, suggested adding Spring (Core DI + real
Spring Batch, optionally Reactor Netty/Spring WebFlux for REST/WS routing)
to Java's Enterprise Runtime Platform stack — explicitly to reconnect with
ADR-0008's original reasoning that Java should be the Batch "known-good
baseline" *because* it has the canonical framework (Spring Batch) that
Python and C lack.

In a follow-up message the same day, the owner rejected that addition:
"Spring을 포기하고 아까 정의한 것을 기반으로 진행할려구요" (dropping
Spring, proceeding based on what was defined earlier) — i.e. the
originally-proposed stack from ADR-0009 (Netty, Quartz, MyBatis + Oracle,
Redis, Kafka, Logback, Micrometer/Prometheus/Grafana, OpenTelemetry),
without Spring anywhere in it.

## Decision

The Java Platform proceeds with no Spring dependency at all:

- **No Spring Core DI** — dependency wiring in the DDD structure is
  hand-built, not assembled through Spring's IoC container.
- **No Spring Batch** — Quartz stays as the Batch *Scheduler*, but the
  Job/Step/Chunk/Reader/Processor/Writer model itself now has to be
  hand-rolled directly in Java — the same way `sun-moon-c-server`'s
  `batch_runner` hand-rolled it in C (a struct of function pointers
  standing in for Reader/Processor/Writer; see that repo's ADR-0001) —
  except in Java idiom (interfaces/classes), not C's function-pointer
  structs.
- **No Reactor Netty / Spring WebFlux** — REST and WebSocket routing sits
  directly on raw Netty channel pipelines.

**Worth flagging directly**: this changes the three-way Batch comparison's
narrative from ADR-0008. ADR-0008 picked Java as the "reference
implementation"/"known-good baseline" specifically *because* it has the
canonical framework Python and C lack. Without Spring Batch, Java's Batch
implementation becomes a third hand-rolled engine — structurally closer to
C's approach (idiomatic-Java hand-rolled, not framework-backed) than to
the "framework-backed known-good baseline" ADR-0008 originally described.
This doesn't make the decision wrong — Java's real differentiator is still
8 years of production Java/OOP design judgment (ADR-0005), independent of
whether Spring Batch specifically is used — but the "uses the canonical
framework" framing no longer applies and should stop being repeated as the
reason Java is the reference point.

## Alternatives considered

- **Partial Spring** (keep Spring Batch only, drop Core DI/WebFlux) — not
  chosen; the owner's instruction was a full drop of Spring, not a partial one.
- **Keep ADR-0009's full Spring proposal** — rejected by explicit owner
  instruction this same session.

## Consequences

- `ROADMAP.md`'s Java section is updated to remove Spring from the
  suggested stack, and to note the Job/Step/Chunk model will be hand-built
  in Java directly (Quartz remains for scheduling/triggering only).
- The "reference implementation via canonical framework" framing carried
  from ADR-0008 needs softening wherever it's repeated: Java's edge is
  production experience, not framework availability, once Spring Batch is
  off the table.
- ADR-0009's other suggested additions are unaffected by this decision —
  HikariCP, Jackson, Jakarta Bean Validation (Hibernate Validator),
  Resilience4j, Redisson, Flyway, and JUnit5/Mockito/Testcontainers are all
  usable standalone, with no Spring dependency, so none of them needed
  reconsidering here.
- Still open (carried from ADR-0009, now narrower): what replaces Spring's
  DI role (manual wiring vs. a lightweight non-Spring DI library), build
  tool (Gradle vs. Maven), repo name, and sequencing against Phase 1.6 and
  the reorganized Go/TS/Rust plan.

## References

- ADR-0008, ADR-0009 (this repo).
- `sun-moon-c-server`'s ADR-0001 (hand-rolled Job/Step/Chunk in C) — the
  structural precedent for hand-rolling the same pattern in Java now.
