---
title: "개발일지 — 2026-06-01 (IDE Problems · 경고 정리)"
excerpt: "Spring 소스·테스트 IDE Problems 정리 — 미사용 필드·deprecated 호출·null type safety·통합 테스트 Mock 공통화."
categories: [deVlog]
tags: [planet645, spring, java, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-01 21:58:07 +0900
last_modified_at: 2026-06-01 21:58:07 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** Cursor Problems에 뜨던 **미사용 필드·deprecated API·null type safety·불필요 suppress** 를 Spring·테스트 코드에서 정리. 통합 테스트 OAuth/DB Mock은 `testsupport` 설정으로 통합.

## 1. 오늘 목표

### Planet645

**2026-05-31 이월**

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [ ] (선택) `RECOMMEND-V1-E2E-01` · `AD-MISSION-BETA-01`

**당일 신규**

- [x] IDE Problems — 미사용·deprecated·null safety 경고 정리 (`com.mockdan.life-saver-lotto`)

### 개발 운영

**2026-05-31 이월**

- [ ] `SECURITY-SECRETS-CI-01` — secrets-guard CI
- [ ] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate

**당일 신규**

- [x] `@SpringBootTest` 공통 Mock `@Import` (`SpringBootTestExternalDependencyMocks`)

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

---

## 3. 문제와 해결

### 3.1 IDE 경고 유형별 (Planet645)

| 문제 | 원인 | 해결 |
|------|------|------|
| 필드 미사용 | 생성자 주입만 되고 본문 미참조 | 필드 제거 또는 테스트에서 검증 추가 |
| deprecated `publish` / `handlePaymentSucceeded` | 레거시 표시가 내부 호출까지 전파 | 비deprecated API·엔드포인트만 `@Deprecated` 유지 |
| `Long::sum` + `merge` | 원시 `long` 시그니처 vs `BiFunction<Long,…>` | `addCounts` 헬퍼 |
| `@MockBean` 미사용 필드 | Spring만 쓰고 테스트 본문 미참조 | `@Bean` mock 설정 클래스로 이전 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- IDE **Problems는 javac `-Xlint:deprecation`과 목록이 다를 수 있다** — 언어 서버 null analysis·미사용 로컬 변수는 별도 스캔.
- `@MockBean` 필드 대신 **`@TestConfiguration` `@Bean` mock** 이 미사용 필드 경고를 줄이면서 동일 컨텍스트를 만든다.

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

- [ ] `SECURITY-SECRETS-CI-01` — secrets-guard CI
- [ ] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| IDE Problems · 경고 정리 | [§2.1](#21-ide-problems--경고-정리-planet645) |
| 통합 테스트 Mock 공통화 | [§2.2](#22-통합-테스트-mock-공통화-개발-운영) |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| 환경 표기 | 로컬 prod-profile 회귀 (staging 표기 없음) |
| 전일 일지 | [2026-05-31-developer-log.md](2026-05-31-developer-log.md) |
| 검증 | `./gradlew compileJava compileTestJava` · 관련 단위·통합 테스트 |
