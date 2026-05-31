---
title: "개발일지 — 2026-05-30 (번호 추천 v1 · 2축 · FAST3/ML5)"
excerpt: "동일 6번호 버그 원인 규명 → Q&A 1–25 설계 SoT → P1–P5 구현(2축·4단계 UI·동점 redraw). Gradle 469 green. smoke·일부 단위 테스트는 미착."
categories: [deVlog]
tags: [planet645, recommend, ml, spring, thymeleaf, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-30 23:47:43 +0900
last_modified_at: 2026-05-31 17:34:43 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** tier/type만 바꿔도 **12,13,18,27,33,34**가 반복되던 원인(as-built Rule top-6)을 확인 → **Q&A 1–25** 설계 확정 → **`RECOMMEND-NUMBERS-V1-01`** P1–P5 구현 · `./gradlew test` **469 green**. 로컬 smoke·`CombinationGenerator` 테스트는 **미착**.

## 1. 오늘 목표

### Planet645

- [x] 동일 번호 버그 원인 분석 · v1 설계(Q&A 1–25) 확정
- [x] P1–P5 구현 — 4단계 UI · 2축 · FAST3/ML5 · 운명·동점 redraw
- [ ] 로컬 smoke — tier·축 조합별 번호 분기 확인

### 개발 운영

- [x] design/decisions/plan SoT · as-built · 보드 `RECOMMEND-NUMBERS-V1-01` 아카이브
- [ ] (선택) Playwright 추천 시나리오 v1 4단계 반영

---

## 2. 주요 변경 사항

### 2.1 설계 Q&A 1–25 · 문서 SoT (Planet645)

사용자 제안(풀·조합·동점·크레딧)을 코드·기존 plan과 대조한 뒤 **순차 Q&A**로 확정.

| 산출 | 경로 |
|------|------|
| Design SoT | `artifact/design/recommendation-ml/numbers-recommendation-design-v1.md` (Q&A 1–25, §13 Q-1~5) |
| Decisions | `artifact/design/decisions/numbers-recommendation-decisions-v1.md` |
| Implementation plan | `artifact/plan/numbers-recommendation-implementation-plan-v1.md` |
| 보드 | `RECOMMEND-HOT-COLD-POOL-01` → **`RECOMMEND-NUMBERS-V1-01`** |

**핵심 결정(요약):** 독립 **2축**(풀 HOT/COLD/NONE **25개** · ML선택 HOT/COLD/NONE) · FAST **3→1** / ML **5→1** · 고정≤5·제외≤20 · R-1~R-4 · ML fallback **과금 중단** · 동점 **redraw**(ML 재호출 없음) · 크레딧 FAST1/ANALYSIS3/PREMIUM6 · redraw ANALYSIS1/PREMIUM3.

### 2.2 문제 배경 — tier만 바꿔도 동일 번호 (Planet645)

| 항목 | as-built(이전) | v1(이후) |
|------|----------------|----------|
| 입력 | tier/type → persist 라벨만 | 풀 축1 + 고정·제외 + tier + ML 축2 |
| 조합 | Rule **top-6** 고정 | FAST 3조합 랜덤 1 / ML 5조합 축2 선정 |
| ML | 번호 미변경·fallback 허용 | 5조합 채점 · fallback **503·무료** |

### 2.3 P1 — 4단계 UI · 입력 검증 (Planet645)

`select.html` step1(풀·고정·제외) → step2(tier) → step3(ML축) · `POST /recommend/select/input` · 세션 `RECOMMEND_INPUT_SESSION` · `RecommendationUserInput` / `RecommendationInputValidator`.

### 2.4 P2 — 풀·조합·룰 (Planet645)

`recommendation.strategy.*` — `NumberPoolBuilder`(25) · `CombinationGenerator` · `AbnormalCombinationRules` · `RecommendationEngineService`. 단위 테스트: `NumberPoolBuilderTest` · `AbnormalCombinationRulesTest` (**`CombinationGeneratorTest`는 미작성**).

### 2.5 P3 — Tier 분기 · ML 축2 (Planet645)

`RecommendationFlowServiceImpl` · `MlScoringPayloadFactory.buildRequestFromCombinations` · `MlCombinationSelector` · `DefaultFinalSelectionStrategy` · `RecommendProductResolver`(ML 전 `abortIfMlDegraded=true`).

### 2.6 P4 — 동점 · 운명 · redraw (Planet645)

`result.html` fate/동점 UI · `POST /recommend/redraw-tie` · `RECOMMEND_REDRAW_REF_TYPE` · `FinalRecommendationDTO` 세션 필드(`tiedCandidateIds`, `storedCombinations`).

### 2.7 P5 — 해설 · as-built · 테스트 (Planet645)

ANALYSIS 고정 1줄 · PREMIUM `LlmExplainService`(플래그) · `artifact/as-built/recommendation-pipeline.md` 갱신 · 테스트 리라이트(`RecommendationFlowServiceImplTest`, `RecommendControllerTest` 등) — **469 green**. 레거시 `type=ai|hot|cold` execute 호환.

### 2.8 보드·아카이브 (개발 운영)

`RECOMMEND-NUMBERS-V1-01` → `archive-history-2026-06-01.md` · `board.md` Active 비움 · task count **55**.

---

## 3. 문제와 해결

### 3.1 Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| 전략 변경해도 동일 6번호 | `recommend(from,to)`만 호출 · Rule top-6 | v1 엔진 + tier·2축 파이프라인 |
| ML tier fallback 후 과금 | resolver 일부만 abort | ML 전 tier 503 + consume 전 차단 |
| 초안 풀 23개 vs 구현 | Q&A에서 **25**로 확정 | `NumberPoolBuilder.POOL_SIZE=25` |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- **설계(Q&A) → plan → 구현**을 같은 날 이어가려면 SoT 경로·보드 ID를 먼저 고정하는 편이 diff 추적에 유리하다.
- tier와 풀·ML선택 축을 분리하지 않으면 UI·크레딧·persist가 한 경로에 뭉개진다.

---

## 5. 다음 액션

### 5.1 미완

> **이월:** [2026-05-27-developer-log.md](2026-05-27-developer-log.md) §5.1 — 5/28–29 일지 없음.

#### Planet645

**5/30에서 신규(추천 v1 후속)**

- [ ] 로컬 `/recommend/select` 4단계 smoke — tier·축별 **서로 다른 번호**
- [ ] 로컬 prod-profile 회귀(execute · result · redraw-tie)
- [ ] `CombinationGenerator` · `RecommendationInputValidator` 단위 테스트

**5/27 이월**

- [ ] 추천·LLM 필터·ML 스코어링 **품질 verify** — v1 구현(5/30) 후 smoke·prod-profile 회귀로 마무리
- [ ] 주소검색 **수동** — `/profile/setup` 팝업 → 저장 → `/mypage/profile` 재조회
- [ ] PASS/NICE 연동 · `CI_ANCHOR_ENABLED` · 동일 CI 재가입 E2E
- [ ] (선택) Ad·미션 베타 노출 — prod-profile 회귀 후

#### 개발 운영

**5/30에서 신규**

- [ ] (선택) Playwright 추천 시나리오 v1 4단계

**5/27 이월**

- [ ] `check-secrets-guard.sh`를 quality-gate에 연동 (baseline 정리 후)
- [ ] dependency-check **CVSS 게이트** 단계적 강화
- [ ] (선택) 새 백로그 항목을 보드에 등록 후 `/task-next`

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| Q&A 1–25 · design/decisions/plan | [§2.1](#21-설계-qa-125--문서-sot-planet645) |
| P1–P5 구현 | [§2.3–§2.7](#23-p1--4단계-ui--입력-검증-planet645) |
| as-built · Gradle test | [§2.7](#27-p5--해설--as-built--테스트-planet645) |
| 보드 아카이브 | [§2.8](#28-보드아카이브-개발-운영) |

---

## 6. 기준

- Design: `artifact/design/recommendation-ml/numbers-recommendation-design-v1.md`
- Plan: `artifact/plan/numbers-recommendation-implementation-plan-v1.md`
- As-built: `artifact/as-built/recommendation-pipeline.md`
- Code: `com.mockdan.life-saver-lotto` — `recommendation.strategy.*`, `RecommendationFlowServiceImpl`, `RecommendController`
- 전일 일지: [2026-05-27-developer-log.md](2026-05-27-developer-log.md) (5/28–29 없음)
