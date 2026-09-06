*🇬🇧 English version: [0006-dual-language-batch-via-shared-contract.md](0006-dual-language-batch-via-shared-contract.md)*

# ADR-0006: Batch 컴포넌트를 두 번 만든다 — C(`sun-moon-c-server`)와 Python(`sun-moon-python-platform`) — 설계 하나, 가능하면 실제 계약도 하나 공유

- **Status**: Accepted — C 쪽은 2026-09-06에 구현 완료(검증 내용은
  [`sun-moon-c-server`의 ADR-0001](https://github.com/schware/sun-moon-c-server/blob/master/docs/adr/0001-batch-job-design_kr.md)과
  이 저장소 `ROADMAP_kr.md`의 Phase 1.5 참고); Python 쪽은 아직 미착수
- **Date**: 2026-09-06
- **Deciders**: 본인

## Context

ADR-0005는 8년간의 Spring Boot Batch 경험을, 이번 전환에서 증명 가능한
가장 깊은 단일 기술 자산으로 짚었고, `sun-moon-python-platform`과
`sun-moon-c-server` 어느 쪽에도 아직 Batch 모양의 컴포넌트가 없다는 것도
표시해뒀다. 이후 본인이 의도를 직접 확인했다: "Batch를 어느 언어로 할지
고르는" 게 아니라, **두 GitHub 저장소 모두에 이 방향을 추가**하는 것 —
`sun-moon-c-server`(어제 만든 C 서버 프레임워크, 이제
https://github.com/schware/sun-moon-c-server 에 올라가 있음)와
`sun-moon-python-platform` 둘 다.

## Decision

같은 개념적 설계 — Spring Batch의 **Job → Step → Chunk (Reader /
Processor / Writer)** 모델 — 을 저장소당 한 번씩, 총 두 번 만들되, 서로
무관한 장난감 배치 잡 두 개를 만드는 대신 `sun-moon-python-platform`의
기존 `order-service` REST 계약(그 저장소의 ADR-0006)을 두 구현의 공유
데이터 소스로 쓴다:

- **Python 쪽(`sun-moon-python-platform`)**: monorepo에 새 `batch-service`를
  추가한다. `order-service`의 기존 공개 REST API(이미 `ai-agent-service`가
  쓰고 있는 것과 같은 이음매 — 그 저장소 README의 "The two seams between
  services" 참고)를 통해 주문 데이터를 읽고, chunk 단위로 처리해서 주기적
  집계(예: 일별 주문 요약)를 만든다. 세부 설계(reader/processor/writer
  인터페이스, Spring Batch의 job repository에 해당하는 실행 이력 추적)는
  ADR-0004에서 Go 컴포넌트에 세운 전례대로, 이 Phase가 실제로 시작될 때
  그 저장소 자체의 ADR로 미룬다.
- **C 쪽(`sun-moon-c-server`)**: 그 저장소 README가 이미 gap으로 표시해둔
  진짜 DB 클라이언트("A real Oracle client behind `core_submit_work()`,
  replacing the `db_demo_service.c` simulation")를 추가한다 — 포트폴리오
  타임라인에 맞게 Oracle 대신 SQLite로 시작 — **그리고** 같은
  `order-service` REST API에서 C용 HTTP 클라이언트(예: libcurl)로 주문
  데이터를 가져와서, Job/Step/Chunk 패턴을 C 관용구로 손으로 구현한다
  (Reader/Processor/Writer 인터페이스 자리에 함수 포인터를 담은 구조체를
  쓴다 — C엔 이런 추상화를 위한 내장 메커니즘이 없으므로).

**C 구현을 독립된 자체 데이터셋이 아니라 같은 REST API로 연결하는 이유**:
이러면 서로 무관한 데모 두 개가 아니라, `sun-moon-python-platform`의
"코드 공유가 아니라 계약(contract)"이라는 architecture(그 저장소의
ADR-0006)가 완전히 다르고 더 오래되고 더 로우레벨인 기술 스택에 걸쳐서도
실제로 성립한다는 걸 증명하는 하나의 통합된 증거가 된다 — 최신 Python
생태계 서비스 두 개 사이에서만이 아니라. 이건 서로 무관한 단일언어 데모
세 개보다 훨씬 강한 포트폴리오 주장이다.

## Alternatives considered

- **Batch를 한 언어만(Python만, 혹은 C만)으로 만든다** — 본인이 명시적으로
  기각함; 둘 중 하나를 고르는 게 아니라 두 방향 다 원함.
- **C 배치 잡을 독립적이고 무관한 자체 데이터셋으로 만든다** — 더
  단순하다(cross-repo HTTP 의존 없음, 그 위에서 만드는 동안
  order-service의 API 계약을 안정적으로 유지할 필요도 없음), 하지만
  가장 강력한 주장(단순한 언어-문법 데모가 아니라, 완전히 다른 기술
  스택에 걸친 실제로 동작하는 polyglot 통합)을 포기하는 것이다.
- **`sun-moon-c-server`를 `sun-moon-python-platform` monorepo에 합친다** —
  기각. 둘은 툴체인이 너무 다르다(CMake/vcpkg vs. uv workspace)는 점에서
  공유 저장소는 마찰만 늘릴 뿐이다; 두 저장소가 실제 HTTP 계약으로 통합돼
  있는 게, 서비스 레벨뿐 아니라 저장소 레벨에서도 같은 "공유 코드 대신
  계약" 원칙을 증명한다.

## Consequences

- 합산 예상 작업량(이 ADR 이전 채팅에서 논의함): 둘 다 합쳐서 파트타임
  기준 약 2.5~3주 — 완전히 독립적으로 만드는 것보다 적다, Job/Step/Chunk
  설계를 *생각*하는 건 한 번만 하면 되고 *코딩*만 두 번 하면 되기 때문.
  그래도 `ROADMAP.md`의 타임라인에 실질적으로 추가되는 작업량이다 — 그
  파일의 수정된 Phase 순서와 갱신된 scope-cutting 메모 참고.
- `sun-moon-c-server`의 새 C HTTP 클라이언트 의존성(libcurl 등)은 자체
  빌드 시스템 연결(CMake)이 필요하다 — 그 저장소가 갖게 될 첫 외부 네트워크
  클라이언트 의존성이다, 지금 vendor하고 있는 것들(cJSON, log.c)을 넘어서.
- `sun-moon-python-platform`의 `order-service` REST API가 이제 *두 가지*가
  아니라 그 이상(`ai-agent-service`, 그리고 곧 Python `batch-service`와
  C 배치 잡 둘 다)에 의존되는 계약이 된다 — 세 곳을 동시에 안 깨고는 그
  API 모양을 함부로 못 바꾸게 된다는 뜻인데, 이건 그 자체로 나중에 쓸 수
  있는 "계약엔 왜 버전 관리 규율이 필요한지"에 대한 현실적이고 증명
  가능한 이야기가 된다.
- ADR-0003의 "여러 프로젝트로 흩어지지 않고 Platform 하나"라는 전략을
  희석시키기보다 오히려 강화한다 — Batch 구현 둘 다 이미 계획돼 있던
  저장소에, 이미 존재하는 계약을 중심으로 추가되는 것이지, 무관한 세
  번째 프로젝트가 아니다.

## References

- ADR-0004, ADR-0005 (이 저장소) — Batch를 애초에 우선순위에 두게 만든
  기술 논지와 경력 타임라인 근거.
- `sun-moon-python-platform/docs/adr/0006-inter-service-communication-http-and-redis_kr.md` —
  세 번째(그리고 저장소를 넘나드는 네 번째) 소비자로 확장되는 REST 계약.
- `sun-moon-c-server/README.md`의 "Planned next steps" — 이 ADR이
  해결하는, 이미 독립적으로 표시돼 있던 진짜 DB 클라이언트 gap.
