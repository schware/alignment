*🇬🇧 English version: [ROADMAP.md](ROADMAP.md)*

# Roadmap — 살아있는 문서(Living Document)

이 문서는 확정된 계획이 아니라 현재 시점의 초안이다.
이 프로젝트의 작업 방식(한 조각을 만들고, 돌아보고, 그 위에 더한다)대로,
각 Phase가 끝나고 우선순위가 바뀌면 이 문서도 바뀔 것으로 예상한다.
Phase의 범위가 실질적으로 바뀌면, 이 파일의 이력을 조용히 지우는 대신 **왜** 바뀌었는지 ADR을 쓴다.

**이 로드맵이 딛고 선 baseline**: ADR-0002/ADR-0005 (총 16년)

- MFC 1~4년차
- C++ Server 5~8년차
- C# POS Client 5~16년차
- Java/Spring Boot REST+Batch 9~16년차
- 아직 cloud-native/AI-ML 만들어 보지 못함
- 2026-09월부터 Q4 2026 타겟까지 약 3.5개월, 시장 무관, 기존 포트폴리오 없음

**따르는 전략**: ADR-0003 (Backend/Platform, AI/ML, DevOps/SRE 세 시그널 전부에
의도적으로 걸치는 Platform 하나), 여기에 ADR-0004(새 언어마다 관통하는
기술적 방향 하나)와 ADR-0006(Batch를 C와 Python 둘 다, 공유 계약을 통해
만듦)이 더해짐.

## Phase 0 — 완료 (2026-09-06 기준)

`sun-moon-python-platform`: 동작하고, 테스트되고, end-to-end로 검증된
Python monorepo — `order-service` + `ai-agent-service`, DDD 구조, 비동기
HTTP/TCP, 서비스 간 Redis pub/sub, 통과하는 테스트 9개, 근거를 기록한
ADR-0000~0009. 이것만으로도 이미 **Backend Engineering**과 **AI/ML
Engineering**에 대한 부분적 증거가 된다(AI Agent 루프는 존재하고
동작하지만, 지금은 결정론적 오프라인 provider 기준 — Phase 1 참고).

