*🇬🇧 English version: [0008-pause-go-ts-rust-add-java-cpp.md](0008-pause-go-ts-rust-add-java-cpp.md)*

# ADR-0008: Go/TypeScript/Rust 재계획은 다음 세션으로 미루고, Java와 C++을 로드맵의 정식 항목으로 추가한다

- **Status**: Partially Accepted — 미루는 것(아래 1번)은 본인의 명시적
  지시대로 Accepted; Java/C++ 항목(2번, 3번)은 **Proposed**, 본인 확인 대기 중
- **Date**: 2026-09-06
- **Deciders**: 본인(1번); Claude 제안, 확인 대기 중(2~3번)

## Context

본인이 한 메시지에서 두 가지를 했다: (a) Go/TypeScript/Rust 로드맵
계획을 지금이 아니라 다음 세션(내일)에 다시 이야기하자고 했다, 먼저
직접 생각을 정리하기 위해서 — 이건 무기한 정지가 **아니라**, 이미
예정된 Phase 1.6(`sun-moon-c-server` 리팩터링)과 같은 시점으로 딱 한
세션만 미루는 것이다; (b) Java와 C++이 — 본인 표현으로 "가장 중요한
언어" — 정작 자기 로드맵 항목을 가진 적이 없었다고 짚었다. 두 언어 다
지금까지는 다른 결정의 *배경 근거*로만 등장했다: Java/Spring Boot는
ADR-0005(Batch를 우선순위에 두게 만든 8년 타임라인)에서, C++은
ADR-0004(IOCP/completion-callback 논지)에서. Go(Phase 2)나
TypeScript(Phase 4)처럼 "이걸 증명하기 위해 실제로 뭘 만든다"는 구체적
항목은 둘 다 없었다.

본인은 이걸 사과("변덕쟁이 모드")로 열었고, 별도로 이 ADR의 초안이
Go/TS/Rust를 미루는 걸 기한 없는 정지로 과하게 표현한 걸 대화 중간에
직접 바로잡았다. 명확히 짚어둘 만하다: 둘 다 사과하거나 과장할 일이
아니다 — 이해가 깊어지면서 계획을 다시 짜는 건 이 프로젝트 자체의
기록된 작업 방식([[user-additive-collaboration-style]])과 맞고,
"내일 다시"는 구체적으로 예정된 시점이지 무기한 보류가 아니다.

## Decision

1. **(Accepted) Go/TypeScript/Rust는 다음 세션에 다시 이야기한다.**
   `ROADMAP.md`의 기존 Phase 2(Go)와 Phase 4(TypeScript Dashboard) 항목을
   "다음 세션에 재검토"로 표시한다 — 내용은 오늘의 출발점으로 남기고
   삭제하거나 틀렸다고 판단하지 않지만, 그 세션이 오기 전까지는 현재
   유효한/확정된 것으로 취급하지도 않는다. Rust(한 번도 Phase로 잡힌
   적 없음, ADR-0004에서 Go보다 뒤로만 미뤄짐)도 같은 방식으로 미룬다.
   이미 같은 다음 세션으로 예정된 Phase 1.6과 시점이 맞는다.

2. **(Proposed) Java: 3자 Batch 비교를 완성한다.** 같은 Job/Step/Chunk
   설계를 세 번째로, 실제 Java/Spring Boot로 만든다(손수 만든 엔진이
   아니라 아마 진짜 Spring Batch로 — Python/C와 달리 Java는 이미 이걸
   위한 정석 프레임워크를 갖고 있다). Python(`batch-service`)과
   C(`batch_runner`)는 이미 구현돼 있다; Java는 **reference
   implementation**이 될 것이다 — 동등한 세 번째 데모가 아니라, 나머지
   둘을 검증하는 "정답" 기준선 — Spring Batch야말로 본인이 8년의 실전
   판단력을 가진 분야이기 때문이다(ADR-0005). 이게 그 전문성을 글로만
   주장하는 대신 포트폴리오가 실제로 *보여주게* 만드는 가장 직접적인
   방법이다.

