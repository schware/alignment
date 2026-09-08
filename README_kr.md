*🇬🇧 English version: [README.md](README.md)*

# Alignment — 이직 준비 과정 기록

- 이 저장소는 단순한 학습 노트가 아니라, 시스템 Architecture의 의사결정을 기록하듯이 이직 준비 과정에서 내린 결정과 그 근거를 ADR (Architecture Decision Record) 형태로 기록하는 공간이다.

## 왜 작성을 하고 있는가?

- 커리어 전환을 준비하는 과정에서는 크게 두 종류의 결정이 발생한다.

  - 기술적 측면: 무엇을 만들 것인지, 어떤 기술과 언어를 사용할 것인지
  - 커리어 측면: 어떤 스킬을 우선적으로 학습할 것인지, 어떤 직무를 목표로 할 것인지, 시간을 어떻게 투자할 것인지
- 기술적 결정은 각 기술 저장소의 ADR에 기록하고, 이 저장소는 커리어 결정과 그 근거를 기록하기 위해 작성되었다.

결국 이 저장소는 기술적 성장과 커리어 성장의 과정을 함께 정리하고 추적하기 위한 기록 공간이다.

## 기술 저장소들과의 관계

- 기술 저장소들은 각각 하나의 시스템이다. 코드, 테스트, Architecture와 관련된 기술적 의사결정을 각자의 ADR로 관리한다.
  이 저장소는 그 시스템들을 만드는 방향성이다. 왜 해당 시스템들을 선택했는지, 왜 특정 기술을 학습하는지, 왜 이 시점에 해당 직무를 목표로 하는지를 기록한다.
  두 종류의 저장소를 분리한 이유는 목적이 다르기 때문이다. 기술 저장소들은 엔지니어링 결과물을 설명하고, 이 저장소는 커리어 전략과 방향성의 과정을 설명한다.

- 하나의 결정이 기술과 커리어 모두에 영향을 주는 경우에는 양쪽 ADR이 서로를 참조한다.

현재 이 방향성이 가리키는 기술 저장소:

- [`sun-moon-python-platform`](https://github.com/schware/sun-moon-python-platform)
- [`sun-moon-c-server`](https://github.com/schware/sun-moon-c-server)

그 외 병렬 트랙(모바일 앱, 그리고 진행 중인
[`sun-moon-java-platform`](https://github.com/schware/sun-moon-java-platform)
현대화 — 새 포트폴리오 근거를 만드는 게 아니라 기존 Java/Spring Boot
baseline을 유지하는 작업)은 `ROADMAP_kr.md` 참고.

## 구조

```
docs/adr/       ADR — 커리어/포트폴리오 전략 결정 하나당 하나
ROADMAP.md      살아있는, Phase별 계획 (Phase가 끝날 때마다 갱신)
```

`docs/adr/0001-use-adrs-for-career-decisions_kr.md`부터 시작해서(이게 왜
존재하는지), `0002`(정직한 시작점), `0003`(그 위에 세운 전략) 순서로 읽고,
그다음 구체적인 계획은 `ROADMAP_kr.md`를 보면 된다.

## 여기 담긴 내용에 대한 참고

- 이 저장소에는 현재 역량에 대한 솔직한 평가, 학습 계획, 직무 전환 목표 시점 등 개인적이고 시간에 민감한 정보가 포함될 수 있다.

따라서 GitHub 공개 여부는 기본 설정에 맡기지 않고 의도적으로 결정해야 하며, 관련 내용은 ROADMAP_kr.md의 "아직 결정하지 않은 사항" 섹션에서 관리한다.
