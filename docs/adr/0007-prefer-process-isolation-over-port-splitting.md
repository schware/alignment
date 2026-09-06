*🇰🇷 Korean version: [0007-prefer-process-isolation-over-port-splitting_kr.md](0007-prefer-process-isolation-over-port-splitting_kr.md)*

# ADR-0007: Prefer process/service separation (and worker-pool separation) over splitting one service's traffic across multiple ports; schedule a matching C refactor

- **Status**: Accepted (Python side); C-side refactor scheduled, not yet started
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

The owner asked whether `order-service` should split its TCP traffic
across multiple ports (isolating business tasks from each other so a slow
or heavy one doesn't degrade others) versus keeping one port per service
and dividing work at the service level instead — explicitly naming the
tradeoff themselves: port-splitting has a real resilience motivation, but
"관리적인 측면에서 매력도가 떨어지는 것은 사실" (honestly less
attractive operationally).

## Decision

**Splitting traffic across multiple ports within one service is not the
right tool for this.** Two ports on the same process still share the same
event loop/CPU/memory — a port boundary alone gives no real isolation. The
isolation the owner actually wants comes from two other places, both
already in use in this project:

1. **Process/service boundaries** — `order-service`, `ai-agent-service`,
   and `batch-service` are already separate processes (`sun-moon-python-platform`'s
   ADR-0005). A slow/overloaded `order-service` cannot degrade
   `ai-agent-service`'s responsiveness; this is already the coarse-grained
   isolation the owner was asking for.
2. **Worker-pool separation within a process**, for isolating slow/blocking
   work from a service's own responsiveness — the same pattern
   `sun-moon-c-server`'s `core_submit_work()` already demonstrates
   (network I/O threads never block on DB-style work; it's offloaded to a
   separate thread pool). Python's equivalent is offloading blocking work
   via a separate task/thread rather than running it inline on the event loop.

**One narrow, legitimate exception noted but not adopted here**:
control-plane vs. data-plane port separation (e.g. a dedicated
health-check/metrics port isolated from business traffic, as Kubernetes
liveness probes commonly do) is a real, different pattern from "split
business tasks across ports" — not in scope for this decision, but worth
remembering as a distinct future option if a specific need for it comes up.

**Python (`sun-moon-python-platform`) requires no code change** — its
existing one-port-per-service model already reflects this decision.

**`sun-moon-c-server` gets a scheduled refactor** (next session, once the
owner's token budget resets): move from one `server` binary running
multiple modes (tcp/http/udp) via config-driven threads across several
special-purpose ports, toward a service-per-process model mirroring
`sun-moon-python-platform`'s — unifying around a consistent port story
per service instead of the current spread across many ports/modes. See
`ROADMAP.md`'s new Phase 1.6.

## Alternatives considered

- **Split `order-service`'s TCP traffic across multiple ports by task
  type** — the option the owner was originally weighing. Rejected: doesn't
  achieve the stated goal (isolating slow work) without also separating
  execution context, and the owner already identified the operational cost
  themselves.
- **Leave `sun-moon-c-server`'s architecture as-is** — considered, but the
  owner wants it to demonstrate the same service-decomposition judgment
  `sun-moon-python-platform` already does, extending ADR-0004's
  "architecture principles transfer across languages" thesis from
  Job/Step/Chunk to service decomposition itself. Deferred to next
  session, not rejected outright.

## Consequences

- No immediate code changes anywhere — this ADR records a direction, and
  schedules one piece of follow-up work.
- `sun-moon-c-server`'s refactor is nontrivial: today it's one CMake
  target (`server`) with config-driven mode selection across several
  ports (8081 HTTP, 8090 TCP, 8082 UDP, 8443 HTTPS, a 9010-9020 TCP-range
  demo), plus the already-separate `batch_runner`. Splitting business
  capabilities into independent binaries/processes touches `main.c`,
  `config.c`, `CMakeLists.txt`, and probably the `config/*.json` files —
  real scope, not a quick edit. Needs its own ADR in that repo once work
  starts, per this project's established practice.
- Gives the eventual C refactor a second concrete "principles transfer
  across languages" data point (alongside Job/Step/Chunk) for the
  portfolio narrative (ADR-0004's thesis).

## References

- This conversation, 2026-09-06.
- `sun-moon-python-platform/docs/adr/0005-monorepo-of-independent-services-via-uv-workspace.md` —
  the service-separation pattern being confirmed for Python and proposed for C.
- `sun-moon-c-server/README.md`'s `core_submit_work()` description — the
  existing worker-pool-separation pattern this ADR points to as the right
  tool for isolating slow work, instead of extra ports.
- `alignment/docs/adr/0004-core-technical-thesis-decouple-io-completion-from-domain-logic.md` —
  the "principles transfer across languages" thesis this extends.