3. **(Proposed) C++: 원래 IOCP 문제를 직접 푸는, 범위를 좁힌 C++20
   coroutine 데모.** ADR-0004의 논지는 C++20 coroutine(`co_await` +
   Boost.Asio나 cppcoro 같은 라이브러리)을 백화점 시스템의
   completion-callback 파편화 문제에 대한 같은 언어 안에서의 해법으로
   이름 붙였다 — `sun-moon-c-server`의 순수 C `batch_runner`(coroutine도
   class도 없음)와는 다른 것. 같은 IOCP 메커니즘 위에서 선형으로 읽히는
   비동기 코드와 깔끔한 객체 분해를 보여주는 작고 범위가 명확한 데모는,
   Go CSP 유비 하나만 있는 것보다 "실제 언어로, 실제 문제에, 실제
   고친 것"이라는 더 날카로운 증거물이 된다.

## Alternatives considered

- **Java/C++도 Go/Rust처럼 새 언어 습득 항목으로 다룬다** — 기각: Java와
  C++은 *배워야 할* 스킬이 아니라 *증명해야 할* 16년치 기존 깊이다.
  이건 다른 종류의 로드맵 항목(습득이 아니라 전시)이고 같은 방식으로
  계획하면 안 된다.
- **Java/C++을 배경 서사로만 남겨두고**(ADR-0004/ADR-0005에서의 역할)
  **더 이상 아무것도 안 한다** — 이게 본인이 명시적으로 바꿔달라고 한
  현상유지였다; 직접 지시로 기각.
- **Java/C++ 범위를 일방적으로 정하고 지금 바로 만들기 시작한다** — 채택
  안 함. 1번(명시적으로 지시받음)과 달리 2~3번은 Claude의 제안이다;
  계획을 추천하자마자 바로 만들기 시작하면, 이런 종류의 범위 추가가
  받아야 할 본인의 확인 단계를 건너뛰게 된다(제안 → 확인 → 이후 ADR/코드
  작성이라는 이 프로젝트의 기존 패턴대로).
- **1번을 기한 없는 정지로 표현한다** — 이건 이 ADR 자체의 첫 초안이었고,
  본인이 대화 중간에 바로잡았다: 구체적인 다음 단계(내일)가 있는 한
  세션짜리 유예이지, 접어둔 주제가 아니다.

## Consequences

- 2~3번이 확정되면: `alignment`의 ADR-0006(이중 언어 Batch)에 후속
  메모가 필요할 것이다 — Batch 비교가 3자 구도가 됐고, Java는 동등한
  구현체가 아니라 reference라는 걸 명시해야 한다.
- C++20-coroutine 데모는 `sun-moon-c-server`의 기존 순수 C
  `batch_runner`와는 별개의 새 작업이다 — 그걸 대체하지 않고 포트폴리오
  전체 면적에 더해진다. ADR-0003의 scope-cutting 규율이 여전히 적용된다.
- 아직 안 정한 것: Java와 C++ 항목의 정확한 범위와, 재정리될
  Go/TS/Rust 계획 및 이미 예정된 Phase 1.6(C 서비스 기반 리팩터링) 대비
  목표 시점 — 전부 다음 세션 논의 주제이고, 이 ADR의 Proposed 항목이
  Accepted로 바뀌고 `ROADMAP.md`에 날짜가 붙기 전에 본인 확인이 필요하다.
- 다음 세션은 이미 세 가지가 예정돼 있다: Phase 1.6(C 서비스
  리팩터링), 재정리된 Go/TypeScript/Rust 계획, 그리고 위 Java/C++ 제안
  확정 여부 — 셋 다 한 번에 다루기보다 순서를 의도적으로 정할 만하다.

## References

- ADR-0004, ADR-0005, ADR-0006 (이 저장소).
- [[user-additive-collaboration-style]] (memory) — 중간에 계획을 다시
  짜는 게 왜 여기서는 문제가 아닌지.
