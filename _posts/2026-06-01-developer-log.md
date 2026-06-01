---
title: "개발일지 — 2026-06-01 (IDE 경고 · 문자 시드 볼 UI)"
excerpt: "Spring IDE Problems 정리와 문자 시드 미리보기 볼 구간 색·번호 표시 수정."
categories: [deVlog]
tags: [planet645, spring, java, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-01 21:58:07 +0900
last_modified_at: 2026-06-01 22:46:25 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** Spring IDE Problems 정리·통합 테스트 Mock 공통화에 이어, **문자 시드 미리보기**에서 로또 볼 **구간 색**과 **번호 6개**가 안정적으로 보이도록 Thymeleaf 마크업을 정리했다.

## 1. 오늘 목표

### Planet645

**2026-05-31 이월**

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [ ] (선택) `RECOMMEND-V1-E2E-01` · `AD-MISSION-BETA-01`

**당일 신규**

- [x] IDE Problems — 미사용·deprecated·null safety 경고 정리 (`com.mockdan.life-saver-lotto`)
- [x] 문자 시드 미리보기 — 로또 볼 구간 색·번호 표시 (`text-seed-preview.html`)

### 개발 운영

**2026-05-31 이월**

- [ ] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate

**당일 신규**

- [x] `@SpringBootTest` 공통 Mock `@Import` (`SpringBootTestExternalDependencyMocks`)
- [x] `SECURITY-SECRETS-CI-01` — `quality-gate` `secrets-guard` job · runbook §4.1

---

## 2. 주요 변경 사항

### 2.1 IDE Problems · 경고 정리 (Planet645)

| 유형 | 조치 | 대표 경로 |
|------|------|-----------|
| 미사용 필드·상수 | 제거 | `CreditBatchServiceImpl`, `ProfileSetupServiceImpl`, `TermsDeemedConsentBatchService`, `PythonScoringServiceImpl`, `MlFeatureVectorMapper` |
| deprecated 호출 | `NotificationService.saveInAppNotification` · `BillingCheckoutService` 인터페이스 정리 | `NotificationDispatcher`, `BillingCheckoutService` |
| `Map.merge` + `Long::sum` null safety | `addCounts(Long, Long)` | `DrawStatisticsSnapshotServiceImpl` |
| HTTP 타임아웃 | `Duration` 오버로드 | `MlRestTemplateConfig`, `LlmExplainServiceImpl`, `PythonScoringServiceImpl` |
| 불필요 `@SuppressWarnings` | 제거 | `UserTermsAgreementTemporalRepository` |
| 테스트 | `verifyNoInteractions` · `assertNotNull` | `WinNumServiceImplTest`, `ScoredRecommendationServiceImplTest` 등 |

### 2.2 통합 테스트 Mock 공통화 (개발 운영)

| 산출 | 경로 |
|------|------|
| `@TestConfiguration` + `@Import` | `src/test/java/.../testsupport/SpringBootTestExternalDependencyMocks.java` |
| 적용 | CSRF·세션·헤더·billing dormant·OpenAuth 등 8개 `@SpringBootTest` |

### 2.3 문자 시드 미리보기 · 로또 볼 UI (Planet645)

| 항목 | 내용 |
|------|------|
| 증상 | `/recommend/text-seed/preview` — 볼이 단색·번호 누락·1개만 표시 |
| 원인 | 커스텀 `.number-ball`(단색) → `lotto-ball` 프래그먼트 `th:replace`/`th:insert`가 레이아웃·`th:each` 조합에서 `num` 미전달 |
| 해결 | `th:each` + `th:text` 유지, `lotto-number lotto-ball-lg` + `lotto-1`~`lotto-5` `th:classappend` (`design-system.css`) |
| 경로 | `templates/content/recommend/text-seed-preview.html` |

### 2.4 secrets-guard CI 연동 (개발 운영)

| 산출 | 경로 |
|------|------|
| `secrets-guard` job · `needs` gate | `.github/workflows/quality-gate.yml` |
| Spring/ML/LLM checkout 후 guard | `scripts/check-secrets-guard.sh` |
| runbook §4.1 | `artifact/runbook/security/secrets-governance.md` |
| 보드·아카이브 | `SECURITY-SECRETS-CI-01` |

---

## 3. 문제와 해결

### 3.1 IDE 경고 유형별 (Planet645)

| 문제 | 원인 | 해결 |
|------|------|------|
| 필드 미사용 | 생성자 주입만 되고 본문 미참조 | 필드 제거 또는 테스트에서 검증 추가 |
| deprecated `publish` / `handlePaymentSucceeded` | 레거시 표시가 내부 호출까지 전파 | 비deprecated API·엔드포인트만 `@Deprecated` 유지 |
| `Long::sum` + `merge` | 원시 `long` 시그니처 vs `BiFunction<Long,…>` | `addCounts` 헬퍼 |
| `@MockBean` 미사용 필드 | Spring만 쓰고 테스트 본문 미참조 | `@Bean` mock 설정 클래스로 이전 |

### 3.2 문자 시드 볼 UI (Planet645)

| 문제 | 원인 | 해결 |
|------|------|------|
| 단색 파란 볼 | 페이지 전용 `.number-ball` CSS | 공용 `lotto-*` 클래스 |
| 번호 없음·볼 1개 | `th:block` + `th:insert`/`th:replace` + `th:each` 동일 노드 | 인라인 `span` + `th:each` + `th:text` + `th:classappend` |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- IDE **Problems는 javac `-Xlint:deprecation`과 목록이 다를 수 있다** — 언어 서버 null analysis·미사용 로컬 변수는 별도 스캔.
- `@MockBean` 필드 대신 **`@TestConfiguration` `@Bean` mock** 이 미사용 필드 경고를 줄이면서 동일 컨텍스트를 만든다.
- 로또 볼 색은 **CSS만이 아니라 마크업·Thymeleaf 전달**이 맞아야 한다 — 프래그먼트가 깨지면 같은 페이지에서 **인라인 `lotto-*` 클래스**가 더 안전하다.

---

## 5. 다음 액션

### 5.1 미완

> **이월:** [2026-05-31-developer-log.md](2026-05-31-developer-log.md) §5.1.

#### Planet645

**2026-05-31 이월**

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [ ] (선택) `RECOMMEND-V1-E2E-01` · `AD-MISSION-BETA-01`

#### 개발 운영

**2026-05-31 이월**

- [ ] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| IDE Problems · 경고 정리 | [§2.1](#21-ide-problems--경고-정리-planet645) |
| 통합 테스트 Mock 공통화 | [§2.2](#22-통합-테스트-mock-공통화-개발-운영) |
| 문자 시드 미리보기 볼 UI | [§2.3](#23-문자-시드-미리보기--로또-볼-ui-planet645) |
| secrets-guard CI | [§2.4](#24-secrets-guard-ci-연동-개발-운영) |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| 환경 표기 | 로컬 prod-profile 회귀 (staging 표기 없음) |
| 전일 일지 | [2026-05-31-developer-log.md](2026-05-31-developer-log.md) |
| 검증 | `./gradlew compileJava compileTestJava` · 관련 단위·통합 테스트 |
