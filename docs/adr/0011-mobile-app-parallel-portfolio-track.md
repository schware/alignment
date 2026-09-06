*🇰🇷 한국어 버전: [0011-mobile-app-parallel-portfolio-track_kr.md](0011-mobile-app-parallel-portfolio-track_kr.md)*

# ADR-0011: Cross-platform mobile app (meeting AI-summary + schedule sync via GitHub) as a parallel, non-competing portfolio track

- **Status**: Accepted
- **Date**: 2026-09-06
- **Deciders**: Project owner

## Context

ADR-0003 committed to **one** Platform project (`sun-moon-python-platform`
plus its polyglot components) as the primary portfolio vehicle, deliberately
spanning Backend/Platform, AI/ML, and DevOps/SRE signals, with an explicit
scope-cutting discipline (`ROADMAP.md`) to avoid diluting that single
narrative.

Separately, the same day, the owner started a new project outside that
repo: a cross-platform (iOS/Android) mobile app at `D:\Claude_Code\App`,
covering two personal-use features — (1) paste a meeting transcript, get an
AI-generated Markdown summary (Claude API), committed to a GitHub repo, and
(2) a morning-plan / evening-review daily schedule, also synced to GitHub as
Markdown. The owner confirmed this is intended as an actual portfolio
signal (mobile + third-party AI/API integration — a skill area the Platform
does not otherwise cover), not a career-irrelevant hobby tool, but also
confirmed it should run **in parallel**, drawing no time from the Q4 2026
Platform roadmap's phases (1.6, 2, 3, 4).

This ADR exists to record that boundary explicitly, the same way ADR-0003
recorded the "one Platform, not three toy projects" boundary — so that if
this track ever starts drawing on Platform time, that drift is visible and
requires its own decision rather than silently eroding the Q4 timeline.

## Decision

Build the mobile app as a **separate, parallel track**, explicitly outside
`ROADMAP.md`'s Phase sequencing and time budget:

- **Stack**: Expo (React Native + TypeScript) — chosen specifically because
  it can be built end-to-end on Windows with no local Mac (EAS Build
  handles iOS compilation in the cloud; local testing goes through the Expo
  Go app on a physical device instead of a simulator).
- **Scope (MVP, built this session)**: 4 screens (Home, Meeting Summary,
  Schedule, Settings); GitHub sync via a user-supplied Personal Access
  Token (no GitHub App/OAuth yet); credentials (GitHub PAT, Anthropic API
  key) stored on-device via `expo-secure-store`; meeting input is
  paste-only (no in-app recording/STT yet); daily schedule stored locally
  (`AsyncStorage`) and pushed to GitHub as Markdown on demand.
- **Repo**: `D:\Claude_Code\App`, independent of `sun-moon-python-platform`
  and of this Alignment repo; no git remote/public-visibility decision made
  yet.
- **Time accounting**: this track is not counted against ADR-0003's "one
  Platform" scope discipline and does not appear as a Phase in
  `ROADMAP.md`'s Q4 2026 sequencing — it is listed there only as a
  parallel-track note, not a scheduled Phase.

## Alternatives considered

- **Skip the mobile track entirely, stay 100% inside
  `sun-moon-python-platform`'s scope** — rejected: mobile/cross-platform
  development and end-to-end third-party API integration (GitHub, Claude)
  from a mobile client are a distinct, employer-visible signal the Platform
  doesn't produce, and the owner independently wants a working personal
  tool regardless of portfolio value.
- **Fully native (separate Swift and Kotlin codebases)** — rejected:
  doubles implementation effort and requires a local Mac for the iOS side,
  which isn't available; not justified for what is meant to be a fast,
  personal-scale tool rather than the primary portfolio vehicle.
- **Flutter** — considered; not chosen for now. Its iOS build path on
  Windows still typically needs a Mac or a separate cloud CI (e.g.
  Codemagic), less turnkey than Expo/EAS. Staying in the JS/TS ecosystem
  also has a minor side benefit of overlapping with the eventual TypeScript
  Dashboard track (ROADMAP.md Phase 4) rather than adding a fully separate
  language surface.
- **Fold it into `ROADMAP.md`'s Q4 sequencing as a real Phase** — rejected
  by explicit owner decision: keeping it parallel avoids forcing a
  scope-cutting tradeoff against Phase 1.6/2/3 (per ADR-0003's existing
  discipline), and the owner does not want this track to compete for that
  time budget.

## Consequences

- A second, independent codebase now exists alongside
  `sun-moon-python-platform`. Whether it eventually becomes a public,
  portfolio-linked repo or stays a private personal tool is undecided —
  the same kind of visibility decision already tracked as open for the
  Platform repo (see this repo's README note on deliberate GitHub
  visibility choices).
- This adds a fifth technology surface (React Native/TypeScript mobile) to
  the owner's overall body of work, but by this ADR's decision it is
  explicitly **not** counted inside ADR-0003's "one Platform, three
  job-family signals" accounting — this repo's scope-cutting rules
  (`ROADMAP.md`) do not apply to it.
- The MVP stores the GitHub PAT and Anthropic API key directly on-device
  and calls both APIs straight from the mobile client. Acceptable for a
  single-user personal tool; would need a backend proxy before this could
  ever be shown as a multi-user artifact — noted in the app repo's own
  README as a known limitation, not re-litigated here.
- Because this track is parallel by decision, any time it actually consumes
  is *not* pre-authorized against Q4 2026 scope-cutting headroom. If it
  starts visibly slipping into time otherwise spent on Phase 1.6/2/3, that
  drift needs its own ADR rather than being absorbed silently.

## References

- ADR-0003 (this repo) — the "one Platform, not three toy projects"
  discipline this ADR deliberately carves an exception around.
- `ROADMAP.md` — updated with a short parallel-track note (not a Phase
  entry) pointing back to this ADR.
- This session's build: `D:\Claude_Code\App` (Expo/TypeScript scaffold;
  `src/lib/{claude,github,scheduleStore,secureStore}.ts`; `README.md`).

---

**To amend this ADR**: don't edit this file — write a new one with a new
number and note "Supersedes ADR-0011." Preserving the trail is the entire
point of using ADRs.
