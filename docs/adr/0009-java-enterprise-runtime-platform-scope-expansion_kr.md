*🇬🇧 English version: [0009-java-enterprise-runtime-platform-scope-expansion.md](0009-java-enterprise-runtime-platform-scope-expansion.md)*

# ADR-0009: Java — Batch 전용 reference implementation에서 Netty 기반 전체 Enterprise Runtime Platform으로 범위 확장

- **Status**: Ongoing (방향성 설정 중) — 오늘은 형태와 후보 스택을 기록해두는
  단계이고, 아키텍처 자체는 의도적으로 아직 열려 있음. ADR-0001이 "Ongoing"을
  하나의 답으로 닫아버리기보다 시간을 두고 계속 다시 다루는 주제에 쓰는 것과
  같은 방식
- **Date**: 2026-09-06
- **Deciders**: 프로젝트 오너(스택과 Platform 형태를 제안함); Claude(추가
  제안과 아래의 범위 변화를 짚음)

## Context

ADR-0008은 Java를 좁게 잡았다 — 그 안의 item 2는 Java를 오직 **Spring Batch
reference implementation**으로만 제안했다. 이미 만들어진 Python
(`batch-service`)과 C(`batch_runner`) 버전에 대한 3자 Job/Step/Chunk 비교의
세 번째 자리 — 새로운 Platform이 아니라 세 번째 Batch 데모였을 뿐이다.

오늘 본인은 훨씬 더 큰 형태를 가지고 왔다: Java, DDD 기반의 **Enterprise
Runtime Platform**으로, Socket, REST API, WebSocket, Batch Job을 단일
Runtime에서 함께 운영하고, OpenSource를 기본으로 하며, 동시 접속자
1,000~10,000명을 목표로 한다. 구조적으로 이것은 `sun-moon-c-server`와 같은
종류의 야심이다(하나의 Runtime, 여러 Transport) — ADR-0008이 잡았던 좁은
Batch 전용 비교가 아니다. 본인은 후보 스택을 제안하며 추가 오픈소스 추천과
방향성을 함께 잡아달라고 요청했고, "고민하다가 정리한 것"이라며 아직 정리
중임을 명시적으로 밝혔다.

## Decision

오늘 제안된 스택, Claude의 추가 제안, 아래의 범위 관찰을 기록한다 —
아키텍처를 확정 짓지는 않는다. 이 ADR은 의도적으로 "Ongoing" 상태로 남겨둔다.
이 프로젝트 자체의 기존 패턴과 같은 방식이다(ADR-0001,
[[feedback-avoid-confirmed-exploratory-framing]] 참고): 방향 전환 자체(Java가
Batch 전용 데모 이상을 갖게 됨)는 지금 기록해둘 가치가 있지만, 구체적인
컴포넌트 경계는 한 번에 닫는 게이트가 아니라 계속 이어지는 방향성 설정
과정이다.

**오늘 제시된 오너의 제안 스택:**

- Netty — Core Runtime / Transport Engine
- Quartz — Batch Scheduler
- MyBatis + Oracle — Persistence Layer
- Redis — Cache / Session / Lock
- Kafka — Event Bus
- Logback — Service별 Logging
- Micrometer / Prometheus / Grafana — Monitoring
- OpenTelemetry — Distributed Tracing

**Claude의 추가 제안과 근거:**

- **Spring (Core DI + Spring Batch — Spring Boot의 내장 Tomcat은 아님).**
  이건 ADR-0008의 원래 요점과 바로 연결된다: Quartz 혼자서는
  Job/Step/Chunk/Reader/Processor/Writer 모델이 없다 — cron 방식 트리거일
  뿐이다. 그 모델을 실제로 제공하는 건 Spring Batch이고, ADR-0008이
  원했던 "정답" 기준선이 되는 이유도 정확히 그것 때문이다 — 본인이 8년의
  실전 경험을 가진 바로 그 분야이기 때문(ADR-0005). 자연스러운 조합은
  Quartz가 Spring Batch job을 *트리거*하는 것이지, Quartz가 Spring
  Batch를 대체하는 게 아니다.
- **Reactor Netty / Spring WebFlux** — REST와 WebSocket이 raw Netty 위에
  손수 만든 라우터가 아니라, Netty의 event loop를 Spring이 관리하는
  서비스들과 공유해야 한다면 고려할 선택지. 아래에서 열린 질문으로 남김.
- **HikariCP** — connection pool. MyBatis + Oracle 조합에서는 실질적으로
  거의 필수.
- **Jackson** — JSON 직렬화. 이 스택에서 사실상 표준.
- **Jakarta Bean Validation (Hibernate Validator)** — Transport 경계에서의
  요청/DTO 검증.
