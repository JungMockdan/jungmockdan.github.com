---
title: "개발일지 — 2026-05-19 (선택 약관 철회·보드 정리)"
excerpt: "온보딩 스킵 완료 처리·선택 약관 단일 폼 sync·PHASE17 prod Step 5 수동 회귀·verify SoT."
categories: [deVlog]
tags: [planet645, spring, terms, mypage, marketing, oauth, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-19 10:00:00 +0900
last_modified_at: 2026-05-19 18:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 오늘 목표](#1-오늘-목표)
  - [Planet645](#planet645)
  - [개발 운영](#개발-운영)
- [2. 주요 변경 사항](#2-주요-변경-사항)
  - [2.1 `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` (Planet645)](#21-terms-optional-consent-withdraw-01-planet645)
  - [2.2 보드 아카이브 · 05-18 일지 정합 (개발 운영)](#22-보드-아카이브--05-18-일지-정합-개발-운영)
  - [2.3 보드 PHASE17 복원 · `executionOrder` 재정렬 (개발 운영)](#23-보드-phase17-복원--executionorder-재정렬-개발-운영)
  - [2.4 약관 PRD·design 역정합 — 0a (Planet645)](#24-약관-prddesign-역정합--0a-planet645)
  - [2.5 `MYPAGE-MYINFO-NAV-01` 착수 전 결정 메모 (Planet645 · 문서만)](#25-mypage-myinfo-nav-01-착수-전-결정-메모-planet645--문서만)
  - [2.6 Playwright — 설정 허브 카피 (Planet645)](#26-playwright--설정-허브-카피-planet645)
  - [2.7 수동 verify 0b · 워크플로 (개발 운영)](#27-수동-verify-0b--워크플로-개발-운영)
  - [2.8 Google OAuth — 대시보드 프로필·통계 (Planet645)](#28-google-oauth--대시보드-프로필통계-planet645)
  - [2.9 `lotto_*` runbook 문서 정합 (개발 운영)](#29-lotto_-runbook-문서-정합-개발-운영)
  - [2.10 prod-profile validate 드리프트 정리 (Planet645)](#210-prod-profile-validate-드리프트-정리-planet645)
  - [2.11 로컬 prod-profile 회귀 — 자동 Step (Planet645)](#211-로컬-prod-profile-회귀--자동-step-planet645)
  - [2.12 PHASE17 Step 5 — 수동 OAuth UI (`local` :8080)](#212-phase17-step-5--수동-oauth-ui-local-8080)
  - [2.13 PHASE17 Step 5 — **prod** `:8080` (2026-05-19)](#213-phase17-step-5--prod-8080-2026-05-19)
  - [2.14 온보딩 스킵 · 선택 약관 화면 (Planet645)](#214-온보딩-스킵--선택-약관-화면-planet645)
- [3. 문제와 해결](#3-문제와-해결)
  - [Planet645](#planet645-1)
    - [3.1 온보딩 스킵 시 `onboarding_completed` 미저장](#31-온보딩-스킵-시-onboarding_completed-미저장)
    - [3.2 알림 설정 저장 → `/terms-agreement`](#32-알림-설정-저장--terms-agreement)
  - [개발 운영](#개발-운영-1)
- [4. 배운 점](#4-배운-점)
- [5. 다음 액션](#5-다음-액션)
  - [5.1 미완](#51-미완)
  - [5.2 완료 (당일 — 상세는 §2)](#52-완료-당일--상세는-2)
- [6. 기준](#6-기준)

<!-- /code_chunk_output -->


> **하루 요약:** 온보딩 **스킵 시 완료 플래그 저장**·선택 약관 **단일 화면 sync**(마케팅 채널 분리). PHASE17 prod Step 5 수동 회귀(M2·M3 Pass, M1-4·M4 결함 확인 후 수정).

## 1. 오늘 목표

### Planet645

- [x] `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` — optional-terms 동의·철회 UX + 서비스
- [x] 단위 테스트 (`TermsConsentServiceTest`, `ProfileEditControllerTest`)
- [x] Google OAuth → `/dashboard` 프로필·통계 수동 verify

### 개발 운영

- [x] 보드 Active 아카이브 · [05-18 일지](/2026-05-18-developer-log) 다음 액션 정합
- [x] 수동 verify **0b** · verify 워크플로(`/verify-scaffold`, `/verify-next`)
- [x] `lotto_*` 스키마 runbook·PRD 문서 잔재 정리

---

## 2. 주요 변경 사항

### 2.1 `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` (Planet645)

as-built(`TERMS-OPTIONAL-CONSENT-MYPAGE-01`)는 **미동의 → 동의**만 가능했다. 본 태스크로 **동의 중 → 철회**를 같은 화면에 추가했다.

| 영역 | 내용 |
|------|------|
| **GET** | `pendingOptionalTerms` + `agreedOptionalTerms` |
| **POST** | `action=consent` + `agreedTermIds` / `action=withdraw` + `withdrawTermIds` |
| **철회** | `UserTermsAgreement.agreed=false`, `terms_consent_event` (`agreed=false`, `MARKETING_SETTINGS`) |
| **마케팅** | `TermsKeys.MARKETING`(TERM-004) 철회 시 `saveMarketingChannels(..., false×4)` |

### 2.2 보드 아카이브 · 05-18 일지 정합 (개발 운영)

| 항목 | 내용 |
|------|------|
| 보드 | `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` → `archive/archive-history-2026-05-24.md`, Active `(none)` |
| 일지 | [2026-05-18](/2026-05-18-developer-log) §5 — 완료 항목 `[x]`, §2.10 교차 링크 |

### 2.3 보드 PHASE17 복원 · `executionOrder` 재정렬 (개발 운영)

[05-13 일지](/2026-05-13-developer-log#5-다음-액션)에만 있던 prod validate·회귀 태스크가 `board.md` 이관 시 **누락**되어 있었음. 복원 후 밴드별 실행 순서 재정렬.

| Task | executionOrder | 비고 |
|------|----------------|------|
| `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` | `105` | high · 다음 착수 |
| `PHASE17-LOCAL-PROD-REGRESSION-01` | `106` | `blockedBy` 105 |
| `TERMS-MINOR-AGE-DECLINE-01` | `110` | `review_pending` |
| `AUTH-LOGOUT-ZERO-01` | `115` | PRD ✅ · design/plan |
| `MYPAGE-CREDIT-PAGE-01` | `200` | |
| `MYPAGE-MYINFO-NAV-01` | `210` | 선행 CREDIT-PAGE |
| `PHASE22-BILLING-SANDBOX-01` | `220` | |
| `PHASE22-BILLING-REFUND-01` | `230` | |
| `PROFILE-ADDRESS-SEARCH-01` | `240` | |
| `IDENTITY-CI-FREE-ABUSE-01` | `250` | |

- Metadata: `backlogRunOrder`, `selectionGuide`, executionOrder 밴드 표
- `artifact/plan/phase17-product-engineering-backlog.md` §C.5 — SoT `board.md`·선행 drift

### 2.4 약관 PRD·design 역정합 — 0a (Planet645)

`artifact/prd/terms-consent.md` §4.5·§6.1·§8#6, `artifact/design/terms-consent/01-schema-revision-guards.md` §5.4·§10.1#7.

**결정(2026-05-19→20):** ① A+C — 동의·철회 SoT `optional-terms`, 채널 `marketing-channels` / ② 일괄 채널 OFF 비제공 / ③ D — 재동의 배너 비채택·`/terms-agreement` / ④ 버전+시행일 UI / ⑤ §4.5·§6.1 경로 명시.

### 2.5 `MYPAGE-MYINFO-NAV-01` 착수 전 결정 메모 (Planet645 · 문서만)

1. **허브 URL** — `/mypage/info` 신설 vs `/mypage` 승격  
2. **드로어·하단탭** — 직링크 vs 허브 경유  
3. **nav 범위** — 구독·플랜·크레딧(CREDIT-PAGE 후)  
4. **설정 경계** — 메시지함 vs 알림 수신 설정 라벨  
5. **URL 유지** vs UI 스펙 경로 이전  

권장 초안: `/mypage/info` 허브 · 드로어 직링크 · 크레딧 nav는 CREDIT-PAGE 후.

### 2.6 Playwright — 설정 허브 카피 (Planet645)

`smoke/e2e/mypage-drawer.spec.ts`: 「내 정보 > 설정」·「약관 및 동의 내역」·`← 내 정보` back assertion 정합.

### 2.7 수동 verify 0b · 워크플로 (개발 운영)

| 항목 | 내용 |
|------|------|
| **0b** | [`TERMS-OPTIONAL-CONSENT-01-manual-verify.md`](../../artifact/ops/verify/TERMS-OPTIONAL-CONSENT-01-manual-verify.md) — M·C·K·W **Pass** (§6) |
| **함정** | `POST marketing-channels` + `curl -L` → 채널 전부 OFF 오탐 — runbook 금지 |
| **철회 후** | `is_agreed=0` → marketing-channels **302 `/terms-agreement`** |
| **워크플로** | `artifact/ops/verify/` · `/verify-scaffold` · `/verify-next` · `manual-verify.mdc` |

### 2.8 Google OAuth — 대시보드 프로필·통계 (Planet645)

[`OAUTH-DASHBOARD-PROFILE-STATS-01-manual-verify.md`](../../artifact/ops/verify/OAUTH-DASHBOARD-PROFILE-STATS-01-manual-verify.md) — `local` · `gogo.*.*@gmail.com` · G1–G4 **Pass**. G4: TNB 로그아웃 미동작 → 시크릿 재로그인(후속 `AUTH-LOGOUT-ZERO-01`).

### 2.9 `lotto_*` runbook 문서 정합 (개발 운영)

레거시 `t_win_*` / `fk_win_base_info` 제거 · Flyway `V1__baseline_schema.sql` SoT.  
`artifact/runbook/setup/local-db-setup.md`, `runbook/backend/05-*`, `prd/project-feature-summary.md`, `as-built/recommendation-pipeline.md`.

### 2.10 prod-profile validate 드리프트 정리 (Planet645)

| 항목 | 내용 |
|------|------|
| DB | `lottery` DROP/CREATE → Flyway V1+V2 fresh |
| 기동 | `docker-compose.override.yml` — `SPRING_PROFILES_ACTIVE=prod` |
| validate | `terms_policy_state.id` TINYINT↔Integer → `TermsPolicyState` `Byte` + columnDefinition |
| 예방 | `UserProfile` VARCHAR(36) PK · DATETIME timestamps |
| 검증 | prod Docker 기동 green · dev-login 404 · `lotto_draw` 1217행 · `./gradlew test` green |
| SoT | [`PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01-discover.md`](../../artifact/ops/verify/PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01-discover.md) |

`PHASE17-LOCAL-PROD-REGRESSION-01` Step 1·2 unblock — Step 3–6(스모크·Playwright·수동 UI) 잔여.

### 2.11 로컬 prod-profile 회귀 — 자동 Step (Planet645)

| Step | 결과 |
|------|------|
| 3 API 스모크 | **37/37** (`smoke_runner.py` → `:18080` prod) |
| 4 Playwright | **18 passed / 9 skipped** — stale `auth.json` 제거 후; dev-login 404 skip 정상 |
| 6 증거 | `smoke/artifacts/prod-regression-2026-05-19/` |

SoT: [`PHASE17-LOCAL-PROD-REGRESSION-01-manual-verify.md`](../../artifact/ops/verify/PHASE17-LOCAL-PROD-REGRESSION-01-manual-verify.md). **Step 5** prod-profile 수동 OAuth는 아래 2.12 후 **prod 재실행** 잔여.

### 2.12 PHASE17 Step 5 — 수동 OAuth UI (`local` :8080)

Docker `:18080` 중지 후 IDE/main **8080** 기동. `GET /dev/login?as=user` → **200** → **`local` 프로파일**(prod 아님). UI 스모크 참고용.

### 2.13 PHASE17 Step 5 — **prod** `:8080` (2026-05-19)

`SPRING_PROFILES_ACTIVE=prod` + MariaDB Docker `localhost:3306`. `dev-login` **404**.

| # | 판정 | 메모 |
|---|------|------|
| M1 | Pass (OAuth) | Google 로그인 OK; 콜백 `/onboarding` |
| M1-4 | **결함** | 온보딩 스킵·완료 후에도 재로그인 시 `/onboarding` 반복 — 스킵 경로가 `POST /onboarding` 없이 `/dashboard` 이동 |
| M2·M3 | Pass | 드로어·약관 내역 |
| M4 | **결함** | 알림 UI OK; 저장 POST → `/terms-agreement` (`TermsConsentGuardInterceptor`) |

### 2.14 온보딩 스킵 · 선택 약관 화면 (Planet645)

| 영역 | 내용 |
|------|------|
| **온보딩** | 스킵 confirm 후 `POST /onboarding`(빈 폼) → `markOnboardingCompleted` — 특성은 마이페이지에서 추후 입력 |
| **선택 약관** | 시행 중 선택 약관 전체를 체크박스 한 목록(동의=on) · **저장** → `syncOptionalTermConsents` |
| **마케팅 채널** | 선택 약관 저장 시 **수신 수단 prefs 변경 없음** — `marketing-channels` 별도 |
| **UI** | 「철회할 약관」 분리 폼·안내 제거 |

단위: `TermsConsentServiceTest`, `ProfileEditControllerTest` green.

### 2.15 PHASE17 Step 5 prod 재검증 — **M1–M5 Pass** (Planet645)

`SPRING_PROFILES_ACTIVE=prod` · `dev-login` 404. M1 OAuth·스킵·재로그인, M2 드로어, M3 약관, M4 알림 저장, M5 선택 약관 sync — **전부 Pass**.  
태스크 `PHASE17-LOCAL-PROD-REGRESSION-01` Step 0–6 완료 → 보드 아카이브.

---

## 3. 문제와 해결

### Planet645

#### 3.1 온보딩 스킵 시 `onboarding_completed` 미저장

| | |
|--|--|
| **문제** | prod OAuth 후 `/onboarding` 반복 |
| **원인** | `features.html` — 입력 0건 스킵 시 `window.location.href='/dashboard'`만 수행, `markOnboardingCompleted` 미호출 |
| **해결** | §2.14 — `features.html` 스킵 시 `POST /onboarding` |

#### 3.2 알림 설정 저장 → `/terms-agreement`

| | |
|--|--|
| **문제** | M4 저장 실패, 302 후 `/terms-agreement` |
| **원인** | `hasBlockingExplicitGap` — `TermsConsentGuardInterceptor`가 `/mypage/**` POST 차단 |
| **해결** | `/terms-agreement` 재동의 후 M4 Pass (§2.15) |

### 개발 운영

| 문제 | 원인 | 해결 |
|------|------|------|
| 05-18 일지 다음 액션에 완료가 `[ ]`로 남음 | 익일 작업 후 일지 미갱신 | 05-18·05-19 일지 `[x]` 정합 |

---

## 4. 배운 점

- 선택 약관 **철회 SoT**는 `optional-terms`(PRD §4.2·§4.5) — 동의 내역 조회 화면만으로는 부족.
- 마케팅 철회는 **약관 행 + 채널 prefs**를 한 흐름에서 같이 끄지 않으면 「동의 AND 채널」 가드와 어긋난다.
- 개발일지에는 **git push·PR·merge**를 쓰지 않고, 기능·verify·보드 SoT에만 남긴다.

---

## 5. 다음 액션

> 잔여 출처: [05-18 §5](jungmockdan.github.com/_posts/2026-05-18-developer-log#5-다음-액션) · 보드 `artifact/ops/tasks/board.md`.

### 5.1 미완

| Task ID | 항목 |
|---------|------|
| `TERMS-MINOR-AGE-DECLINE-01` | 수동 verify 잔여 (`review_pending`) |
| `AUTH-LOGOUT-ZERO-01` | design → plan → implement (드로어 로그아웃) |

### 5.2 완료 (당일 — 상세는 §2)

| §2 | 항목 |
|----|------|
| 2.1 | `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` |
| 2.2 | 보드 아카이브 · 05-18 일지 정합 |
| 2.3 | PHASE17 보드 복원 · executionOrder |
| 2.4 | 약관 PRD·design 0a |
| 2.5 | `MYPAGE-MYINFO-NAV-01` 결정 메모 |
| 2.6 | Playwright `mypage-drawer.spec.ts` |
| 2.7 | 수동 verify 0b · verify 워크플로 |
| 2.8 | OAuth 대시보드 verify |
| 2.9 | `lotto_*` runbook 정합 |
| 2.10 | `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` |
| 2.11 | prod-profile 회귀 Step 3–4·6 |
| 2.12 | PHASE17 Step 5 M1–M4 UI (`local` :8080) |
| 2.13 | PHASE17 Step 5 prod — M2·M3 Pass, M1-4·M4 결함 |
| 2.14 | 온보딩 스킵 · optional-terms sync |
| 2.15 | PHASE17 Step 5 prod M1–M5 Pass · 태스크 아카이브 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` |
| 회귀 | **로컬 prod-profile** (스테이징 없음) |
| 보드 SoT | `artifact/ops/tasks/board.md` — `PHASE17-LOCAL-PROD-REGRESSION-01` 아카이브 완료 |
