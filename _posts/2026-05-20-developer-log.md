---
title: "개발일지 — 2026-05-20 (로그아웃·마이페이지 크레딧·내정보 내비)"
excerpt: "로그아웃 v1·크레딧 SSR·내 정보 허브/서브nav. CREDIT·MYINFO verify runbook 스캐폴드 — `/verify-next`는 회사 일괄 예정."
categories: [deVlog]
tags: [planet645, spring, identity, security, logout, mypage, credits, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-20 16:00:00 +0900
last_modified_at: 2026-05-21 00:30:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** `AUTH-LOGOUT-ZERO-01` 완료 후 마이페이지 **크레딧 SSR**·**내 정보 허브/서브nav** 구현, CREDIT·MYINFO **verify runbook** 스캐폴드. 시나리오 실행은 회사 일괄 세션으로 보류.

## 1. 오늘 목표

### Planet645

- [x] `AUTH-LOGOUT-ZERO-01` — design · implement · 단위·MockMvc
- [x] `MYPAGE-CREDIT-PAGE-01` — design · implement
- [x] `MYPAGE-MYINFO-NAV-01` — design · implement

### 개발 운영

- [x] 보드 아카이브(`AUTH-LOGOUT-ZERO-01`) · Active 정리 · `SECURITY-GLOBAL-CSRF-01` blockedBy 해제
- [x] CREDIT·MYINFO verify runbook · run record 스캐폴드

---

## 2. 주요 변경 사항

### 2.1 `AUTH-LOGOUT-ZERO-01` — 로그아웃 제로베이스 (Planet645)

as-built는 GET `/logout`·고정 `logoutSuccessUrl("/")`·감사·API 미구현. PRD Q1–Q8·design(OD-1·OD-2)에 맞춰 v1 구현.

| 영역 | v1 동작 |
|------|---------|
| **SSR** | 드로어·`reconsent` — `POST /logout` + CSRF hidden |
| **CSRF** | `requireCsrfProtectionMatcher` — **POST `/logout` only** |
| **세션** | 현재 브라우저 1건 무효화 |
| **리다이렉트** | same-origin Referer allowlist path 복귀, else `/` |
| **API** | `POST /api/v1/auth/logout` → 204 |
| **감사** | `IdentityAuditLog` `LOGOUT` — 감사 실패해도 로그아웃 성공 |

**코드 SoT:** `LogoutService`, `IdentityLogoutHandler`, `PostLogoutRedirectService`, `AuthLogoutController`, `SecurityConfig`, `fragments/tnb.html`, `terms/reconsent.html`.

**검증:** `AuthLogoutSecurityTest`, `LogoutServiceTest`, `PostLogoutRedirectServiceTest`.

**문서:** `artifact/design/user-identity/auth-logout-flow.md`, `artifact/plan/auth-logout-implementation.md`.

### 2.2 보드 · 백로그 (개발 운영)

| 항목 | 내용 |
|------|------|
| 아카이브 | `AUTH-LOGOUT-ZERO-01` → `archive/archive-history-2026-05-24.md` |
| Active | `MYPAGE-CREDIT-PAGE-01`(200), `MYPAGE-MYINFO-NAV-01`(210) — verify in_progress |
| 다음 | `SECURITY-GLOBAL-CSRF-01`(116) — `blockedBy` 해제 |

### 2.3 `MYPAGE-CREDIT-PAGE-01` — 크레딧 SSR·이력 (Planet645)

| 항목 | v1 |
|------|-----|
| **라우트** | `GET /mypage/credits` |
| **잔액** | `EntitlementService.getBalance` — TNB·SSR 단일 소스 |
| **이력** | `credit_ledger` · KST·만료일 컬럼 |
| **API** | `GET /api/v1/credits/history` |
| **드로어** | `drawer_nav_credits` → `/mypage/credits` |

**코드 SoT:** `MypageController.credits`, `credits.html`, `EntitlementServiceImpl.getLedgerHistoryPage`, `CreditController`.

**검증:** `EntitlementServiceTest`, `ProfileEditControllerTest.creditsPage`.

**문서:** `artifact/plan/mypage-credit-page-01-design.md` · PRD `mypage-drawer.md` §3.2·US-5.

### 2.4 `MYPAGE-MYINFO-NAV-01` — 내 정보 허브·서브nav (Planet645)

| 항목 | v1 |
|------|-----|
| **허브** | `GET /mypage/info` |
| **Fragment** | `mypage-myinfo-menu`, `mypage-myinfo-subnav` · `activeKey` |
| **서브** | profile · inbox · history · subscribe · credits |
| **드로어** | `drawer_nav_myinfo_hub` · 직링크 유지 |
| **설정** | back `← 설정` → `/mypage` |

**검증:** `ProfileEditControllerTest` (`infoHub`, `myinfoActiveKey`).

**문서:** `artifact/plan/mypage-myinfo-nav-01-design.md` · PRD `mypage-drawer.md` §3.1·§3.2.

### 2.5 PRD 동기화 — `mypage-drawer.md` (Planet645)

| 태스크 | PRD 반영 |
|--------|----------|
| `MYPAGE-CREDIT-PAGE-01` | §1 비목표 정리 · §3.2 `/mypage/credits` · **US-5** `EntitlementService.getBalance` + 드로어→상세 |
| `MYPAGE-MYINFO-NAV-01` | §3.1 `GET /mypage/info` 노드 · §3.2 **내 정보 허브** 행 |

보드 design 체크리스트 PRD delta·라우트 한 줄 **완료**.

### 2.6 마이페이지 verify 스캐폴드 (개발 운영)

| Verify ID | Runbook | 필수 시나리오 |
|-----------|---------|----------------|
| `MYPAGE-CREDIT-PAGE-01` | `artifact/ops/verify/MYPAGE-CREDIT-PAGE-01-runbook.md` | C01–C07, B01 (C08–C10 조건부 skip) |
| `MYPAGE-MYINFO-NAV-01` | `artifact/ops/verify/MYPAGE-MYINFO-NAV-01-runbook.md` | N01–N09, B01 |

Run: `runs/2026-05-20-MYPAGE-*-01.md` · `BASE=http://localhost:18080` · 전항 `pending`. `/verify-next` 실행은 회사 일괄(CREDIT+MYINFO).

---

## 3. 문제와 해결

### Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| GET `/logout` 북마크 | matcher POST only | GET 404 — design 명시 |
| `credits/history` 계약만 존재 | API 미구현 | `credit_ledger` + SSR 동일 서비스 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 로그아웃 CSRF는 **한 엔드포인트 matcher**로 충분 — 마이페이지 POST 전역은 `SECURITY-GLOBAL-CSRF-01`로 분리.
- `ux-gap` 마이페이지는 design-delta + runbook으로 implement·verify를 끊을 수 있다.
- verify **scaffold**와 **실행**을 분리하면 집/회사 환경을 나누기 쉽다.

---

## 5. 다음 액션

> SoT: `artifact/ops/tasks/board.md`.

### 5.1 미완

#### Planet645

- [ ] `MYPAGE-CREDIT-PAGE-01` · `MYPAGE-MYINFO-NAV-01` — `/verify-next` 일괄 · Pass 후 아카이브

#### 개발 운영

- [ ] `SECURITY-GLOBAL-CSRF-01` — design → implement

### 5.2 완료 (당일 — 상세 §2)

| §2 | 항목 |
|----|------|
| 2.1 | `AUTH-LOGOUT-ZERO-01` |
| 2.2 | 보드 아카이브 · Active |
| 2.3 | `MYPAGE-CREDIT-PAGE-01` |
| 2.4 | `MYPAGE-MYINFO-NAV-01` |
| 2.5 | PRD `mypage-drawer.md` 동기화 |
| 2.6 | verify runbook 스캐폴드 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` |
| 보드·verify | `artifact/ops/tasks/board.md`, `artifact/ops/verify/` |
