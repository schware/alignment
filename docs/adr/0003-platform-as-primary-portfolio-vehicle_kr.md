*🇬🇧 English version: [0003-platform-as-primary-portfolio-vehicle.md](0003-platform-as-primary-portfolio-vehicle.md)*

# ADR-0003: 하나의 Platform 프로젝트를 메인 포트폴리오 vehicle로 삼고, 세 타겟 직무군 전부에 의도적으로 걸치게 한다

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: 본인

## Context

`sun-moon-python-platform`의 ADR-0009는 순수 Framework 대신 Platform
모양(Core Runtime/Agent Runtime/Domain SDK/Transport Layer/Event
Bus/Dashboard/CLI)을 선호하는 **기술적** 이유를 기록해뒀다 — 프로젝트가
실제로 되어가고 있는 모습에 더 맞는다는 것. 별개로, ADR-0002의
baseline(기존 포트폴리오 없음, 세 개의 서로 다른 직무군 — Backend/Platform
Engineering, AI/ML Engineering, DevOps/SRE — 을 우선순위 없이 다 노림)을
감안하면 커리어 전략 질문도 하나 답해야 했다: 세 군데 다 걸치는 것 하나를
만들지, 아니면 직무군별로 작은 프로젝트 세 개를 따로 만들지.

## Decision

**Platform 하나**를 만든다, 포트폴리오 프로젝트 세 개가 아니라. 그리고
그 컴포넌트 구성을 세 타겟 직무군에 의도적으로 대응시킨다:

- **Core Runtime / Transport Layer / Event Bus / CLI** (Go) →
  Backend/Platform Engineering 시그널
- **Agent Runtime / Domain SDK** (Python) → AI/ML Engineering 시그널
- **Dashboard**(TypeScript), 그리고 모든 컴포넌트에 걸친 Docker/CI/배포
  작업 → DevOps/SRE 시그널

한 사람이 만든, 세 영역 전부에 설득력 있게 걸쳐 있는 하나의 일관된
Platform은, 서로 연결 안 되는 토이 프로젝트 세 개보다 더 강한 포트폴리오
서사다 — 부분들이 서로 맞물리게 시스템을 설계할 수 있다는 걸 보여주는데,
이건 senior/platform급 직무가 특히 찾는 시그널이다. 또한 한 직무 서사를
위해 한 작업이, 최종적으로 다른 직무에 지원하게 되더라도 낭비되지 않는다.

## Alternatives considered

- **직무군별로 작은 프로젝트 세 개** — 프로젝트당 리스크는 낮고(작은 걸
  끝내기가 더 쉬움), 각 프로젝트를 그 직무의 기대치에 정확히 맞출 수
  있다. 지금은 기각했다 — (a) 완성된 작은 프로젝트 세 개는 "이것저것
  건드려본" 인상을 주지만, 통합된 시스템 하나는 "설계할 수 있다"는 인상을
  준다, (b) ADR-0002의 타임라인(약 3.5개월, 기존 포트폴리오 없음)으로는
  독립된 결승선 세 개를 편하게 맞추기 어렵다 — 완성된 하나 대신 미완성
  세 개로 끝날 가능성이 높다.
- **직무군 하나만 골라서 깊게 판다** — 리스크가 가장 낮고, 초점이 가장
  명확하고, 아마 다듬어진 결과물 하나를 가장 빨리 만드는 길이다. 본인이
  세 군데 다 열어두고 싶다고 명시적으로 밝혔으므로(ADR-0002) 기각했다 —
  다만 타임라인이 빠듯해지면 돌아갈 수 있는 대안으로 살려둔다
  (`ROADMAP.md`의 scope-cutting 메모 참고).

## Consequences

- 실행 리스크가 올라간다: 미완성된 다국어 Platform은 완성된 단일언어
  프로젝트 하나보다 약한 포트폴리오다. `ROADMAP.md`에 명시적이고 가차없는
  우선순위/scope-cutting 체크포인트가 필요하다.
- 추가하는 컴포넌트 하나하나가 특정 직무군의 기대치에 대해 정당화될 수
  있어야 한다, 그냥 기술적으로 재밌어서가 아니라 — 안 그러면 다시
  "그 자체를 위한 Framework"로 흘러가버리고 의도된 포트폴리오 결과물에서
  멀어진다.
- 이 저장소의 로드맵이 `sun-moon-python-platform`의 기술 로드맵과 직접
  묶인다 — 한쪽이 바뀌면 다른 쪽도 점검해야 할 가능성이 높다(예: 그
  저장소의 ADR-0009 open questions가 "Go 프로세스 하나"가 아니라 "독립
  Go 서비스 3개"로 정리되면, 이 ADR의 타임라인 가정도 다시 봐야 한다).

## References

- `sun-moon-python-platform/docs/adr/0009-pivot-framework-to-platform.md` —
  이 커리어 쪽 결정이 딛고 서 있는 기술 쪽 근거.
- ADR-0002 (이 저장소) — 이 결정이 전제로 삼은 baseline.
