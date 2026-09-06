*🇬🇧 English version: [0004-core-technical-thesis-decouple-io-completion-from-domain-logic.md](0004-core-technical-thesis-decouple-io-completion-from-domain-logic.md)*

# ADR-0004: 핵심 기술 논지 — completion 기반 I/O와 domain logic을 분리한다 (IOCP에서 얻은 교훈)

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: 본인

## Context

본인의 과거 C/C++ 작업(여러 백화점/쇼핑몰이 접속하는 멀티테넌트 서버)에
대한 가장 뚜렷한 불만족은 비동기 I/O 실력 부족이 아니었다 — IOCP(I/O
Completion Port)를 썼는데, 이건 그 문제에 대해 실제로 옳고 고성능인
선택이다. 불만족은 구조적인 것이었다: 비즈니스 로직이 별도의 completion
콜백 함수들(connect/read-complete/write-complete/disconnect류 핸들러)에
흩어져버렸고, 이 때문에 제대로 된 객체 분해가 어려워졌으며, 결국 너무 많은
코드가 하나의 큰 source에 몰리게 됐다. IOCP가 준 성능 이점(I/O가 언제
끝나는지 정확히 아는 것)은 진짜였지만, I/O 이벤트를 중심으로 코드를
조직하는 구조적 비용을 상쇄하지 못했다.

## Decision

이걸 프로젝트의 핵심 기술 논지로 채택하고, 새로 배우려는 모든 언어(ADR-0002/
ROADMAP: Go, Rust, TypeScript, JavaScript)를 관통하는 하나의 실로 의도적으로
삼는다 — 각 언어의 동시성 모델을 서로 무관한 암기 대상으로 다루지 않는다:

- **Python(2026-09-06에 이미 적용함)**: `asyncio`의 `async`/`await`가 같은
  completion 기반 메커니즘(Linux의 epoll) 위에서 선형으로 읽히는 코드를
  준다. 더 중요한 건 `sun-moon-python-platform`의 `AggregateRoot`(예:
  `Order`)가 의도적으로 I/O를 전혀 모르게 돼 있다는 것 — interfaces/
  infrastructure 계층이 모든 completion/네트워킹 메커니즘을 갖고,
  application 계층이 오케스트레이션하고, domain 계층은 그걸 조립하는 데
  I/O 왕복이 몇 번 걸렸든 상관없이 모든 비즈니스 규칙을 응집된 객체 하나로
  갖는다. 이게 "I/O 핸들러에 흩어진 비즈니스 로직" 문제에 대한 직접적인
  architecture 답이다.
- **C#(이미 알고 있음)**: `async`/`await`가 같은 모델이다, 이미 실무로
  써본 언어에서 — Go로 넘어가기 전에 이 교훈을 확인하는 빠르고 저위험한
  체크포인트다.
- **Go(Phase 2 타겟)**: 다르고, 어쩌면 더 우아한 답 — goroutine +
  channel(CSP). Go 런타임의 네트워크 poller는 실제로 Windows에서 IOCP를,
  Linux에서 epoll을 그대로 감싸고 있다. goroutine 안에서는 블로킹처럼
  보이는 read를 그냥 부르면, 런타임이 OS 스레드를 막지 않고 알아서
  처리한다 — 그래서 비즈니스 로직은 애초에 "async 색깔"이 붙은 함수일
  필요가 없다. IOCP를 직접 다뤄본 사람에게는 기계적으로 가장 가까운
  사촌이라 아마 가장 직관적으로 와닿을 동시성 모델이다.
- **Rust / TypeScript / JavaScript**: 다시 async/await 계열, 각자 자기
  state machine/event loop 구현 위에서 같은 개념을 구현한다.
- **C/C++ 자체**: 원래 문제에 대한 같은 언어 안에서의 직접적인 해법은
  C++20 coroutine(`co_await`) + 현대적인 비동기 I/O 라이브러리(예:
  Boost.Asio의 coroutine 지원, cppcoro)다 — 즉 원래 시스템의 근본 원인은
  IOCP가 아니라, 그 당시 도구들이 강제했던 coroutine 이전의 콜백 스타일이었을
  가능성이 높다.

## Alternatives considered

- **각 언어의 동시성 모델을 서로 무관한 잡학으로 다룬다** — 기각. 본인의
  진단이 이미 관통하는 실을 찾아냈다. 이걸 무시하면 이 계획 전체에서 가장
  값진, 힘들게 얻은 직관을 낭비하는 것이다.
- **원래 백화점 C++ 시스템을 coroutine으로 다시 짜서 포트폴리오 프로젝트로
  삼는다** — 검토했지만, 그 코드베이스는 십중팔구 회사 소유의 proprietary
  코드라 공개 포트폴리오에 쓸 수 없다. 같은 교훈을 Platform의 Go
  컴포넌트(Phase 2)에서 새로 증명하는 게 완전히 유효하고 이 문제를 피한다.

## Consequences

- Phase 2(`sun-moon-python-platform`의 Go gateway/transport 컴포넌트)가
  "폴리글랏 증명"보다 더 날카로운 근거를 갖게 된다 — 실제로 겪은 구체적인
  전문적 좌절에 대한 의도적이고 직접적인 반박이 되고, 이건 인터뷰에서
  뻔한 "저도 Go 할 줄 알아요"보다 훨씬 기억에 남는 이야기로 읽힌다.
- 그 Go 컴포넌트 자체의 ADR(Phase 2 시작할 때 `sun-moon-python-platform`에
  쓸)에 짧은 비교 메모를 넣어야 한다 — goroutine vs. 원래 백화점
  시스템의 completion-callback 스타일. proprietary 코드를 노출하지
  않으면서 개인적 좌절을 증명 가능한 성장으로 바꾼다.
- ADR-0002의 언어 목록에 미세하게 가중치를 다시 준다: C#의 async/await를
  다시 보는 건 거의 공짜다(이미 알고 있으므로) — Go 가기 전 짧은 몸풀기로
  해볼 만하다; Rust의 async/await는 기다려도 된다, Go의 CSP 모델이 먼저
  증명해야 할 더 대조적이고 우선순위 높은 교훈이기 때문이다.

## References

- 이 대화, 2026-09-06 — 백화점/쇼핑몰 IOCP 시스템이 원 사례.
- 여기서 이름 붙인 일반 패턴: completion-callback / continuation-passing
  스타일 비동기 I/O 아래에서의 비즈니스 로직 파편화(비공식적으로 async I/O
  문헌에서 "callback hell" 혹은 "stack ripping"이라 부르기도 함).
- Tony Hoare, *Communicating Sequential Processes* (1978) — Go의
  goroutine + channel 모델의 이론적 뿌리.
- `sun-moon-python-platform/docs/adr/0003-domain-driven-design-for-business-logic_kr.md` —
  위에서 말한 "domain logic은 I/O를 모른다"의 구체적 구현.
