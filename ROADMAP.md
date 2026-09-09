*🇰🇷 Korean version: [ROADMAP_kr.md](ROADMAP_kr.md)*

# Roadmap — a living document

This document is a draft as of now, not a fixed plan. Per this project's
working style (build a piece, reflect, add to it), expect this document to
change as each phase completes and priorities shift. When a phase's scope
changes materially, write an ADR about *why* rather than silently editing
this file's history away.

**Baseline this is built on**: ADR-0002/ADR-0005 (16 years total)

- MFC years 1-4
- C++ server years 5-8
- C# POS Client years 5-16
- Java/Spring Boot REST+Batch years 9-16
- no cloud-native/AI/ML experience built yet
- ~3.5 months from 2026-09 to a Q4 2026 target, market-agnostic, no existing portfolio

**Strategy this follows**: ADR-0003 (one Platform, deliberately
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
- A live deployment somewhere free/cheap (~~Fly.io~~, Render, a small VPS) —
  a working URL beats a thousand words in an interview. **Correction,
  2026-09-09**: Fly.io's free tier ended October 2024 — new accounts get a
  short trial, then a paid minimum. Cross this off as a free candidate for
  any track. See the Java section below for a researched, concrete pick
  (Render + Neon) that could generalize to Python/Go too when this phase
  actually starts.

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

## Activity log (outside this roadmap's Phases) — `sun-moon-java-platform` modernization

Not a Phase, and not itself new portfolio-building work in the ADR-0003 sense — the value here is *maintaining and modernizing* the pre-existing Java/Spring Boot expertise (the years 9-16 baseline in ADR-0005), not acquiring a new skill this roadmap tracks. Recorded so it's visible, the same way the Mobile App track is.

- **What, 2026-09-08**: converted `sun-moon-java-platform` from a single WAR deployed on shared Jetty into three independently deployed Spring Boot services (Order/KDS/Delivery) split across separate GitHub repos joined by a git-submodule umbrella, each running in its own Docker container with its own `context-path` (`/order`, `/kds`, `/delivery`). Order and Delivery moved to PostgreSQL+JSONB after discovering the server's 2010-era CPU lacks the AVX instructions MongoDB requires; KDS uses Redis. Fixed a stale path-redaction bug left over from the WAR/Jetty era (health/metrics endpoints were leaking the raw Docker container path since the old redaction marker no longer matched), and kept the OpenAPI `servers` URL in each service in sync with its new context-path.
- **Why it's recorded at all**: the resulting `-jetty`-suffixed repos and the `docs/adr` records in `sun-moon-java-platform` (0001-0006) are independently defensible evidence of the ADR-0005 Java/Spring Batch baseline being actively maintained, not stale — worth being visible from this repo even though it isn't part of the Q4 portfolio-building sequence.
- This work doesn't compete with Phase 1/1.5/1.6/2/3/4 for time budget — it happened alongside them, not instead of them.
- **Naming collision, worth flagging explicitly**: the *from-scratch, Netty-based, no-Spring* Java portfolio project described just below (the "Proposed... Enterprise Runtime Platform" section) happens to share the exact repo name `sun-moon-java-platform` with this real, pre-existing Spring Boot system — same name, same GitHub repo even, but different git branches: `main` is this real Spring Boot/WAR-turned-microservices system (this activity-log entry); `master` is the from-scratch, no-Spring rebuild described below. They are two unrelated codebases living in the same repo under different branches — don't conflate them when reading this roadmap, that repo's own commit history, or its `docs/adr/` (each branch has its own independent ADR numbering).

## Proposed (unscheduled) — Java: a DDD-based Enterprise Runtime Platform

Per [ADR-0008](docs/adr/0008-pause-go-ts-rust-add-java-cpp.md),
[ADR-0009](docs/adr/0009-java-enterprise-runtime-platform-scope-expansion.md),
and [ADR-0010](docs/adr/0010-java-drop-spring.md), **not yet fully
confirmed** — the shape below is Ongoing direction-setting, not a
scheduled scope. ADR-0008 originally proposed Java only as a Batch-only
reference implementation; ADR-0009 records that the owner expanded this on
2026-09-06 into a full single-runtime Platform, structurally closer to
`sun-moon-c-server`'s ambition (one runtime, multiple transports) than to
a third Batch demo; ADR-0010 (same day) records the owner rejecting
ADR-0009's Spring addition, keeping the stack Spring-free.

- **Shape**: one Java runtime hosting Socket, REST API, WebSocket, and
  Batch Job together, DDD-structured, open-source-first, targeting
  1,000-10,000 concurrent connections.
- **Confirmed stack (owner, 2026-09-06, no Spring per ADR-0010)**: Netty
  (Core Runtime / Transport, including raw REST/WS routing on Netty's own
  channel pipelines — no Reactor Netty/WebFlux), Quartz (Batch Scheduler —
  triggers a hand-rolled Job/Step/Chunk engine, doesn't use Spring Batch),
  MyBatis + Oracle (Persistence), Redis (Cache/Session/Lock), Kafka (Event
  Bus), Logback (Logging), Micrometer/Prometheus/Grafana (Monitoring),
  OpenTelemetry (Tracing).
- **Also suggested, Spring-independent (ADR-0009, unaffected by the
  Spring drop)**: HikariCP; Jackson; Jakarta Bean Validation (Hibernate
  Validator); Resilience4j; Redisson; Flyway; JUnit5 + Mockito +
  Testcontainers.
- Java is still the highest-real-production-experience implementation in
  the Job/Step/Chunk comparison (8 years, ADR-0005), but per ADR-0010 this
  is no longer "the one using the canonical framework" — the Job/Step/
  Chunk/Reader/Processor/Writer model is now hand-rolled in Java idiom,
  structurally closer to C's hand-rolled `batch_runner` than to
  framework-backed Spring Batch.
- **Design note (2026-09-06)**: Netty covers Socket + REST API + WebSocket
  natively (`HttpServerCodec`/`HttpObjectAggregator` for REST,
  `WebSocketServerProtocolHandler` for WS, a hand-written
  `ByteToMessageDecoder`/`MessageToByteEncoder` pair for raw Socket, all on
  the same `ChannelPipeline` mechanism) and comfortably clears the
  1,000-10,000 concurrent-connection target — this is judged feasible, and
  structurally the same proof `sun-moon-c-server` already gave in C via
  libuv. The one thing that needs deliberate design, precisely because
  Spring isn't there to paper over it: Netty's event-loop threads must
  never block, but MyBatis + Oracle (JDBC) calls are blocking by nature —
  so domain-service calls that hit Oracle need to be handed off to a
  separate worker thread pool and the result bridged back onto the Netty
  channel, rather than called inline on an I/O thread.
- **Started, 2026-09-06**: `sun-moon-java-platform` scaffolded at
  `D:\Claude_Code\Network\Java` — Gradle 8.11 (Kotlin DSL, wrapper
  committed) + JDK 21, both freshly installed for this (neither was
  present before). First slice built and verified end-to-end: one Netty
  `ServerBootstrap` (`CoreRuntime`) serving `GET /health` via a small
  hand-written REST router, returning `{"status":"UP"}`, then a clean
  shutdown. Everything else in the stack above is still unimplemented —
  see that repo's own `docs/adr/0002` for the full plan and what's left.
  Build tool (Gradle) and repo name (`sun-moon-java-platform`) are
  therefore now decided as a side effect of starting; DI-replacement
  approach and sequencing against Phase 1.6/Go/TS/Rust remain open.
- **Expanded, same day**: the owner asked to expand every planned piece at
  once rather than one at a time. Done and **live-verified** (see that
  repo's `docs/adr/0003`): Socket + WebSocket transports (real socket/WS
  clients against the real server), the hand-rolled Job/Step/Chunk Batch
  engine triggered by a real Quartz `Scheduler` (5 seeded orders, revenue
  310.74, matches the Python/C order-summary job's shape), Micrometer
  (`GET /metrics` returns real Prometheus scrape text), OpenTelemetry
  (a real span logged per `/health` call), Jakarta Bean Validation +
  Resilience4j (exercised live through a new `POST /orders` endpoint — a
  blank `customerId` really returns `400`). **Not live-verified** — this
  dev environment has no Docker/Oracle/Redis/Kafka: the real
  `MyBatisOrderRepository`/`RedissonCacheClient`/`KafkaEventPublisher`
  adapters were written (hexagonal ports + fakes, same pattern as
  `sun-moon-python-platform`'s ADR-0008) and compile, but `Bootstrap`
  wires in their in-memory fakes by default and no test exercises the real
  ones. A `docker-compose.yml` (Oracle/Redis/Kafka/Prometheus/Grafana) is
  included for when that infra is actually stood up. Full build (`./gradlew
  clean build`) and all 11 tests pass.
- **Not yet decided** (see ADR-0009/0010): what replaces Spring's DI role
  (manual wiring vs. a lightweight non-Spring DI library), sequencing
  against Phase 1.6 and the reorganized Go/TS/Rust plan, and when (or
  whether) Docker gets installed here to actually verify the Oracle/Redis/
  Kafka adapters live.
- **Build order revised, 2026-09-08**: the plan was REST API → Client
  (device) app next; the owner found Client needs BO (Back Office, the
  internal admin system) to exist first, so BO moves ahead of Client. Two
  scope clarifications came with it: "Client" in this project means a
  connected device/terminal (echoing the owner's own C# POS Client
  production background, `Alignment` ADR-0005), not a SaaS tenant; and
  device connection/handshake handling is explicitly **not** BO's job — a
  separate, not-yet-designed **Device Server** will own that, so BO's
  relationship to devices is CRUD-only (master data, not live connection
  state). BO's minimal scope: operator/admin accounts, Common Code (공통코드)
  CRUD, Device CRUD — added as new `/bo/*` endpoints in
  `sun-moon-java-platform` itself, not a separate service. Auth: session-based
  (`HttpOnly`/`Secure`/`SameSite` cookie), chosen over JWT — an admin
  panel wants immediate revocation, the Core Runtime is still one process
  so JWT's statelessness isn't needed yet, and it fits the existing
  hexagonal ports-+-fakes pattern (`SessionStore` port, `InMemorySessionStore`
  now, `RedisSessionStore` later). Recorded in that repo's own
  `docs/adr/0004-bo-endpoints-session-auth.md`.
- **BO auth built and live-verified, same day**: the 3-tier permission
  model (전체관리자/super-admin bypass, per-screen, per-screen-per-action —
  조회/신규/저장/삭제) plus `POST /bo/auth/login`/`logout`, `GET /bo/auth/me`,
  the `AuthorizedEndpoint` decorator, BCrypt hashing, and startup seeding
  of the first super admin (`BO_ADMIN_USERNAME`/`PASSWORD`) — all built and
  driven over real HTTP in a new `BoAuthTest` (5/5 passing, 16/16 total in
  the repo). Recorded in `docs/adr/0006-bo-auth-built-and-verified.md`.
  Common Code and Device CRUD screens themselves are still unbuilt — only
  the auth/permission layer they'll sit behind exists.
- **Persistence switched from Oracle to PostgreSQL, same day**
  (`docs/adr/0005`): more deployable on Phase 3's free/cheap targets than
  Oracle, and installable natively via `winget` unlike Oracle (which needs
  Docker). PostgreSQL was briefly installed locally to close the
  "not-live-verified" gap immediately, then reconsidered and uninstalled
  — it registers as an always-running Windows Service (unlike JDK/Gradle),
  a footprint that should have been flagged before installing rather than
  after. Live DB verification is now deliberately deferred to deploy time;
  local dev continues on in-memory fakes, same discipline as ADR-0003.
- Both commits pushed to `github.com/schware/sun-moon-java-platform` on
  its `master` branch — deliberately **not** the repo's default `main`
  branch, which already holds a different, pre-existing Spring
  Boot/WAR/Jetty MSA project (order/kds/delivery submodules) unrelated to
  this Claude-Code-driven rebuild. Worth remembering next session: this
  repo currently hosts two unrelated codebases on two branches.
- **Deployment target picked, 2026-09-09** (`docs/adr/0007`, that repo):
  **Render** (app, Docker) **+ Neon** (PostgreSQL) — both researched and
  confirmed genuinely permanently free as of today, not trial credit,
  per the owner's explicit "전부 지속적인 무료로만" requirement. Fly.io
  ruled out (dead since Oct 2024); Render's *own* free Postgres ruled out
  (30-day expiry); Oracle Cloud Always Free noted as a viable, more
  DevOps-signal-heavy alternative but deferred (self-managed, recently
  shrunk, idle-reclaim risk) — see Phase 3's correction above. Sequencing:
  Common Code CRUD → Device CRUD → then the 11-step deployment checklist
  in ADR-0007 (Dockerfile, Neon setup, Render env vars, switching
  `Bootstrap` off the in-memory fakes for prod, enabling the `Secure`
  cookie flag over real HTTPS). Not yet executed — CRUD screens come first.
- **Deployed, 2026-09-09** — the Java platform now runs somewhere other
  than a laptop, which is the concrete thing Phase 3 was always aiming at.
  Same day, in order: Device CRUD (the last BO screen) and a BO/Order port
  split; an as-built **design document** (`docs/DESIGN.md` + `_kr`), whose
  writing surfaced a real defect — endpoints were running on Netty's
  event-loop threads, so wiring the real JDBC adapters would have stalled
  every connection those threads served; that was fixed (bounded worker
  pool, ADR-0010) and proven with a test asserting the handler thread is
  *not* an event loop; then Dockerfile, adapter auto-selection and the
  `Secure` cookie flag (ADR-0011).
- **Deployment target changed to the owner's own Debian server**
  (ADR-0012), superseding the Render+Neon choice above. That research
  assumed no existing infrastructure — but the server already running the
  Spring Boot services is more free (already paid for), never sleeps, and
  can publish more than one port, which Render cannot and which the Socket
  transport will need for a Device Server. Ports were allocated against
  what is actually listening there, not assumed: this platform took
  8083/8084/9090 without touching the Spring services on 8080/8081/8082.
- **What actually ran**: the Dockerfile built on first try (368 MB, on a
  2010-era CPU), and on the server BO login, Common Code and Device CRUD
  (Korean text included), the Order API, the Socket echo and Prometheus
  metrics were all verified live. The four pre-existing services stayed at
  200 throughout. Running in fake-repository mode; two `sudo` steps remain
  (create the database, open UFW), which an automated session cannot do.
- **Still unproven, and worth being plain about**: the 1,000-10,000
  concurrent connection target. A 2010 4-core box shared with four other
  containers is not where that gets tested, and no load test has been run
  anywhere.

- **Undeployed the same week, and split into three repositories,
  2026-09-09** — the port allocation above (8083/8084/9090) was wrong, and
  the way it was wrong turned out to be structural rather than numeric.
  The server's scheme allocates **by service**: 8000 hub, 8080 BO,
  8081-8089 REST APIs, 9011/9021/9031 sockets for Java/Python/C. One
  container binding three slots across three categories cannot be made to
  fit it by renumbering. The container was removed (ADR-0013), and the
  question became "how many containers", not "which port".
- **The answer: a kernel plus two services.**
  [`sun-moon-platform-core`](https://github.com/schware/sun-moon-platform-core)
  is the shared runtime — Netty listener binding, REST routing, the
  off-event-loop execution contract, the MyBatis/Flyway/Micrometer/OTel
  wiring — as a library with no `main`, no ports, and no domain.
  [`sun-moon-platform-bo`](https://github.com/schware/sun-moon-platform-bo-netty)
  is Back Office (8080, LAN-only); `sun-moon-java-platform` `master` keeps
  what faces devices. Each service carries the kernel as a git submodule
  wired through a Gradle composite build — no artifact registry, so no
  publish token to keep alive for a dependency with two consumers on one
  disk (ADR-0014).
- **The boundary was made to hold before any repository was created**,
  which is the part worth keeping: three things leaked downward — the
  runtime bound a socket port itself, the HTTP initializer imported a
  WebSocket echo handler, and the MyBatis config listed four domain
  mappers. Each was a real coupling, each was fixed in place, and the 26
  tests stayed green throughout. Only then was anything moved. BO's
  packages were renamed `com.sunmoon.bo.*` so the direction of dependency
  is visible at every import rather than asserted in a build file.
- **Verified after the split, not just compiled**: BO starts from its own
  `installDist` distribution and round-trips login, session cookie, and
  Common Code CRUD; the three repositories hold 3 + 12 + 11 = 26 tests,
  the same count as before. This narrows ADR-0002's "single runtime"
  premise — Socket/REST/WebSocket/Batch together is still the platform's
  shape, but BO was administration, not workload, and never belonged
  inside it.
- **A regression worth recording**: moving the pre-existing Spring Order
  service from 8080 to 8083 (to free 8080 for BO) without opening 8083 in
  UFW left it up, healthy, answering on `localhost` — and unreachable from
  outside for hours. The symptom does not look like a firewall problem,
  which is exactly why it cost time. Written into `Debian-Setting`'s
  `docs/docker.md` next to the fix.

- **BO reversed to Spring the same day, 2026-09-09** — and this one is
  worth recording precisely because it undoes a decision made hours
  earlier. Spring was dropped platform-wide on 2026-09-06, two days before
  BO was proposed; BO never got its own decision, it inherited one made
  when it did not exist. That inheritance was harmless while BO was
  `/bo/*` routes inside a Netty runtime — Spring MVC cannot share a
  process with raw Netty pipelines, so there was no choice to make. **The
  repository split is what created the choice**, and the comparison was
  one-sided: every mechanism BO hand-wrote (`SessionStore`, cookie
  parsing, `PasswordHasher`, the `AuthorizedEndpoint` decorator) is what
  `spring-boot-starter-security` exists to provide, every gap it still had
  (Operator screen, CORS, distributed sessions) is configuration in
  Spring, and BO has **no throughput requirement at all** — a handful of
  operators. It was paying the full cost of having no framework and
  collecting none of the benefit.
- **What that produced**: `sun-moon-java-platform-bo`, the fourth service
  in the *existing Spring MSA family* on that repo's `main` branch,
  alongside Order/KDS/Delivery. The permission model survived the rewrite
  unchanged — three tiers, `create` and `save` distinct because 신규 and
  저장 are distinct permissions — now expressed as Spring Security
  authorities so each controller states its requirement inline. 19 tests,
  including the README's curl sequence driven through the real filter
  chain. The Netty implementation is archived as
  `sun-moon-platform-bo-netty`.
- **The by-product was the more useful artifact.** "Follow the family's
  conventions" turned out to be unanswerable: they existed only as "copy
  the last service", and copying the wrong sibling is invisible. So the
  umbrella repo now has `docs/CONVENTIONS.md` — every rule written as a
  command that loops over all four services, so drift shows up as an odd
  row. It caught two things immediately: BO had inherited Order's legacy
  package name (`com.sunmoon.platform`, which KDS and Delivery do not
  use), and **the umbrella's submodule pointers were four commits behind
  on all three services** — the four missing commits being exactly the
  conventions being documented.
- **Honest note on the portfolio claim.** The hand-built BO demonstrated
  understanding the mechanisms, and still exists with its reasoning
  intact. What the reversal demonstrates is rarer and harder to fake:
  noticing that a component carried the wrong amount of machinery for its
  job, and undoing a decision from the same day rather than defending it.
  The kernel extracted that morning now has one consumer instead of two,
  and whether it still deserves a separate repository is left open rather
  than answered by reflex.

- **BO got a console, in TypeScript + React, 2026-09-09** — and this is
  the entry that touches Phase 4. BO had been an API with Swagger UI
  standing in for screens: fine for proving endpoints, useless for showing
  an operator what they are allowed to do. I had started a plain HTML/JS
  version and said so in one line without asking; the owner asked what the
  frontend language had been decided as, which was the right question,
  because it had not been decided — it had been assumed. **Phase 4's
  TypeScript Dashboard is the reason it went to TypeScript.** Building the
  console there turns that phase from the most-likely-to-be-cut item into
  an artifact inside a system that already runs.
- **It ships inside BO's own jar**, built by a `node:20-alpine` stage in
  the Dockerfile so the 2010-era deploy host still has no Node on it. The
  reason is not neatness: **BO has no CORS configuration**, so a frontend
  on its own origin could not send the session cookie or the CSRF token
  until one existed. Same origin removes that class of problem rather than
  solving it.
- **One design flaw surfaced and was fixed during the build**: the SPA
  route `/bo/board` and the endpoint `GET /bo/board` were the same URL, so
  refreshing a screen returned JSON. The API moved to `/bo/api/*`, and the
  client routes forward to `index.html` one enumerated path at a time — a
  catch-all would have answered genuine API 404s with a page of HTML.
- **What the console shows is the permission model itself.** The left menu
  lists only screens the operator holds some permission on, home is a
  table of what they may do on each, and buttons that would return 403 are
  disabled. That is presentation only — every controller method still
  carries its own `@PreAuthorize` — but it makes the three-tier model
  visible for the first time, which is worth more in an interview than the
  model existing in code.
- A **공지사항 board** was added as the first screen built end to end this
  way (domain → JDBC → controller → screen), with its author taken from
  the session rather than the request body and a test pinning that. 27
  tests. Verified as far as the login screen rendering and every route
  resolving; the screens behind the login were not clicked through, since
  entering a password is not something an automated session does.

- **The device-facing runtime found its purpose, 2026-09-09** — it had
  been "8083 reserved, business undecided" in its own design document
  since the split. It is the **Device Server**: it holds the terminals'
  WebSocket connections and pushes them order events, which is what raw
  Netty and a 10,000-connection target existed for. The Order service
  gained a life cycle (PLACED → ACCEPTED/REJECTED/EXPIRED → PRODUCED →
  DELIVERING → COMPLETED) with the transitions owned in one place, so the
  different systems arriving in an order nobody controls cannot skip a
  step; an illegal move is a 409 rather than a quiet overwrite.
- **Two business rules that only exist because someone thought about
  failure.** An order nobody accepts expires — and that rule lives in
  Order rather than the Device Server, because it has to hold when the
  Device Server is down. An order placed with no terminal connected is
  refused immediately rather than left to time out — and that one lives in
  the Device Server, because connectedness is only known there. Then a
  third: a terminal that dropped moments ago gets three minutes of grace,
  because "the shop is closed" and "the wifi blinked" are different
  situations and the first rule punished the wrong one.
- **A POS terminal in React/TypeScript**
  (`sun-moon-terminal-pos`), the first of five repositories the owner
  asked to split out. The WebSocket is treated as a nudge rather than as
  the data: a frame means something changed, and the terminal then asks
  what is true. That is what makes a screen unplugged for a minute come
  back correct instead of showing a view assembled from whichever frames
  it caught.
- **Three defects that only running it could find**, and they are the
  entry's real content. Netty compares a WebSocket's whole URI to the
  configured path, so `/ws?deviceId=…` was not recognised as `/ws` —
  invisible while the handler was an echo with no query string. The
  kernel's `JsonResponses` could not serialize an `Instant` at all, which
  turned any endpoint returning a timestamp into a 500. And Order
  publishes plain JSON through Spring's `StringRedisTemplate` while
  Redisson defaults to Kryo, so every event died on "unregistered class
  ID" in a stack trace naming neither service. **No test on either side
  could have caught the last one: each service was correct alone.**
- **Flyway, at the owner's instruction, after schema.sql cost exactly what
  it was documented as costing.** Two order rows written before the life
  cycle existed deserialized with a null timestamp and stopped the expiry
  sweep for every other order, every ten seconds. An idempotent `CREATE`
  can add a column; it cannot reach the rows already there. A migration
  fixed it in one file, and the family convention now points that way.
- **A terminal turned out to be `(store, device id)`, not a device id** —
  found by the owner opening two POS browser windows for two shops, which
  is exactly what the screen exists to let someone do. Keying the
  registry on device id alone made both shops' `pos-01` the same
  terminal: each connection displaced the other, each displacement looked
  like an ordinary dropped socket, each side reconnected, and the two
  windows knocked each other offline in a loop that burned a CPU core
  until stopped by hand. That was not a usage mistake; the model was
  wrong, and the loop was a demonstration of exactly how wrong. Re-keyed
  as `TerminalId(storeId, deviceId)` for the connection and
  `TerminalGroup(storeId, type)` for routing and the grace-period check,
  with a genuinely displaced connection now closed with WebSocket code
  4001 so the losing client stops fighting for an id it no longer holds
  instead of retrying forever.
- **End to end, confirmed live** — not through the app's own account of
  itself. A real WebSocket client connected as `store-01/pos-01`; an
  order placed at a store with no terminal present was auto-rejected in
  well under a second; one placed with that terminal connected stayed
  `PLACED` and the push frame arrived at the client; accepting it through
  the Device Server's proxy set Order's `acceptedBy` to `pos-01`, checked
  by querying Order directly rather than trusting the proxy's reply; and
  re-accepting the same order came back 409. The browser automation tool
  used earlier turned out to sandbox outbound connections to a port other
  than the one it navigated to, which looked like a connection failure
  and was actually the tool, not the app — worth remembering the next
  time a "the browser can't connect" result doesn't match what curl says.
- **Where it stands**: Order publishes to Redis and the Device Server
  subscribes — both verified live, the first time the Redisson adapter has
  ever run.
  [`sun-moon-terminal-pos`](https://github.com/schware/sun-moon-terminal-pos)
  (React + TypeScript) is built, deployed as static files on the `:8000`
  hub, and is deliberately public and unauthenticated — every order in
  this system is a preset sample by design, so there is nothing behind
  that screen worth protecting yet; that boundary gets revisited the day
  a real order exists. KDS, DID and the two channel apps
  (order-placing, delivery) are not built. Also not enforced: the
  owner's rule that a store may have several terminals but only one
  receives orders — it needs BO to know about stores and a
  `receivesOrders` flag, blocked on a service-to-service auth decision
  between the Device Server and BO that has not been made. The owner's
  earlier production design (PUSH-OMS/OMS/RIMS/DV-POS) was shared as
  context and settled two things: BO plays RIMS, and PUSH-OMS existed only
  because that system had no WebSocket — so this one does not need the
  split.
- **What, 2026-09-10**: Built one of the two missing channel apps — the
  order-placing channel — as `sun-moon-java-platform-channel-order`.
  Store select → menu select → order, and that's the whole app. 10 stores
  (1매장..10매장) and 10 menu items per store (1메뉴..10메뉴) are a fixed
  catalog in code; no database, no login — same reasoning as the POS
  channel, nothing here worth protecting. Followed BO's own ADR-0005
  pattern (Spring + React embedded in the same jar, same-origin, no CORS);
  no client-side router either, since the whole UI is two screens reached
  by state, never a URL. Order gained a `menuName` field riding the
  existing `data JSONB` column — no migration needed, unlike `storeId`
  earlier, which needed a real column because it's queried on. The owner
  explicitly excluded two things from today's scope: showing a store as
  CLOSED when its POS terminal isn't live, and the periodic check that
  would drive it (confirmed Spring's own `@Scheduled` — matching Order's
  `AcceptanceTimeout` — is the right idiom here, not the Job/Step/Chunk
  Batch engine; deferred anyway). Added a card on the `:8000` hub linking
  to its own port (8085), same pattern as BO, unlike POS which is static
  files on `:8000` itself. **Not on GitHub yet** — creating a new public
  repo is outside this session's auto-run authorization, so the commit is
  local only; the server got the source copied over directly and was
  built and live-verified there instead.
- **What, 2026-09-10**: Added a menu-registration screen to BO — menu
  code/name/image URL/description, no 대/중/소 size tiers yet. Price isn't
  a plain column the way `deviceType` is on Device — it's a `menu_prices`
  history table, each row covering a date range (null end date = still
  current), and "메뉴 연동" resolves to whichever row's range contains the
  lookup date. Two overlapping ranges for the same menu are rejected
  outright at write time, so there is never a date two rows disagree
  about. Followed BO's existing `schema.sql` convention (hasn't moved to
  Flyway the way Order has) and reused the Device/CommonCode screen
  pattern, but the menu-list-plus-price-history-panel underneath is the
  first master-detail screen in this family.

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
- Java's Platform architecture per ADR-0009/0010: DI-replacement approach
  (Spring is dropped, so manual wiring vs. a lightweight non-Spring DI
  library is still open), and sequencing against Phase 1.6 and the
  reorganized Go/TS/Rust plan. Build tool (Gradle) and repo name
  (`sun-moon-java-platform`) are decided — see the Java section above,
  "Started, 2026-09-06."

## Already decided (for the record)

- GitHub visibility: both `sun-moon-python-platform` and this repo are
  Public, deliberately, including the personal/timeline content here — the
  owner's own current employer is already aware, and public visibility is
  intentionally being used as a commitment device.
- GitHub visibility, extended 2026-09-08: the `sun-moon-java-platform`
  family (Order/KDS/Delivery, plus the archived `-jetty` predecessors) and
  `sun-moon-app` are now also Public, for the same reason — deliberately,
  not by default. Archived repos were unarchived, made public, then
  re-archived (GitHub's API refuses a visibility change on an archived
  repo). `Debian-Setting` stays Private (it documents the home server's
  actual configuration).
