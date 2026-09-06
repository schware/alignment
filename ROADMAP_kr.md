*🇬🇧 English version: [ROADMAP.md](ROADMAP.md)*

# Roadmap — 초안, additive하게 계속 고쳐감

이건 확정된 약속이 아니라 첫 초안이다. 이 프로젝트의 작업 방식(한 조각을
만들고, 돌아보고, 그 위에 더한다 — `sun-moon-python-platform`이 이걸
참조하는 ADR 참고)대로, 각 Phase가 끝나고 우선순위가 바뀌면 이 문서도
바뀔 것으로 예상한다. Phase의 범위가 실질적으로 바뀌면, 이 파일의 이력을
조용히 지우는 대신 **왜** 바뀌었는지 ADR을 쓴다.

**이 로드맵이 딛고 선 baseline**: ADR-0002 (C/C++/C#/Java + 서버통신/POS
배경, 아직 cloud-native/AI-ML 경험 없음, 2026-09-06부터 2026년 12월
타겟까지 약 3.5개월, 시장 무관, 기존 포트폴리오 없음). **따르는 전략**:
ADR-0003 (Backend/Platform, AI/ML, DevOps/SRE 세 시그널 전부에 의도적으로
걸치는 Platform 하나).

## Phase 0 — 완료 (2026-09-06 기준)

`sun-moon-python-platform`: 동작하고, 테스트되고, end-to-end로 검증된
Python monorepo — `order-service` + `ai-agent-service`, DDD 구조, 비동기
HTTP/TCP, 서비스 간 Redis pub/sub, 통과하는 테스트 9개, 근거를 기록한
ADR-0000~0009. 이것만으로도 이미 **Backend Engineering**과 **AI/ML
Engineering**에 대한 부분적 증거가 된다(AI Agent 루프는 존재하고
동작하지만, 지금은 결정론적 오프라인 provider 기준 — Phase 1 참고).

## Phase 1 — Python 쪽을 깊게 만든다 (목표: 약 6주, 10월 말까지)

목표: 새 언어를 더하기 전에, "배관이 동작한다"를 "이건 실제로 방어 가능한
엔지니어링이다"로 바꾼다.

- 실제 Anthropic provider 연결(지금은 `echo`뿐) — API 키가 필요하고,
  실제 모델을 상대로 한 번도 안 돌려본 코드가 아니라 진짜 검증 pass가
  필요하다.
- `order-service`에 Alembic 마이그레이션 추가(지금은 시작할 때
  `create_all` — 그 저장소 README에 이미 gap으로 표시돼 있음).
- 두 번째 bounded-context 서비스(예: `delivery-service`) 추가 — DDD/monorepo
  패턴이 예시 하나를 넘어 일반화되는지 증명. 그 저장소의
  ADR-0003/ADR-0006에서 이미 미검증으로 표시해둔 부분.
- 기본 CI(GitHub Actions): push마다 두 서비스의 pytest 스위트 실행.
  README의 초록색 체크 배지 하나가, 포트폴리오 저장소를 훑어보는
  리뷰어에게 실제 가치 이상으로 설득력 있다.

**왜 이게 먼저인가**: 리스크가 가장 낮고 레버리지는 가장 큰 Phase다 —
새 언어를 배울 필요 없이, 이미 만든 시스템을 "데모"에서 "방어 가능한 것"으로
바꾼다. 아직 불안정한 Python 기반 위에 바로 Go/TypeScript로 건너뛰는 건
실수일 것이다.

## Phase 2 — Go 컴포넌트 하나 (목표: 약 4주, 11월 말까지)

목표: 절반씩 만든 세 개가 아니라, 잘 만든 Go 조각 **하나**로 polyglot
Platform 역량을 증명한다 — 그리고 ADR-0004에 따라, 그냥 "Go를 써본다"가
아니라 실제로 겪은 구체적인 문제(과거 IOCP 기반 C++ 작업에서
completion-callback에 흩어졌던 domain logic)를 의도적으로 해결하는 데
쓴다.

- 선택적 1~2일 몸풀기: Go 들어가기 전에 C#의 `async`/`await`(이미
  알고 있음)를 다시 보면서 ADR-0004의 "IOCP → 선형 async 코드" 교훈을
  재확인한다 — 저위험, 빠른 확신 체크.
