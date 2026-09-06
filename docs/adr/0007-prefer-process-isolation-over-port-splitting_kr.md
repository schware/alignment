*🇬🇧 English version: [0007-prefer-process-isolation-over-port-splitting.md](0007-prefer-process-isolation-over-port-splitting.md)*

# ADR-0007: 서비스 하나의 트래픽을 여러 Port로 나누기보다 Process/Service 분리(그리고 worker-pool 분리)를 우선한다; C 쪽에 맞춰 리팩터링 예정

- **Status**: Accepted (Python 쪽); C 쪽 리팩터링은 예정됨, 아직 시작 안 함
- **Date**: 2026-09-06
- **Deciders**: 본인

## Context

본인이 `order-service`의 TCP 트래픽을 여러 port로 나눠서 업무끼리
격리할지(느리거나 무거운 업무 하나가 다른 걸 못 느려지게), 아니면 port는
서비스당 하나로 두고 업무 분리는 서비스 레벨에서 할지를 물었다 — 트레이드
오프를 스스로 정확히 짚었다: port 분리엔 진짜 회복력(resilience)
동기가 있지만, "관리적인 측면에서 매력도가 떨어지는 것은 사실"이라고.

## Decision

**하나의 서비스 안에서 트래픽을 여러 port로 나누는 건 이 목적에 맞는
도구가 아니다.** 같은 프로세스 안의 port 두 개는 여전히 같은 이벤트
루프/CPU/메모리를 공유한다 — port 경계만으로는 진짜 격리가 안 된다.
본인이 실제로 원하는 격리는 이 프로젝트에서 이미 쓰고 있는 다른 두
곳에서 나온다:

1. **Process/서비스 경계** — `order-service`, `ai-agent-service`,
   `batch-service`는 이미 별도 프로세스다(`sun-moon-python-platform`의
   ADR-0005). 느리거나 과부하된 `order-service`가 `ai-agent-service`의
   응답성을 떨어뜨릴 수 없다; 이게 이미 본인이 요청한 굵은 단위의
   격리다.
2. **프로세스 안에서의 worker-pool 분리** — 느리거나 블로킹하는 작업이
   서비스 자체의 응답성을 해치지 않게 격리하는 용도로,
   `sun-moon-c-server`의 `core_submit_work()`가 이미 보여준 패턴과
   같다(네트워크 I/O 스레드는 DB류 작업에 절대 안 막히고, 별도
   스레드풀로 넘어간다). Python의 대응은 블로킹 작업을 이벤트 루프에서
   바로 돌리지 않고 별도 task/스레드로 넘기는 것이다.

**짚어두지만 이번엔 채택하지 않는, 좁고 정당한 예외 하나**: control
plane vs. data plane port 분리(예: 업무 트래픽과 분리된 전용
health-check/metrics port, Kubernetes의 liveness probe가 흔히 이렇게
함)는 "업무를 port로 나눈다"와는 다른, 실제로 존재하는 패턴이다 — 이번
결정의 범위는 아니지만, 나중에 구체적인 필요가 생기면 별개의 선택지로
기억해둘 만하다.

**Python(`sun-moon-python-platform`)은 코드 변경이 필요 없다** — 이미
있는 서비스당 port 하나 모델이 이 결정을 그대로 반영하고 있다.

**`sun-moon-c-server`는 리팩터링이 예정됐다**(다음 세션, 본인의 토큰
예산이 초기화되면): 여러 전용 port에 걸쳐 config로 여러 모드(tcp/http/udp)를
스레드로 돌리는 `server` 바이너리 하나짜리 구조에서, 서비스당 프로세스
하나 모델(`sun-moon-python-platform`을 거울삼아)로 옮겨간다 — 지금처럼
여러 port/모드에 흩어지는 대신 서비스마다 일관된 port 하나로 통일한다.
`ROADMAP.md`의 새 Phase 1.6 참고.

## Alternatives considered

- **`order-service`의 TCP 트래픽을 업무 종류별로 여러 port로 나눈다** —
  본인이 원래 저울질하던 옵션. 기각: 실행 컨텍스트를 같이 분리하지
  않고는 목표(느린 작업 격리)를 달성 못 하고, 본인 스스로 이미 운영
  비용을 짚었다.
- **`sun-moon-c-server` architecture를 그대로 둔다** — 검토했지만,
  본인은 `sun-moon-python-platform`이 이미 보여준 것과 같은
  서비스-분해 판단력을 C에서도 증명하고 싶어 한다, ADR-0004의 "architecture
  원칙은 언어를 넘어 전이된다"는 논지를 Job/Step/Chunk에서 서비스 분해
  자체로 확장하는 것. 완전히 기각한 게 아니라 다음 세션으로 미룸.

## Consequences

- 지금 당장은 어디에도 코드 변경이 없다 — 이 ADR은 방향을 기록하고,
  후속 작업 하나를 예정에 넣을 뿐이다.
- `sun-moon-c-server`의 리팩터링은 만만치 않다: 지금은 CMake 타겟
  하나(`server`)가 여러 port(HTTP 8081, TCP 8090, UDP 8082, HTTPS 8443,
  9010~9020 TCP-range 데모)에 걸쳐 config로 모드를 고르고, 이미 분리돼
  있는 `batch_runner`가 별도로 있다. 업무를 독립 바이너리/프로세스로
  쪼개는 건 `main.c`, `config.c`, `CMakeLists.txt`, 아마 `config/*.json`
  파일들까지 건드린다 — 빠른 편집이 아니라 진짜 범위가 있는 작업이다.
  작업이 시작되면 이 프로젝트의 기존 관행대로 그 저장소 자체 ADR이
  필요하다.
- 언젠가의 C 리팩터링이 포트폴리오 서사에 "원칙이 언어를 넘어
  전이된다"(ADR-0004의 논지)는 두 번째 구체적 증거(Job/Step/Chunk에
  이어)를 준다.

## References

- 이 대화, 2026-09-06.
- `sun-moon-python-platform/docs/adr/0005-monorepo-of-independent-services-via-uv-workspace_kr.md` —
  Python에는 이미 확인됐고 C에는 제안된 서비스 분리 패턴.
- `sun-moon-c-server/README.md`의 `core_submit_work()` 설명 — 추가
  port 대신 느린 작업을 격리하는 올바른 도구로 이 ADR이 가리키는 기존
  worker-pool 분리 패턴.
- `alignment/docs/adr/0004-core-technical-thesis-decouple-io-completion-from-domain-logic_kr.md` —
  이게 확장하는 "원칙은 언어를 넘어 전이된다"는 논지.
