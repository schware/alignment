*🇬🇧 English version: [0001-use-adrs-for-career-decisions.md](0001-use-adrs-for-career-decisions.md)*

# ADR-0001: `sun-moon-python-platform`이 architecture 결정을 ADR로 남기는 것처럼, 이직 준비 과정도 ADR로 문서화한다

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: 본인

## Context

본인은 이직을 준비하면서, 그 과정의 주된 vehicle로 `sun-moon-python-platform`
(원래 `Network` 작업에서 시작된) Platform 프로젝트를 만들고 있다. 지금
두 종류의 결정이 동시에 일어나고 있다: 기술적 결정(무엇을 만들지, 어떤
컴포넌트에 어떤 언어를 쓸지)과 커리어 결정(어떤 스킬을 우선순위에 둘지,
어떤 직무를 타겟할지, 시간을 어디에 얼마나 쓸지). 기술 쪽은 이미 문서화
관행이 있다 — `sun-moon-python-platform/docs/adr/`의 ADR들. 커리어 쪽은
없었고, 커리어 관련 판단의 근거야말로 압박 속에서 잊어버리거나(예: 인터뷰
직전, 6개월 뒤 "왜 그때 Rust 대신 Go를 골랐지" 하고 다시 고민하게 되는 것)
다시 논쟁거리가 되기 쉬운 것들이다.

## Decision

같은 ADR 형식(Context/Decision/Alternatives considered/Consequences/
References — `0000-template.md` 참고)을 이 저장소의 커리어/포트폴리오
전략 결정에도 쓴다. `sun-moon-python-platform`의 기술 ADR과는 별도로
관리하되, 관련 있을 때는 명시적으로 서로 참조한다. 이 저장소는 코드베이스가
아니다 — 그 저장소가 시스템 architecture의 근거를 담는 것처럼, 여기는
커리어 전략의 근거를 담기 위해 존재한다.

기술 ADR에는 없던 Status 값 하나를 추가한다: **Ongoing** — 한 시점에
하나의 답으로 결론짓기보다 의도적으로 계속 열어두고 다듬어가는 결정에 쓴다.
이건 본인이 이미 일부 기술 방향에 대해 선호하는 방식과 같다
(`sun-moon-python-platform`의 ADR-0009, "confirmed-vs-exploratory가 아니라
direction-setting"이라는 프레이밍 참고).

## Alternatives considered

- **번호 매긴 ADR 대신 하나의 러닝 노트 문서** — 시작은 더 쉽지만, 다른
  저장소의 ADR-0001에서도 짚었듯 문서 하나로는 "아직 유효한 근거"와
  "이미 뒤집힌 근거"를 구분하기 어렵다. 커리어 방향은 여러 달에 걸쳐
  기술 방향 못지않게(어쩌면 더) 자주 바뀔 가능성이 있어서 같은 문제가,
  어쩌면 더 심하게 적용된다.
- **아무 문서화도 안 하고 그냥 만들면서 본다** — 본인이 명시적으로 이
  과정을 문서화하고 싶다고 했으므로 실제 선택지는 아니었다. 다만 빠르게
  움직이고 자연스러운 체크포인트에서만 되돌아보는 걸 선호하는 사람에게는
  합리적인 대안이라 여기 기록만 해둔다.

## Consequences

- 여기 쓰는 모든 ADR은 나중에 인터뷰 답변과 이력서/포트폴리오 서술의
  원재료가 된다 — "왜 이렇게 만들었나요" 질문에 인터뷰 압박 속에서
  기억을 재구성하는 대신, 날짜 찍힌 정직한 기록을 근거로 답할 수 있다.
- 이미 빠듯한 타임라인(ADR-0002 참고) 위에 글쓰기 부담이 더 얹힌다 — 실제로
  아무것도 안 만들면서 "문서만 쓰며 생산적인 척"하게 되지 않도록 주의할
  필요가 있다. 대략적인 기준: 방향이 실제로 바뀔 때 ADR을 쓰고, 사소한
  작업 하나하나에는 쓰지 않는다.
- 이 저장소엔 개인/타임라인 관련 민감한 정보가 담길 가능성이 높다(ADR-0002
  참고) — GitHub 공개 범위는 기본값에 맡기지 말고 별도로 신중히 결정해야
  한다(이 ADR 시점까지는 아직 결정 안 됨, 별도 추적).

## References

- `sun-moon-python-platform/docs/adr/0001-use-adrs-for-decisions.md` —
  이 관행이 모델로 삼은 ADR.
- Michael Nygard, *Documenting Architecture Decisions* (2011) — 같은 원전.
  이 포맷은 소프트웨어 architecture를 넘어, 근거를 날짜와 함께 남길 가치가
  있는 모든 결정에 일반화된다.