- **Resilience4j** — circuit breaker / retry / rate limiter. 1,000~10,000
  동시 접속 목표와 직접 관련.
- **Redisson** — 손수 만든 Redis 락 대신, 검증된 distributed-lock 시맨틱을
  가진 유지보수되는 Redis client.
- **Flyway** — Oracle 스키마 마이그레이션. Python 쪽에서 이미 지적된 Alembic
  공백(`ROADMAP.md` Phase 1)과 대응.
- **JUnit5 + Mockito + Testcontainers** — 단위 테스트뿐 아니라 "실제 인스턴스
  대상으로 검증"하는, Python/C Batch 작업에서 이미 쓴 패턴과 일치
  (ROADMAP.md Phase 1/1.5).

## 범위 관찰 (핵심 open question)

오늘 본인이 설명한 것은 ADR-0008의 item 2보다 구조적으로 훨씬 크다. 그
항목은 Java를 기존 Python/C Batch 작업 옆에 놓이는 Batch 전용 reference
implementation으로 좁게 잡았다. 지금 테이블 위에 있는 것 — Socket + REST +
WebSocket + Batch를 함께 호스팅하는 Netty Core Runtime — 은
`sun-moon-c-server` 자체와 같은 종류의 야심이다: 세 번째에 얹히는 네 번째
Batch 데모가 아니라, 세 번째 완전한 Platform 재구현이다.

이게 반드시 잘못된 방향이라는 뜻은 아니다 — Java는 8년의 실전 경험이라는
실체가 있고(ADR-0005), ADR-0008 자체의 논리에 따르면 이 로드맵 전체에서
신규 습득 리스크가 가장 낮은 언어다. 하지만 이건 ADR-0003의 scope-cutting
원칙을 다시 여는 일이기도 하다: Phase 3는 이미 원래 Q4 2026 타겟을 넘기고
있고, Phase 4(TypeScript Dashboard)는 시간이 부족해지면 가장 먼저 잘릴
후보로 이미 표시돼 있다. 전체 Java Platform을 추가하는 건 ADR-0008이
제안했던 Batch 전용 버전보다 전체 포트폴리오 표면적에 더 큰 추가다.

## Alternatives considered

- **Java를 Batch 전용으로 좁게 유지** (ADR-0008 원래의 item 2) — 더 빠르고
  리스크가 낮으며 이미 제안돼 있음; 채택하지 않음 — 오늘 본인의 메시지가
  이미 이 범위를 넘어섰기 때문. ADR-0008의 좁은 프레이밍을 아무 이의 없이
  그대로 두는 대신, 이 변화를 여기에 기록해둠.
- **이번 턴에서 전체 아키텍처를 결정** (DI 방식, 라우팅 레이어, repo 이름,
  Phase 1.6 및 재정리된 Go/TS/Rust 계획과의 순서) — 채택하지 않음. 이
  프로젝트의 기존 패턴과 일치 — 제안하고, 오너의 확인을 받은 다음에야
  ADR이 "Ongoing"/"Proposed"에서 "Accepted"로 넘어간다.

## Consequences

- `ROADMAP.md`의 "제안됨 (아직 일정 없음) — Java" 섹션을 이 ADR과 함께
  업데이트해서 확장된 스택과 이 범위 관찰을 반영한다 — 여전히 일정 없음,
  아직 목표 날짜는 없음.
- 본인이 전체 Platform 범위를 확정하면, 후속 ADR(또는 이 ADR이 Accepted로
  전환)이 다음을 기록해야 한다: repo 이름(후보: `sun-moon-java-platform`,
  `sun-moon-python-platform` / `sun-moon-c-server` 명명 규칙과 일치), DI/
  라우팅 아키텍처 결정, Phase 1.6 및 재정리된 Go/TS/Rust 계획과의 순서.
- **아직 결정 안 됨**: DI container 방식(Netty가 이미 transport인 상황에서
  Spring Core를 쓸지 말지); REST/WS 라우팅이 raw Netty 위에 있을지 Reactor
  Netty/Spring WebFlux 위에 있을지; build tool(Gradle vs Maven); repo
  이름; 그리고 이미 "다음 세션"에 대기 중인 다른 항목들(Phase 1.6, 재정리된
  Go/TS/Rust 계획)과 어떻게 순서를 맞출지.

## References

- ADR-0008 (this repo) — 이 ADR이 확장하는, 더 좁았던 원래 Java 제안.
- ADR-0003(scope-cutting 원칙), ADR-0004(언어 간 기술적 논지), ADR-0005(8년
  Java/Spring Boot 기준선) — 모두 이 repo.
- `sun-moon-c-server`의 아키텍처(하나의 Runtime, 여러 Transport)가 여기서
  제안되는 것의 구조적 선례.
