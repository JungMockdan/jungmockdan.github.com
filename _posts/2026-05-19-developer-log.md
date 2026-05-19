---
title: "개발일지 — 2026-05-19 (선택 약관 철회·보드 정리)"
excerpt: "TERMS-OPTIONAL-CONSENT-WITHDRAW-01: optional-terms 동의·철회, TERM-004 철회 시 마케팅 채널 OFF. Spring PR #11·워크스페이스 보드 아카이브 #16 merge."
categories: [deVlog]
tags: [planet645, spring, terms, mypage, marketing, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-19 10:00:00 +0900
last_modified_at: 2026-05-20 12:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 오늘 목표](#1-오늘-목표)
  - [Planet645](#planet645)
  - [개발 운영](#개발-운영)
- [2. 주요 변경 사항](#2-주요-변경-사항)
  - [2.1 `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` (Planet645)](#21-terms-optional-consent-withdraw-01-planet645)
  - [2.2 개발 운영](#22-개발-운영)
  - [2.3 저녁 — 보드 PHASE17 복원 · `executionOrder` 재정렬 (개발 운영)](#23-저녁--보드-phase17-복원--executionorder-재정렬-개발-운영)
  - [2.4 `MYPAGE-MYINFO-NAV-01` 착수 전 결정 메모 (Planet645 · 문서만)](#24-mypage-myinfo-nav-01-착수-전-결정-메모-planet645--문서만)
  - [2.5 Playwright — 설정 허브 카피 정합 (Planet645)](#25-playwright--설정-허브-카피-정합-planet645)
  - [2.6 수동 verify 0b · 워크플로 정리 (개발 운영)](#26-수동-verify-0b--워크플로-정리-개발-운영)
- [3. 문제와 해결](#3-문제와-해결)
  - [Planet645](#planet645-1)
  - [개발 운영](#개발-운영-1)
- [4. 배운 점](#4-배운-점)
- [5. 다음 액션](#5-다음-액션)
  - [5.1 미완 — 보드 외·문서·운영](#51-미완--보드-외문서운영)
  - [5.2 완료 (당일·이관)](#52-완료-당일이관)
- [6. 기준](#6-기준)

<!-- /code_chunk_output -->


> **하루 요약:** `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` 구현·PR merge. 저녁에는 `board.md`에 PHASE17 prod validate·회귀 태스크 복원 및 `executionOrder` 밴드 재정렬, `MYPAGE-MYINFO-NAV-01` 착수 전 IA 결정 메모 정리. **후속:** 0a 약관 PRD·design §5.4 역정합(마케팅 SoT·재동의 UI) 반영.

## 1. 오늘 목표

### Planet645

- [x] `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` — optional-terms 동의·철회 UX + 서비스
- [x] 단위 테스트 (`TermsConsentServiceTest`, `ProfileEditControllerTest`)
- [x] Spring PR [#11](https://github.com/JungMockdan/com.mockdan.life-saver-lotto/pull/11) merge

### 개발 운영

- [x] 보드 Active에서 태스크 아카이브 · [2026-05-18 일지](/2026-05-18-developer-log) 다음 액션 정합
- [x] 워크스페이스 PR [#16](https://github.com/JungMockdan/com.mockdan.life-saver-lotto-workspace/pull/16) merge
- [x] GitHub Pages push (05-18·05-19 일지) — `gh-pages`←`main` · Actions `pages.yml`

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

### 2.2 개발 운영

| 항목 | 내용 |
|------|------|
| 보드 | `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` → `archive/archive-history-2026-05-24.md`, Active `(none)` |
| 일지 | [2026-05-18](/2026-05-18-developer-log) §5 다음 액션 — 완료 항목 `[x]` 반영, §2.10 교차 링크 |

### 2.3 저녁 — 보드 PHASE17 복원 · `executionOrder` 재정렬 (개발 운영)

[05-13 일지](/2026-05-13-developer-log#5-다음-액션)에만 있던 prod validate·회귀 태스크가 `board.md` 이관 시 **누락**되어 있었음. 복원 후 전체 백로그 실행 순서를 밴드별로 다시 맞췄다.

| Task | executionOrder | 비고 |
|------|----------------|------|
| `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` | `105` | high · 다음 착수 |
| `PHASE17-LOCAL-PROD-REGRESSION-01` | `106` | `blockedBy` 105 |
| `TERMS-MINOR-AGE-DECLINE-01` | `110` | 05-18 `230` → verify 대기(`review_pending`) |
| `AUTH-LOGOUT-ZERO-01` | `115` | 05-18 `200` → PRD ✅ · design/plan |
| `MYPAGE-CREDIT-PAGE-01` | `200` | 05-18 `210` |
| `MYPAGE-MYINFO-NAV-01` | `210` | 05-18 `220` · 선행 CREDIT-PAGE |
| `PHASE22-BILLING-SANDBOX-01` | `220` | 05-18 `240` |
| `PHASE22-BILLING-REFUND-01` | `230` | 05-18 `250` |
| `PROFILE-ADDRESS-SEARCH-01` | `240` | 05-18 `260` |
| `IDENTITY-CI-FREE-ABUSE-01` | `250` | 05-18 `270` |

- Metadata: `backlogRunOrder`, `selectionGuide`, executionOrder **밴드 표** 추가
- `artifact/plan/phase17-product-engineering-backlog.md` §C.5 — SoT 경로 `board.md`·선행 drift 명시

### 2.4 `MYPAGE-MYINFO-NAV-01` 착수 전 결정 메모 (Planet645 · 문서만)

구현 전 제품·IA에서 확정할 항목을 정리했다(코드 변경 없음).

1. **허브 URL** — `/mypage/info` 신설 vs `/mypage`(현재 설정 전용) 승격
2. **드로어·하단탭 진입** — 직링크 유지 vs 허브 경유
3. **nav 범위** — 구독·플랜·크레딧(CREDIT-PAGE 라우트 후) 포함 여부
4. **설정과 경계** — 메시지함(`/notifications`) vs 알림 수신 설정(`/mypage/settings/notifications`) 라벨 분리
5. **기존 URL 유지** vs UI 스펙 경로(`/mypage/recommendations` 등) 이전

권장 초안: `/mypage/info` 허브 · 드로어 직링크 유지 · nav에 크레딧은 CREDIT-PAGE 후 추가.

### 2.5 Playwright — 설정 허브 카피 정합 (Planet645)

`smoke/e2e/mypage-drawer.spec.ts`: as-built 제목 「내 정보 > 설정」·약관 화면 「약관 및 동의 내역」·`← 내 정보` back 링크에 맞게 assertion 수정.

### 2.6 수동 verify 0b · 워크플로 정리 (개발 운영)

| 항목 | 내용 |
|------|------|
| **0b** | [`TERMS-OPTIONAL-CONSENT-01-manual-verify.md`](../../artifact/ops/verify/TERMS-OPTIONAL-CONSENT-01-manual-verify.md) — M·C·K·W **Pass** (§6 증거) |
| **함정** | `POST marketing-channels`에 `curl -L` → 302 동일 URL 재-POST로 채널 전부 OFF 오탐 — runbook에 금지 |
| **철회 후** | `is_agreed=0` → marketing-channels **302 `/terms-agreement`** (인페이지 게이트 grep 아님) |
| **워크플로** | `artifact/ops/verify/` · `/verify-scaffold` · `/verify-next` · 규칙 `manual-verify.mdc` — **Agent가 curl/DB 먼저 실행**, 막힐 때만 수동 |

---

## 3. 문제와 해결

### Planet645

_(해당 없음)_

### 개발 운영

| 문제 | 원인 | 해결 |
|------|------|------|
| 05-18 일지 다음 액션에 완료 태스크가 `[ ]`로 남음 | 익일 merge 후 일지 미갱신 | 05-18·05-19 일지 동시 정리; 완료는 `[x]` + PR 링크 |

---

## 4. 배운 점

- 선택 약관 **철회 SoT**는 동의 내역 조회 화면(`/mypage/settings/terms`)이 아니라 **optional-terms**에 두는 것이 PRD §4.2·§4.5와 맞다.
- 마케팅 철회는 **약관 행 + 채널 prefs**를 한 트랜잭션 흐름에서 같이 끄지 않으면 「동의 AND 채널」 가드와 어긋난다.

---

## 5. 다음 액션

> **05-18 잔여 출처:** [05-18 §5](/2026-05-18-developer-log#5-다음-액션) — 번호는 §2.3 재정렬 후 보드 기준(05-18 `executionOrder`와 다름).

### 5.1 미완 — 보드 외·문서·운영

[05-18 §5.3](/2026-05-18-developer-log#53-잔여-업무--verify문서-보드-외) **0a·0b** 매핑 계승. **0c·0d**는 보드 등재 후 §5.1로 이동(`PHASE17-LOCAL-PROD-REGRESSION-01` · `TERMS-MINOR-AGE-DECLINE-01`).

| 05-18 | Task ID | 항목 | 메모 |
|-------|---------|------|------|
| 0a | `TERMS-OPTIONAL-CONSENT-MYPAGE-01` _(archived)_ | [x] PRD `terms-consent.md` §4.5·§6.1·§8#6 + design `01-schema-revision-guards.md` §5.4 | 결정(2026-05-19→20): **① A+C** — 동의·철회 SoT `optional-terms`, 채널 prefs `marketing-channels` / **②** 일괄 채널 OFF 비제공·발송은 동의∧채널∧게이트 AND / **③ D** — 재동의 배너 비채택·`/terms-agreement` 리다이렉트 / **④** 버전+시행일 UI 필수 / **⑤** §4.5·§6.1 에 경로 명시 |
| 0b | `TERMS-OPTIONAL-CONSENT-MYPAGE-01` · `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` _(archived)_ | [x] optional-terms → marketing-channels UI 스모크 | [`TERMS-OPTIONAL-CONSENT-01-manual-verify.md`](../../artifact/ops/verify/TERMS-OPTIONAL-CONSENT-01-manual-verify.md) §6 **0b PASS** |
| — | _(보드 없음)_ | [x] Google OAuth 재로그인 — 대시보드 프로필·통계 | [`OAUTH-DASHBOARD-PROFILE-STATS-01-manual-verify.md`](../../artifact/ops/verify/OAUTH-DASHBOARD-PROFILE-STATS-01-manual-verify.md) §6 **PASS** (`local` · `gogo.*.*@gmail.com`) · G4 시크릿 재로그인(로그아웃 UI 후속) |
| — | _(보드 없음)_ | [x] 스키마 rename PR (Spring + ML) | Spring #8·ML #1 머지 · runbook/PRD `t_win_*` 문서 정리(§5.2) · prod validate(`105`) **별도** |
| — | _(운영·보드 없음)_ | [x] GitHub Pages push (05-18·05-19 일지) | `.github/workflows/pages.yml` · legacy 빌드 `errored` → Actions 배포 |

### 5.2 완료 (당일·이관)

| 구분 | 항목 |
|------|------|
| Planet645 | `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` · Spring PR [#11](https://github.com/JungMockdan/com.mockdan.life-saver-lotto/pull/11) |
| 개발 운영 | 보드 PHASE17 복원·Order 밴드 · PR [#18](https://github.com/JungMockdan/com.mockdan.life-saver-lotto-workspace/pull/18) · phase17 §C.5 · `mypage-drawer.spec.ts` |
| 개발 운영 | [05-18 일지](/2026-05-18-developer-log) 완료 항목 `[x]` 정합 |
| 개발 운영 | 약관 PRD·설계 역정합 — `artifact/prd/terms-consent.md` §1·§4.5·§6.1·§8#6, `artifact/design/terms-consent/01-schema-revision-guards.md` §5.4·§10.1#7 (0a 결정 반영) |
| 개발 운영 | 수동 verify **0b** Pass · `artifact/ops/verify/` runbook/record 패턴 · `/verify-scaffold` · `/verify-next` |
| 개발 운영 | GitHub Pages — `gh-pages`에 `main` merge · Actions `pages.yml` (legacy Jekyll 빌드 실패 대체) |
| Planet645 | Google OAuth → `/dashboard` 프로필·통계 수동 verify Pass — [`OAUTH-DASHBOARD-PROFILE-STATS-01-manual-verify.md`](../../artifact/ops/verify/OAUTH-DASHBOARD-PROFILE-STATS-01-manual-verify.md) |
| 개발 운영 | `lotto_*` rename 문서 정합 — `artifact/runbook/setup/local-db-setup.md`, `runbook/backend/05-*`, `prd/project-feature-summary.md`, `as-built/recommendation-pipeline.md` |
| 아카이브(이전) | `PHASE17-02-FOLLOWUP-MKT-SEND-CHECK`, `NOTIF-MKT-SEND-GUARD-01` 등 — [05-18 §5.1](/2026-05-18-developer-log#51-오늘-완료아카이브-태스크-id) |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` · merge `73ec8a5` (PR #11) |
| 회귀 | 스테이징 없음 → **로컬 prod-profile** |
| 보드 SoT | `artifact/ops/tasks/board.md` — Active 비어 있음 (2026-05-19) |