- 후보: `order-service`/`ai-agent-service` 앞단의 Go 기반
  Gateway/Transport 서비스 — 원래 C 프로젝트의 "combined-server" 아이디어와
  개념적으로 비슷하고, 본인의 가장 강한 기존 스킬(C/C++ 시스템 프로그래밍
  — Go의 사고모델이 가깝다)을 새롭고 수요 있는 언어로 옮기는 직접적인
  다리가 된다. 이 컴포넌트의 ADR(`sun-moon-python-platform`에 작성)에는
  goroutine/channel을 백화점 시스템의 completion-callback 스타일과 명시적으로
  비교하는 내용을 넣어야 한다 — ADR-0004 참고.
- Rust는 당분간 Go보다 뒤로 미룬다(ADR-0004) — Go의 CSP 모델이 먼저
  증명해야 할 더 대조적인 교훈이고, Rust의 async/await는 나중에 이어가면
  된다.
- 범위는 의도적으로 작게: Go 바이너리 하나, 명확한 역할 하나(요청
  라우팅/프록시, 어쩌면 기본 메트릭 수집), 잘 테스트되고,
  `sun-moon-python-platform`에 그 선택을 설명하는 자체 ADR도 딸려 있게.
- 여기서 `sun-moon-python-platform`의 ADR-0009 open question 1번(독립 Go
  서비스 여러 개 vs. 내부 모듈을 가진 Go 프로세스 하나)이 논의만이 아니라
  **실제로 만들면서** 첫 구체적인 답을 얻는다.

**왜 전체 Core Runtime/Transport/Event Bus 분할이 아니라 컴포넌트
하나인가**: ADR-0003이 이미 너무 많이 걸치는 것의 실행 리스크를 짚어뒀다.
완성되고 잘 문서화된 Go 서비스 하나가, 미완성 세 개보다 더 강한 인터뷰
소재다.

## Phase 3 — DevOps 다듬기 (목표: 약 2~3주, 12월 중순까지)

목표: Phase 1~2의 **AI/ML + Backend** 작업이 실제로 운영 가능한 시스템처럼
보이고 동작하게 만든다 — DevOps/SRE 시그널은 대부분 여기서 나온다.

- `docker-compose.yml`이 실제로 빌드·실행되는지 검증(그 저장소에 미검증으로
  표시돼 있음 — 이 환경엔 테스트할 Docker가 없었음).
- Go 컴포넌트까지 포함하는 실제 CI 파이프라인(lint + build + test).
- 무료/저가 어딘가(Fly.io, Render, 작은 VPS)에 실제 배포 — 인터뷰에서
  동작하는 URL 하나가 천 마디 말보다 낫다.

## Phase 4 — 선택 / stretch (시간이 남으면만)

- `/health`와 최근 주문/에이전트 노트를 보여주는 최소한의 TypeScript
  Dashboard. 명시적으로 가장 낮은 우선순위 컴포넌트 — Q4 타겟을 넘기거나,
  전환 시점 이후로 완전히 미뤄져도 실패로 취급하지 않는다. 원래
  Platform 다이어그램의 모든 칸이 다 채워져야만 정당하고 제시할 만한
  포트폴리오가 되는 건 아니다.

## 명시적 scope-cutting 규칙

시간이 부족해지면, Phase 1/2의 깊이를 깎기 전에 이 목록의 아래쪽(Phase 4,
그다음 Phase 3 후반부)부터 잘라낸다 — 이게 등장할 만한 어떤 인터뷰
대화에서도, 작지만 완성되고 잘 문서화된 시스템이 방대하지만 절반만 만든
시스템보다 낫다.

## 아직 안 정한 것

- "Q4 2026"이 "전환 전에 포트폴리오가 인터뷰 준비 완료 상태여야 한다"는
  뜻인지, "전환 직후에도 계속 만들어간다"는 뜻인지(ADR-0002에서 열어둔
  부분) — Phase 3/4를 얼마나 압축할지에 영향.
- 이 저장소와 앞으로 `sun-moon-python-platform`의 GitHub 공개 범위 — 이
  저장소의 타임라인/전환 관련 내용이 순수 architecture 근거보다 더
  민감하다는 점을 감안해야 함.
