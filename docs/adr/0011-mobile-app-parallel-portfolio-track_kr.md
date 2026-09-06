*🇬🇧 English version: [0011-mobile-app-parallel-portfolio-track.md](0011-mobile-app-parallel-portfolio-track.md)*

# ADR-0011: 크로스플랫폼 모바일 앱(회의 AI 요약 + GitHub 일정 동기화)을, 로드맵과 경쟁하지 않는 별도 병렬 트랙으로 진행

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: 본인

## Context

ADR-0003은 `sun-moon-python-platform`과 그 polyglot 컴포넌트들로 이루어진
**하나의** Platform 프로젝트를 메인 포트폴리오 vehicle로 삼기로 결정했다 —
Backend/Platform, AI/ML, DevOps/SRE 세 시그널에 의도적으로 걸치면서,
`ROADMAP.md`의 명시적인 scope-cutting 원칙으로 그 서사가 희석되지 않게
관리한다.

같은 날, 본인은 이 저장소 밖에서 별도 프로젝트를 시작했다: `D:\Claude_Code\App`에
만드는 크로스플랫폼(iOS/Android) 모바일 앱으로, 두 가지 개인 用途 기능을
담는다 — (1) 회의 녹취록을 붙여넣으면 Claude API가 Markdown 요약을
생성하고 GitHub 저장소에 커밋하는 기능, (2) 아침 계획/저녁 확인 형태의
일일 일정을 마찬가지로 Markdown으로 GitHub에 동기화하는 기능. 본인은 이게
커리어와 무관한 취미 도구가 아니라 실제 포트폴리오 시그널(모바일 +
서드파티 AI/API 통합 — Platform이 다루지 않는 스킬 영역)로 의도한 것임을
확인했고, 동시에 이 작업이 **병렬로** 진행되어야 하며 Q4 2026 Platform
로드맵의 Phase(1.6, 2, 3, 4) 시간 예산을 전혀 가져가지 않아야 한다는 점도
확인했다.

이 ADR은 그 경계를 명시적으로 기록해두기 위해 존재한다 — ADR-0003이
"토이 프로젝트 세 개가 아니라 Platform 하나"라는 경계를 기록해둔 것과 같은
방식으로. 나중에 이 트랙이 실제로 Platform 시간을 가져가기 시작하면, 그
변화가 조용히 Q4 타임라인을 갉아먹는 대신 눈에 보이게, 그리고 별도의
결정을 거치게 하려는 목적이다.

## Decision

모바일 앱은 **별도의 병렬 트랙**으로 만든다, `ROADMAP.md`의 Phase 순서와
시간 예산 밖에서:

- **스택**: Expo (React Native + TypeScript) — 로컬 Mac 없이 Windows에서
  끝까지 빌드 가능하기 때문에 선택했다(iOS 컴파일은 EAS Build가 클라우드에서
  처리하고, 로컬 테스트는 시뮬레이터 대신 실기기의 Expo Go 앱으로 한다).
- **범위 (이번 세션에서 만든 MVP)**: 화면 4개(홈, 회의 요약, 일정, 설정);
  사용자가 직접 발급한 Personal Access Token으로 GitHub 연동(아직 GitHub
  App/OAuth 아님); GitHub PAT와 Anthropic API 키는 `expo-secure-store`로
  기기에 저장; 회의 입력은 텍스트 붙여넣기만 지원(앱 내 녹음/STT는 아직
  없음); 일일 일정은 로컬(`AsyncStorage`)에 저장하고 필요할 때 Markdown으로
  GitHub에 푸시.
- **저장소**: `D:\Claude_Code\App`, `sun-moon-python-platform`과 이
  Alignment 저장소 모두와 독립적이다. git remote/공개 범위는 아직 결정 안 됨.
- **시간 회계**: 이 트랙은 ADR-0003의 "Platform 하나" scope 원칙에
  포함되지 않고, `ROADMAP.md`의 Q4 2026 순서에도 Phase로 등장하지 않는다 —
  거기엔 병렬 트랙 메모로만, 예정된 Phase가 아닌 형태로 남긴다.

## Alternatives considered