이미 GitHub에 올라간 것 하나 더: `sun-moon-c-server` — 기존에 만든 C
서버 프레임워크(libuv 기반 TCP/HTTP/WS/TLS, 1만 동시접속까지 검증됨),
이제 C 언어 방향의 근거지가 됨(새로 추가된 Phase 1.5와 ADR-0006 참고).

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
- **완료, 2026-09-06**: Spring Batch의 Job → Step → Chunk
  (Reader/Processor/Writer) 모델을 구현하는 `batch-service` — `order-service`의
  기존 공개 REST API(이 작업과 함께 추가된 진짜 `limit`/`offset`
  페이지네이션 포함)로 주문 데이터를 읽어서 주기적 집계(일별 주문
  요약)를 만든다. 이 Phase에서 가장 시그널이 강한 추가다 — 가장 깊고 가장
  최근인 8년의 전문 분야(ADR-0005)를 직접 재현한다. 세부 설계는 그
  저장소 자체의
  [ADR-0010](https://github.com/schware/sun-moon-python-platform/blob/master/docs/adr/0010-batch-service-design_kr.md)에
  있고, `sun-moon-c-server`의 C 구현과의 직접 비교도 포함돼 있다. 영향받는
  패키지 전체에서 테스트 17개 통과; 실제 동작 중인 `order-service`(주문
  10개, 매출 정확히 일치)로 end-to-end 검증함.

**왜 이게 먼저인가**: 리스크가 가장 낮고 레버리지는 가장 큰 Phase다 —
새 언어를 배울 필요 없이, 이미 만든 시스템을 "데모"에서 "방어 가능한 것"으로
바꾼다. 아직 불안정한 Python 기반 위에 바로 Go/TypeScript로 건너뛰는 건
실수일 것이다.

## Phase 1.5 — C Batch + 저장소 간 통합 — 완료, 2026-09-06

**양쪽 다 완료.** C 쪽 2026-09-06(아래); Python 쪽(위 Phase 1)도 같은 날,
`order-service`가 진짜 페이지네이션을 갖게 되자마자.

목표(ADR-0006에 따라): 같은 Job/Step/Chunk 설계를 C에서도 증명한다 — 이
언어엔 추상화를 위한 내장 메커니즘이 없어서, ADR-0004의 논지를 가장
어렵게, 그래서 가장 설득력 있게 증명하는 경우가 된다 — 그리고
`sun-moon-python-platform`의 "코드 공유가 아니라 계약"이라는 architecture가
완전히 다르고 더 오래된 기술 스택에 걸쳐서도 성립하는지 증명한다.

- ~~`sun-moon-c-server`에 진짜 DB 클라이언트(SQLite) 추가~~ — **이번
  Phase에서는 의도적으로 하지 않음.** `sun-moon-c-server` 자체
  [ADR-0001](https://github.com/schware/sun-moon-c-server/blob/master/docs/adr/0001-batch-job-design_kr.md)에서
  명시적으로 범위 밖으로 뺐다: 이 batch job의 저장 방식은 파일(NDJSON +
  JSON)이지 데이터베이스가 아니라서, `db_demo_service.c`의 시뮬레이션-DB
  gap은 그 저장소의 "Planned next steps"에 별개의, 아직 일정 안 잡힌
  항목으로 그대로 남는다.
- **완료**: `order-service`의 공개 `GET /orders`를 호출하는 C용 HTTP
  클라이언트(libcurl, CMake에 연결) — 이미 `ai-agent-service`가 쓰고
  있는 같은 계약을, 이제 C에서도 동작한다는 걸 증명.
- **완료**: Job/Step/Chunk 패턴을 C 관용구로 손수 구현(Reader/Processor/
  Writer 자리에 함수 포인터를 담은 구조체) —
  `include/batch.h`/`src/batch.c`(범용 엔진) +
  `src/services/order_summary_batch_job.c`(실제 job: chunk마다 매출
  합산, NDJSON 스트리밍, JSON summary + JSON execution report 작성).
  `config/batch-job.json`이 이걸 구동하고, 이번 Phase의 "batch를 JSON으로
  관리" 지시와 그대로 맞아떨어진다. 실제 동작 중인 `order-service`를
  상대로 end-to-end 검증함: 단일 chunk(`chunk_size: 5`, 주문 3개)와
  다중 chunk(`chunk_size: 2`, 2+1) 둘 다 정확한 매출 합계를 냈고,
  `scripts/run_batch.sh`가 `batch_runner`의 exit code(성공 0 / 실패 1)를
  cron/systemd-timer 스케줄링용으로 정확히 전달함. 전체 설계와 자체
  "Alternatives considered"/"Consequences"는
  [`sun-moon-c-server`의 ADR-0001](https://github.com/schware/sun-moon-c-server/blob/master/docs/adr/0001-batch-job-design_kr.md)
  참고.
- **완료**: 위 Python `batch-service`와의 직접 비교 — 같은 job 의미,
  같은 출력 모양, 같은 내결함성 모델. 다만 Python reader는 HTTP 위에서
  진짜로 chunk에 bound되는 반면, 이 C reader는 아직 한 번에 전부
  받아온다(이 저장소 README의 "Known gap" 메모와 갱신된 "Planned next
  steps" 참고).

**왜 Phase 1 바로 다음, Go보다 먼저인가**: C는 이미 아는 영역이다(저위험)
— 완전히 새 언어(Go)를 다루기 전에 이걸 먼저 배치하면, 자신감이 높은
상태에서 완성된 비교쌍(Python batch vs. C batch, 같은 설계)을 만들게
되고, Phase 2가 Go를 맨땅에서 시작하는 대신 두 번째 데이터 포인트 위에서
시작하게 해준다.

**일정 영향, 추정 vs. 실제**: 이 Phase는 약 2~3주로 추정했었다; 실제로는
C와 Python 양쪽 다 계획을 세운 바로 그날(2026-09-06) Claude와 함께
완료됐다. 앞으로의 모든 Phase가 이 정도로 압축될 거라고 가정하기보다는
하나의 보정 데이터 포인트로 삼을 만하다 — 이번 Phase는 이미 설계된
패턴(Job/Step/Chunk)을 새로 발명하는 대신 두 번 재사용한 것이라, 추정보다
빨리 끝난 이유의 상당 부분이 거기 있다. 아래 Phase 2/3의 목표 일정은
Phase 2 자체가 두 번째 데이터 포인트를 줄 때까지 당초 추정대로 남겨둔다.

## Phase 1.6 — `sun-moon-c-server`: port 통일, 서비스 기반 architecture로 리팩터링 (목표: 다음 세션)

목표([ADR-0007](docs/adr/0007-prefer-process-isolation-over-port-splitting_kr.md)에
따라): `sun-moon-c-server`를 `sun-moon-python-platform`의 서비스당
프로세스 모델에 맞춘다 — ADR-0004의 "원칙은 언어를 넘어 전이된다"는
논지가 Job/Step/Chunk 하나만이 아니라 서비스 분해 자체로도 확장된다는
두 번째 구체적 증거.

- `server`의 지금 모양을 다시 본다: CMake 타겟 하나, 프로세스 하나,
  config로 모드(tcp/http/udp)를 골라 스레드로 돌리는 방식, 여러 전용
  port(HTTP 8081, TCP 8090, UDP 8082, HTTPS 8443, 9010~9020 TCP-range
  데모)에 흩어져 있음 — 그리고 이미 별도 일회성 바이너리인 `batch_runner`.
- 업무 능력마다 독립적으로 실행 가능한 서비스/바이너리로
  (`order-service`/`ai-agent-service`/`batch-service`를 거울삼아) 옮겨간다,
  각각 일관된 port 하나씩 — 지금처럼 프로세스 하나가 config + 스레드로
  여러 모드를 저글링하는 대신.
- 아직 안 정한 것: C 쪽의 정확한 서비스 경계(기존 모드들이 어떤
  서비스가 될지), `combined-server.json`/멀티스레드 편의 모드를
  없앨지 옵션으로 남길지. 시작되면 `sun-moon-c-server` 자체 ADR이 필요함.
- 아직 코드 변경 없음 — 이 Phase는 다음 세션에 시작한다(본인이 직접
  정한 일정: "내일 토큰이 초기화되면").

## Phase 2 — Go 컴포넌트 하나 — 다음 세션에 재검토 (ADR-0008에 따라)

무기한 정지가 아니다 — Phase 1.6과 같은 다음 세션(내일)에 다시
다룬다. 본인이 Go/TypeScript/Rust에 대한 생각을 먼저 스스로 정리하고
싶어 한다; 아래 내용은 오늘 시점의 내용이고, 그 다음 세션의 출발점으로
남겨둔다, 확정된 계획이 아니라.

**오늘(2026-09-06) 기준 내용, 다음 세션에 재검토 예정:**

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

## Phase 3 — DevOps 다듬기 (목표: 약 2~3주, Q4 2026을 넘겨 전환 시점까지 이어질 가능성 높음 — 위 일정 메모 참고)

목표: Phase 1~2의 **AI/ML + Backend** 작업이 실제로 운영 가능한 시스템처럼
보이고 동작하게 만든다 — DevOps/SRE 시그널은 대부분 여기서 나온다.

- `docker-compose.yml`이 실제로 빌드·실행되는지 검증(그 저장소에 미검증으로
  표시돼 있음 — 이 환경엔 테스트할 Docker가 없었음).
- Go 컴포넌트까지 포함하는 실제 CI 파이프라인(lint + build + test).
- 무료/저가 어딘가(Fly.io, Render, 작은 VPS)에 실제 배포 — 인터뷰에서
  동작하는 URL 하나가 천 마디 말보다 낫다.

## Phase 4 — 선택 / stretch — 다음 세션에 재검토 (ADR-0008에 따라)

Phase 2와 마찬가지로 무기한 정지가 아니다 — 재정리된 Go/TypeScript/Rust
계획과 함께 다음 세션에 다시 다룬다(TypeScript가 이 Phase의 언어다).

- `/health`와 최근 주문/에이전트 노트를 보여주는 최소한의 TypeScript
  Dashboard. 명시적으로 가장 낮은 우선순위 컴포넌트 — Q4 타겟을 넘기거나,
  전환 시점 이후로 완전히 미뤄져도 실패로 취급하지 않는다. 원래
  Platform 다이어그램의 모든 칸이 다 채워져야만 정당하고 제시할 만한
  포트폴리오가 되는 건 아니다.

## 별도 병렬 트랙 (이 로드맵의 시간 예산 밖) — Mobile App

[ADR-0011](docs/adr/0011-mobile-app-parallel-portfolio-track_kr.md)에
따라, **Phase가 아니다** — 이 로드맵의 Q4 2026 순서/scope-cutting 규칙
밖에 의도적으로 둬서, Phase 1.6/2/3/4의 시간과 절대 경쟁하지 않게 한다.
존재를 눈에 보이게 하기 위해서만 여기 적어둔다.

- **무엇**: `D:\Claude_Code\App`(별도 저장소)에 만드는 크로스플랫폼
  (iOS/Android) 앱, Expo/React Native/TypeScript. 기능 두 가지: 회의
  녹취록을 붙여넣으면 Claude API가 Markdown 요약을 만들어 GitHub에
  커밋; 아침 계획/저녁 확인 형태의 일일 일정을 Markdown으로 GitHub에
  커밋.
- **왜 그래도 기록해두는가**: 본인은 이게 커리어와 무관한 사이드 도구가
  아니라 실제 포트폴리오 시그널(모바일 + 서드파티 AI/API 통합)로 의도한
  것임을 확인했다 — 다만 이 저장소의 Q4 타겟과는 명시적으로 병렬이고
  순서에 안 들어간다.
- **현황, 2026-09-06**: MVP 스캐폴드 완료, 브라우저 미리보기로 스모크
  테스트 완료 — 화면 4개, GitHub PAT 기반 동기화, 자격 증명 기기 내
  보안 저장, 회의 입력은 텍스트 붙여넣기만(녹음/STT는 아직 없음).
- 이 트랙이 Phase 1.6/2/3에 배정된 시간을 실제로 가져가기 시작하면,
  조용히 Q4 타임라인을 갉아먹게 두지 말고 그 드리프트 자체에 대한 별도
  ADR을 써야 한다(ADR-0011의 Consequences 참고).

## 활동 로그 (이 로드맵의 Phase 밖) — `sun-moon-java-platform` 현대화

Phase가 아니고, ADR-0003 의미의 새로운 포트폴리오 구축 작업도 아니다 —
여기서의 가치는 새 스킬 습득이 아니라 이미 있던 Java/Spring Boot
전문성(ADR-0005의 9~16년차 baseline)을 유지·현대화한 것이다. Mobile App
트랙과 같은 이유로, 존재를 눈에 보이게 하기 위해서만 여기 적어둔다.

- **무엇, 2026-09-08**: `sun-moon-java-platform`을 하나의 WAR + 공용
  Jetty 배포에서, Order/KDS/Delivery 3개의 독립 Spring Boot 서비스로
  분리했다 — 각각 별도 GitHub 저장소로 나뉘고 git submodule umbrella로
  묶여 있으며, 각자 자기만의 Docker 컨테이너와 자기만의
  `context-path`(`/order`, `/kds`, `/delivery`)를 갖는다. 서버의
  2010년형 CPU가 MongoDB가 요구하는 AVX 명령어를 지원하지 않는다는 걸
  발견한 뒤 Order/Delivery는 PostgreSQL+JSONB로, KDS는 Redis로 갔다.
  WAR/Jetty 시절 남아있던 경로 redaction 버그(health/metrics 엔드포인트가
  옛 redaction 마커와 안 맞는 Docker 컨테이너 경로를 그대로 노출하던
  것)를 고쳤고, 각 서비스의 OpenAPI `servers` URL도 새 context-path와
  맞춰뒀다.
- **왜 그래도 기록해두는가**: `-jetty` 접미사가 붙은 저장소들과
  `sun-moon-java-platform`의 `docs/adr`(0001~0006) 기록은 ADR-0005의
  Java/Spring Batch baseline이 정체돼 있지 않고 실제로 계속 관리되고
  있다는 독립적인 근거가 된다 — Q4 포트폴리오 구축 순서에는 안 들어가도
  이 저장소에서 보일 가치는 있다.
- 이 작업은 Phase 1/1.5/1.6/2/3/4의 시간 예산과 경쟁하지 않는다 — 그
  Phase들과 별개로, 병행해서 진행됐다.
- **이름 충돌, 명시적으로 짚어둠**: 바로 아래("제안됨... Enterprise
  Runtime Platform" 섹션)에서 설명하는, *처음부터 새로 만드는, Netty
  기반, Spring 없는* Java 포트폴리오 프로젝트가 이 실제 기존 Spring Boot
  시스템과 정확히 같은 repo 이름 `sun-moon-java-platform`을 씁니다 —
  이름도 같고 GitHub repo도 같지만, git 브랜치가 다릅니다: `main`이 이
  실제 Spring Boot/WAR→microservices 시스템(이 활동 로그 항목)이고,
  `master`가 아래에서 설명하는 처음부터 새로 만든 Spring-없는
  재구현입니다. 같은 repo 안에 서로 무관한 두 코드베이스가 브랜치로만
  나뉘어 있는 것 — 이 로드맵을 읽을 때나, 그 repo 자체의 커밋 이력이나
  `docs/adr/`을 읽을 때 혼동하지 말 것(브랜치마다 ADR 번호 체계도 독립적임).

## 제안됨 (아직 일정 없음) — Java: DDD 기반 Enterprise Runtime Platform

[ADR-0008](docs/adr/0008-pause-go-ts-rust-add-java-cpp_kr.md)과
[ADR-0009](docs/adr/0009-java-enterprise-runtime-platform-scope-expansion_kr.md)에
따라, **아직 확정 안 됨** — 아래 내용은 Ongoing 방향성 설정이지 확정된
범위가 아니다. ADR-0008은 원래 Java를 Batch 전용 reference
implementation으로만 제안했다; ADR-0009는 본인이 2026-09-06에 이걸 단일
Runtime Platform 전체로 확장했음을 기록한다 — 세 번째 Batch 데모보다는
`sun-moon-c-server`의 야심(하나의 Runtime, 여러 Transport)에 구조적으로
더 가깝다.

- **형태**: Socket, REST API, WebSocket, Batch Job을 함께 호스팅하는 단일
  Java Runtime, DDD 구조, OpenSource 우선, 동시 접속자 1,000~10,000명
  목표.
- **후보 스택 (본인 제안, 2026-09-06)**: Netty(Core Runtime / Transport),
  Quartz(Batch Scheduler), MyBatis + Oracle(Persistence), Redis(Cache/
  Session/Lock), Kafka(Event Bus), Logback(Logging), Micrometer/
  Prometheus/Grafana(Monitoring), OpenTelemetry(Tracing).
- **Claude의 추가 제안 (ADR-0009)**: Spring Core DI + 실제 Spring Batch
  (Quartz는 이걸 대체하지 않고 트리거만 함 — ADR-0008의 원래 "정답
  기준선" 요점과 다시 연결); REST/WS 라우팅을 Netty의 event loop 위에
  올리기 위한 Reactor Netty/Spring WebFlux; HikariCP; Jackson; Jakarta
  Bean Validation; Resilience4j; Redisson; Flyway; JUnit5 + Mockito +
  Testcontainers.
- Java는 이 Platform 안에서도 Job/Step/Chunk 비교의
  **reference implementation** 역할은 그대로 유지한다 — Python/C Batch
  작업이 검증받는 "정답" 기준선, Spring Batch가 본인이 8년의 실전 경험을
  가진 분야이기 때문(ADR-0005) — 다만 이제 그 비교는 전체 범위가 아니라
  더 큰 Runtime의 한 조각이다.
- **아직 결정 안 됨** (ADR-0009 참고): DI/라우팅 아키텍처, build tool,
  repo 이름(후보: `sun-moon-java-platform`), Phase 1.6 및 재정리된
  Go/TS/Rust 계획과의 순서.

- **같은 주에 다시 내리고, 저장소 셋으로 쪼갬, 2026-09-09** — 위의 port
  배정(8083/8084/9090)이 틀렸는데, 틀린 방식이 번호가 아니라 구조였다.
  서버의 체계는 **service 단위**로 배정한다 — 8000 hub, 8080 BO,
  8081~8089 REST API, 9011/9021/9031이 Java/Python/C의 Socket. container
  하나가 서로 다른 세 범주의 세 칸을 물고 있으면 번호를 다시 매겨서
  맞출 수 있는 문제가 아니다. container를 내렸고(ADR-0013), 질문은 "어느
  port"가 아니라 "container 몇 개"가 됐다.
- **답: kernel 하나에 service 둘.**
  [`sun-moon-platform-core`](https://github.com/schware/sun-moon-platform-core)가
  공용 runtime이다 — Netty listener bind, REST routing, event loop 밖에서
  실행한다는 계약, MyBatis/Flyway/Micrometer/OTel 결선. `main`도 port도
  domain도 없는 library다.
  [`sun-moon-platform-bo`](https://github.com/schware/sun-moon-platform-bo-netty)가
  Back Office(8080, LAN 전용)이고, `sun-moon-java-platform`의 `master`는
  장비를 마주하는 쪽을 갖는다. 각 service는 kernel을 git submodule로
  물고 Gradle composite build로 쓴다 — artifact registry를 안 쓰므로,
  같은 디스크에 있는 소비자 둘짜리 의존을 위해 publish token을 관리할
  일이 없다(ADR-0014).
- **저장소를 만들기 전에 경계가 실제로 성립하는지부터 확인했다.** 남길
  가치가 있는 건 이 부분이다 — 아래로 새는 곳이 셋 있었다. runtime이
  socket port를 직접 bind했고, HTTP initializer가 WebSocket echo handler를
  import했고, MyBatis 설정이 domain mapper 넷을 나열하고 있었다. 셋 다
  진짜 결합이었고, 셋 다 제자리에서 고쳤고, 그 동안 26개 테스트가 계속
  통과했다. 그러고 나서야 파일을 옮겼다. BO의 package는
  `com.sunmoon.bo.*`로 바꿨다 — 의존의 방향이 build 파일의 주장이 아니라
  모든 import에서 눈에 보이도록.
- **분리 후 컴파일만이 아니라 실제로 검증했다**: BO를 자기
  `installDist` 배포본에서 띄워 로그인·session cookie·공통코드 CRUD를
  왕복시켰고, 세 저장소의 테스트는 3 + 12 + 11 = 26개로 분리 전과 같다.
  이는 ADR-0002의 "단일 runtime" 전제를 좁힌다 —
  Socket/REST/WebSocket/Batch를 함께 두는 것은 여전히 이 platform의
  모양이지만, BO는 workload가 아니라 관리였고 애초에 그 안에 있을
  것이 아니었다.
- **기록해 둘 만한 회귀**: 기존 Spring Order service를 8080에서
  8083으로 옮기면서(8080을 BO 자리로 비우려고) UFW에 8083을 안 열어,
  컨테이너는 떠 있고 health도 UP이고 `localhost`로는 응답하는데 밖에서만
  몇 시간 동안 안 닿았다. 증상이 방화벽 문제처럼 보이지 않는다는 것이
  바로 시간을 잡아먹은 이유다. `Debian-Setting`의 `docs/docker.md`에
  해법과 나란히 적어뒀다.

- **같은 날 BO를 Spring으로 되돌림, 2026-09-09** — 몇 시간 전의 결정을
  뒤집은 것이라 오히려 기록할 값이 있다. Spring 포기는 2026-09-06에
  platform 전체에 대해 정해졌고, BO 이야기가 나온 건 그 이틀 뒤다. 즉 BO는
  자기 결정을 받은 적이 없고, 자기가 존재하지도 않던 시점의 결정을 물려받은
  것이다. 물려받은 것 자체는 문제가 아니었다 — BO가 Netty runtime 안의
  `/bo/*` route였을 때는 Spring MVC와 raw Netty pipeline이 한 프로세스에
  못 있으니 선택지가 없었다. **선택지를 만든 건 저장소 분리였고**, 그러자
  비교는 한쪽으로 기울었다. BO가 직접 만든 모든 것(`SessionStore`, cookie
  파싱, `PasswordHasher`, `AuthorizedEndpoint` decorator)이
  `spring-boot-starter-security`가 존재하는 이유 그 자체였고, 아직 없던
  것들(운영자 관리 화면, CORS, 분산 session)은 Spring에서는 설정이며,
  결정적으로 BO에는 **처리량 요구가 아예 없다** — 운영자 몇 명이다.
  framework 없는 대가는 전부 치르고 이득은 하나도 못 받는 자리였다.
- **결과물**: `sun-moon-java-platform-bo`. 그 저장소 `main` 브랜치의
  *기존 Spring MSA 계열*에 Order/KDS/Delivery와 나란히 붙는 네 번째
  service다. 권한 모델은 그대로 살아남았다 — 3단계, 신규와 저장이 다른
  권한이므로 `create`와 `save`도 분리 — 다만 이제 Spring Security
  authority로 표현돼서 controller마다 필요한 권한을 그 자리에 적는다.
  테스트 19개, README의 curl 절차를 실제 filter chain으로 통과시키는 것까지
  포함한다. Netty 구현은 `sun-moon-platform-bo-netty`로 archive했다.
- **부산물이 더 쓸모 있었다.** "계열의 관례를 따르라"는 답할 수 없는
  주문이었다 — 관례가 "직전 service를 복사한다"로만 존재했고, 잘못된
  형제를 복사하면 티가 안 난다. 그래서 umbrella 저장소에
  `docs/CONVENTIONS.md`를 뒀다. 모든 규칙이 네 service를 전부 도는 명령으로
  쓰여 있어서, 어긋나면 출력에서 한 행만 다르게 나온다. 만들자마자 둘을
  잡았다 — BO가 Order의 옛 package 이름(`com.sunmoon.platform`, KDS와
  Delivery는 안 쓰는 것)을 물려받은 것, 그리고 **umbrella의 submodule
  pointer가 세 service 모두 4커밋씩 뒤처져 있던 것**. 그 빠진 4커밋이
  하필 문서화하려던 관례 그 자체였다.
- **포트폴리오 주장에 대한 솔직한 메모.** 직접 만든 BO는 메커니즘을
  이해하고 있음을 보여줬고, 그 코드와 근거는 archive된 채 그대로 남아 있다.
  이번 반전이 보여주는 것은 그와 다르고 흉내내기 더 어렵다 — 어떤 부품이
  자기 일에 비해 과한 기계를 지고 있다는 걸 알아채고, 같은 날 내린 결정을
  방어하지 않고 되돌리는 것. 그날 아침 뽑아낸 kernel은 소비자가 둘에서
  하나로 줄었고, 그래도 별도 저장소일 값이 있는지는 반사적으로 답하지 않고
  열어뒀다.

- **BO에 콘솔이 붙었다, TypeScript + React, 2026-09-09** — Phase 4와
  맞닿는 항목이다. BO는 화면 대신 Swagger UI를 세워둔 API였다. endpoint를
  증명하기엔 충분하지만, 운영자에게 "당신이 무엇을 할 수 있는지"를 보여주지
  못한다. 나는 순수 HTML/JS로 만들기 시작하면서 한 줄 통보만 하고 확인을
  받지 않았고, owner가 "프런트엔드 언어를 무엇으로 정했죠"라고 물은 것이
  옳았다. 정한 적이 없었고 내가 가정했을 뿐이다. **TypeScript로 간 이유는
  Phase 4의 TypeScript Dashboard다.** 콘솔을 거기서 만들면, 가장 먼저
  잘릴 후보였던 그 Phase가 이미 돌고 있는 시스템 안의 실물이 된다.
- **콘솔은 BO의 jar 안에 실려 나간다.** Dockerfile의 `node:20-alpine`
  단계에서만 빌드하므로 2010년형 배포 서버에는 여전히 Node가 없다. 이유는
  깔끔함이 아니다 — **BO에는 CORS 설정이 없어서**, 다른 origin의
  프런트엔드는 CORS가 생기기 전까지 session cookie도 CSRF token도 보내지
  못한다. 같은 origin은 그 문제를 푸는 게 아니라 아예 없앤다.
- **만드는 도중 설계 결함이 하나 드러나 고쳤다**: SPA 경로 `/bo/board`와
  endpoint `GET /bo/board`가 같은 URL이라, 화면에서 새로고침하면 JSON이
  내려왔다. API를 `/bo/api/*`로 옮기고, 화면 경로는 하나씩 명시해서
  `index.html`로 넘긴다 — 와일드카드로 받았다면 API의 진짜 404까지 HTML
  페이지로 답했을 것이다.
- **콘솔이 보여주는 것은 권한 모델 그 자체다.** 왼쪽 메뉴에는 권한 있는
  화면만 뜨고, 홈은 화면별로 무엇이 허용되는지의 표이며, 403이 날 버튼은
  비활성이다. 이건 표시일 뿐이고 실제 차단은 여전히 각 controller의
  `@PreAuthorize`가 한다 — 다만 3단계 모델이 처음으로 눈에 보이게 됐고,
  면접에서는 모델이 코드에 있다는 것보다 그쪽이 값이 크다.
- 이 방식으로 처음부터 끝까지 만든 첫 화면으로 **공지사항 게시판**을
  추가했다(domain → JDBC → controller → 화면). 작성자는 request body가
  아니라 session에서 가져오고, 그것을 고정하는 테스트가 있다. 테스트
  27개. 검증은 로그인 화면이 그려지고 모든 경로가 해석되는 데까지다 —
  로그인 뒤 화면은 직접 눌러보지 않았다. 비밀번호를 대신 입력하는 것은
  자동화 세션이 할 일이 아니기 때문이다.
- **Netty runtime이 자기 업무를 찾았다 — Device Server.** 분리 이후 계속
  "8083 예약, 업무 미정"이었던 그 빈칸이 채워졌다. 단말(POS/KDS/DID)의
  WebSocket 연결을 붙들고, Order service의 Redis 이벤트를 받아 그
  단말들에 push한다 — raw Netty와 동시접속 1만 목표가 애초에 있던
  이유다. Order에는 진짜 생애주기가 생겼다(PLACED →
  ACCEPTED/REJECTED/EXPIRED → PRODUCED → DELIVERING → COMPLETED, 잘못된
  전이는 409). port는 8087로 옮겼다 — 8083은 이제 Spring Order service의
  영구 자리다.
- **직접 돌려보지 않았으면 못 잡았을 결함 셋.** Netty의 WebSocket
  handler가 URI 전체를 경로와 비교해서 `/ws?deviceId=…`가 `/ws`로 인식
  안 됐다 — echo 시절엔 query string이 없어서 안 드러났다. Kernel의
  `JsonResponses`가 `Instant`를 아예 직렬화 못 해서, 시각을 반환하는
  어떤 endpoint도 500이 났다. 그리고 Order는 Spring의
  `StringRedisTemplate`로 평문 JSON을 발행하는데 Redisson은 기본이
  Kryo라서, 모든 이벤트가 "unregistered class ID"로 죽었고 스택트레이스에
  두 서비스 이름 중 어느 것도 안 나왔다. **마지막 것은 어느 쪽 단위
  테스트로도 못 잡는다 — 두 서비스 각자는 완전히 정상이었다.**
- **schema.sql이 문서에 적어둔 대로 정확히 대가를 치렀다, owner
  지시로 Flyway로.** 생애주기가 생기기 전에 쓰인 주문 행 두 개가
  timestamp가 null인 채로 역직렬화됐고, 만료 sweep이 10초마다 그
  주문들에서 죽으면서 다른 모든 주문의 만료도 같이 막았다. 멱등한
  `CREATE`는 컬럼을 추가할 수 있어도 이미 있는 행은 못 고친다. Migration
  하나로 해결했고, 계열 관례도 그쪽으로 방향을 틀었다.
- **단말은 device id가 아니라 `(매장, device id)`였다** —
  owner가 두 매장용 POS 브라우저 창을 두 개 열어보면서 드러났다, 그
  화면이 애초에 그러라고 만든 것인데. Device id만으로 키를 잡으니 두
  매장의 첫 카운터가 같은 단말이 됐다 — 연결마다 서로를 밀어내고, 밀려난
  쪽은 평범하게 끊긴 것처럼 보여서 재접속했고, 두 창이 서로를 끝없이
  끊어내는 루프가 CPU 코어 하나를 잡아먹었다. 이건 사용 실수가 아니라
  모델이 틀린 것이었고, 그 루프는 정확히 얼마나 틀렸는지의 시연이었다.
  `TerminalId(storeId, deviceId)`(연결 식별용)와
  `TerminalGroup(storeId, type)`(라우팅·유예 판단용)로 다시 키를 잡았고,
  진짜로 밀려난 연결은 WebSocket code 4001로 닫혀서 그 클라이언트가 더
  이상 갖고 있지 않은 id를 붙잡고 영원히 재시도하는 대신 멈춘다.
- **End to end, 실물로 확인** — 이 앱 자신의 말이 아니라. 실제
  WebSocket client가 `store-01/pos-01`로 접속한 상태에서: 단말 없는
  매장에 넣은 주문은 1초도 안 돼 자동 거절; 그 단말이 붙어 있는 매장에
  넣은 주문은 `PLACED`로 남고 push 프레임이 그 client에 도착; 이 서버의
  중계로 수락하면 Order의 `acceptedBy`가 `pos-01`로 실제 기록됨(중계의
  응답만이 아니라 Order를 직접 조회해서 재확인); 이미 수락된 주문을
  다시 수락하면 409. 세션 앞부분에 쓴 브라우저 자동화 도구는 자기가
  navigate한 것과 다른 port로 나가는 요청을 자체적으로 막고 있었는데,
  겉으론 연결 실패처럼 보였지만 실제로는 앱이 아니라 그 도구의
  문제였다 — "브라우저가 연결이 안 된다"는 결과가 curl과 다르게 나올 때
  다시 떠올릴 만한 일이다.
- **지금 상태**: Order가 Redis에 발행하고 Device Server가 구독하는 것 —
  둘 다 라이브 검증됐고, Redisson adapter가 처음으로 실제 돌아본
  순간이다. [`sun-moon-terminal-pos`](https://github.com/schware/sun-moon-terminal-pos)
  (React + TypeScript)를 만들어 `:8000` hub에 정적 파일로 배포했고,
  일부러 공개·미인증 상태로 뒀다 — 이 시스템의 모든 주문은 설계상
  샘플이라, 지금은 그 화면 뒤에 보호할 게 없다. 그 경계는 진짜 주문이
  생기는 날 다시 봐야 한다. KDS, DID, 그리고 채널 앱 둘(주문 접수,
  배달)은 아직 없다. 강제도 안 되는 것: owner의 규칙 — 한 매장에 단말이
  여럿일 수 있지만 주문을 받는 건 하나뿐 — 이 자리가 없다. BO가 매장을
  알아야 하고 `receivesOrders` 플래그가 있어야 하는데, Device Server와
  BO 사이의 서비스 간 인증 결정이 안 돼서 막혀 있다. owner의 예전 실전
  설계(PUSH-OMS/OMS/RIMS/DV-POS)를 참고로 공유받아 두 가지를 정리했다:
  BO가 RIMS 역할을 하고, PUSH-OMS는 그 시스템에 WebSocket이 없어서
  나뉜 것이었으므로 여기는 그 분리가 필요 없다.
- **무엇, 2026-09-10**: 채널 앱 둘 중 하나 — 주문 접수(주문 채널) —를
  `sun-moon-java-platform-channel-order`로 만들었다. 매장 선택 → 메뉴
  선택 → 주문, 그게 전부다. 매장 10개(1매장~10매장)와 매장당 메뉴
  10개(1메뉴~10메뉴)는 고정 카탈로그로 코드에 있고, DB도 로그인도 없다
  — 이 화면 뒤에 지킬 게 없다는 건 위 POS 채널과 같은 이유다. BO의
  ADR-0005 패턴(Spring + 같은 jar에 내장된 React, same-origin, CORS
  없음)을 그대로 따랐고, 화면이 매장/메뉴 두 개뿐이라 client 쪽 라우터도
  없다. Order에는 `menuName` 필드를 추가해 기존 `data JSONB` 컬럼에
  얹었다(마이그레이션 불필요 — `storeId`가 실제 컬럼이 필요했던 것과
  다르다, 그건 쿼리 대상이라서). owner가 명시적으로 오늘 범위에서 뺀
  것: POS 단말 켜짐 여부로 매장 CLOSE 표시하는 기능, 그리고 그걸 위한
  주기적 live-check(Batch/Job-Step-Chunk 엔진이 아니라 Order 자신의
  `AcceptanceTimeout`처럼 Spring `@Scheduled`가 맞는 방식이라는 점은
  확인했지만 구현은 다음으로 미뤘다). `:8000` hub에 자기 port(8085)로
  가는 카드를 올렸다 — BO와 같은 패턴이고, 정적 파일만 올리는 POS와는
  다르다. **GitHub에는 아직 없다** — 이 세션의 자동 실행 권한 밖의
  동작(새 public repo 생성)이라 로컬 커밋만 해두고, 서버에는 소스를
  직접 복사해 빌드·배포해 실물로 검증했다.
- **무엇, 2026-09-10**: BO에 메뉴 등록 화면을 추가했다 —
  메뉴코드/이름/이미지 URL/설명, 대/중/소 옵션은 이번에 뺐다. 가격은
  Device의 `deviceType`처럼 그냥 컬럼이 아니라, `menu_prices`라는
  이력 테이블이다: 행마다 시작일~종료일(종료일 null = 진행중)을 갖고,
  "메뉴 연동시" 쓸 가격은 조회하는 날짜가 걸리는 행을 찾아서 정한다.
  같은 메뉴에 기간이 겹치는 두 행은 등록 시점에 409로 막는다 — 두
  행이 어느 날의 가격에 대해 서로 다르게 말하는 상황 자체를 만들지
  않기 위해서다. BO의 `schema.sql` 컨벤션(Order와 달리 아직 Flyway로
  안 옮김)을 그대로 따랐고, 화면도 Device/CommonCode 패턴을 재사용하되
  메뉴 목록 아래 가격 이력 패널을 붙이는 master-detail 구조는 이번에
  처음 생겼다.

## 제안됨 (아직 일정 없음) — C++: 범위를 좁힌 C++20 coroutine IOCP 해법

역시 ADR-0008에 따라, **아직 확정 안 됨**.

- ADR-0004는 C++20 coroutine(`co_await` + Boost.Asio/cppcoro)을 원래
  백화점 시스템의 completion-callback 파편화 문제에 대한 같은 언어
  안에서의 해법으로 이름 붙였다 — `sun-moon-c-server`의 순수 C
  `batch_runner`(coroutine도 class도 없음)와는 다른 것.
- 작고 범위가 명확한 데모: 같은 IOCP 메커니즘 위에서 선형으로 읽히는
  비동기 코드, 깔끔한 객체 분해 — Go CSP 유비 하나만 있는 것보다 "실제
  언어로, 실제 고친 것"이라는 더 날카로운 증거물.

## 명시적 scope-cutting 규칙

시간이 부족해지면, Phase 1/1.5/2의 깊이를 깎기 전에 이 목록의 아래쪽(Phase
4, 그다음 Phase 3 후반부)부터 잘라낸다 — 이게 등장할 만한 어떤 인터뷰
대화에서도, 작지만 완성되고 잘 문서화된 시스템이 방대하지만 절반만 만든
시스템보다 낫다. Phase 1.5(ADR-0006)를 추가한 건 Phase 4(TypeScript
Dashboard)가 통째로 잘릴 가능성을 낮추는 게 아니라 오히려 높인다 — 이건
의도적인 트레이드오프였다: Batch는 8년의 전문성(ADR-0005)을 직접
증명하고, Dashboard는 전혀 경험 없는 스킬을 증명한다. 둘 중 하나를
골라야 한다면 Phase 1.5가 이긴다.

## 아직 안 정한 것

- "Q4 2026"이 "전환 전에 포트폴리오가 인터뷰 준비 완료 상태여야 한다"는
  뜻인지, "전환 직후에도 계속 만들어간다"는 뜻인지(ADR-0002에서 열어둔
  부분) — Phase 1.5가 이미 Phase 3를 원래 Q4 타겟 너머로 밀어냈으니
  (위 그 Phase의 메모 참고) 이제 더 시급한 질문이 됐다.
- Go/TypeScript/Rust의 재정리된 모양(Phase 2/4, 다음 세션에 재검토) —
  본인이 다음 대화 전에 직접 다시 생각해보는 중이다; 확인 없이 지금의
  Phase 2/4 내용을 여전히 유효한 것으로 가정하지 말 것.
- 위 Java/C++ 제안이 확정될지, 확정된다면 Phase 1.6 및 재정리된
  Go/TS/Rust 계획과 어떤 순서로 갈지 — 전부 다음 세션 논의 주제다.
- ADR-0009에 따른 Java의 Platform 아키텍처: DI/라우팅 방식, build tool,
  repo 이름, 그리고 (ADR-0008의 원래 Batch 전용 범위 대비) 전체 Platform
  범위가 실제로 만들어질 것인지.

## 이미 정한 것 (기록용)

- GitHub 공개 범위: `sun-moon-python-platform`과 이 저장소 둘 다
  Public이다, 의도적으로 — 여기 담긴 개인적/타임라인 내용까지 포함해서.
  본인의 현재 회사 대표도 이미 알고 있는 상황이고, 공개 상태 자체를
  결심을 다지는 장치로 의도적으로 쓰고 있다.
- GitHub 공개 범위, 2026-09-08 확대: `sun-moon-java-platform` 계열
  (Order/KDS/Delivery, 그리고 archived된 이전 `-jetty` 저장소들)과
  `sun-moon-app`도 같은 이유로 이제 Public이다 — 기본값이 아니라
  의도적으로. archived 저장소는 unarchive → public 전환 → 다시 archive
  순서로 처리했다(GitHub API가 archived 저장소의 공개 범위 변경 자체를
  거부하기 때문). `Debian-Setting`은 홈 서버의 실제 설정을 담고 있어서
  Private으로 남겨뒀다.