- **모바일 트랙을 아예 안 하고 `sun-moon-python-platform` 범위 안에만
  머문다** — 기각. 모바일/크로스플랫폼 개발과, 모바일 클라이언트에서
  GitHub·Claude 같은 서드파티 API를 엔드투엔드로 통합하는 경험은 Platform이
  만들어내지 않는, 채용 담당자에게 보이는 별개의 시그널이다. 그리고 본인은
  포트폴리오 가치와 무관하게 실제로 쓸 개인 도구를 원한다.
- **완전 네이티브(Swift와 Kotlin을 각각 별도 코드베이스로)** — 기각.
  구현량이 두 배가 되고 iOS 쪽엔 로컬 Mac이 필요한데 지금 없다. 메인
  포트폴리오 vehicle이 아니라 빠르게 쓸 개인 규모 도구를 만드는 목적에
  정당화되지 않는다.
- **Flutter** — 검토했으나 지금은 채택 안 함. Windows에서 iOS 빌드 경로가
  보통 여전히 Mac이나 별도 클라우드 CI(Codemagic 등)를 필요로 해서
  Expo/EAS만큼 매끄럽지 않다. JS/TS 생태계에 머무는 것 자체도, 별도 언어
  표면을 하나 더 늘리는 대신 나중에 만들 TypeScript Dashboard 트랙
  (ROADMAP.md Phase 4)과 살짝 겹치는 부수적 이점이 있다.
- **`ROADMAP.md`의 Q4 순서 안에 실제 Phase로 넣는다** — 본인의 명시적
  결정으로 기각. 병렬로 유지하면 ADR-0003이 이미 세워둔 원칙(Phase
  1.6/2/3와의 scope-cutting trade-off)과 부딪히지 않는다. 본인은 이 트랙이
  그 시간 예산을 두고 경쟁하길 원하지 않는다.

## Consequences

- `sun-moon-python-platform` 옆에 두 번째 독립 코드베이스가 생겼다. 이게
  나중에 공개되어 포트폴리오에 링크되는 저장소가 될지, 비공개 개인 도구로
  남을지는 아직 정해지지 않았다 — Platform 저장소에 대해서도 이미 열려있는
  것과 같은 종류의 공개 범위 결정이다(이 저장소 README의 "의도적으로 GitHub
  공개 범위를 정해야 한다"는 메모 참고).
- 본인의 전체 작업물에 다섯 번째 기술 표면(React Native/TypeScript
  모바일)이 추가되지만, 이 ADR의 결정에 따라 ADR-0003의 "Platform 하나,
  세 직무군 시그널" 회계에는 **포함되지 않는다** — 이 저장소의
  scope-cutting 규칙(`ROADMAP.md`)은 이 트랙에 적용되지 않는다.
- MVP는 GitHub PAT와 Anthropic API 키를 기기에 직접 저장하고, 두 API를
  모바일 클라이언트에서 바로 호출한다. 개인 用途 단일 사용자 도구로는
  괜찮지만, 여러 사용자에게 보여줄 결과물이 되려면 백엔드 프록시가
  필요하다 — 앱 저장소 자체 README에 이미 알려진 제한사항으로 적어뒀고,
  여기서 다시 다루지 않는다.
- 이 트랙은 결정에 따라 병렬이므로, 실제로 여기 쓰인 시간은 Q4 2026
  scope-cutting 여유분으로 미리 승인된 것이 **아니다**. 만약 이게 Phase
  1.6/2/3에 쓰였어야 할 시간을 눈에 띄게 잠식하기 시작하면, 조용히
  흡수시키지 말고 그 드리프트 자체에 대한 별도 ADR을 써야 한다.

## References

- ADR-0003 (이 저장소) — 이 ADR이 의도적으로 예외를 만드는 "토이 프로젝트
  세 개가 아니라 Platform 하나" 원칙.
- `ROADMAP.md` — Phase 항목이 아니라, 이 ADR을 가리키는 짧은 병렬 트랙
  메모를 추가함.
- 이번 세션의 빌드: `D:\Claude_Code\App` (Expo/TypeScript 스캐폴드;
  `src/lib/{claude,github,scheduleStore,secureStore}.ts`; `README.md`).

---

**이 ADR을 수정하고 싶다면**: 이 파일을 고치지 말고 새 번호로 ADR을 만들어
"Supersedes ADR-0011"을 적는다 — 흐름이 끊기지 않게 남기는 게 ADR로 하는
이유 전부다.
