---
title: "개발일지 — 2026-05-21 (마이페이지 verify · 세션 쿠키 기준선)"
excerpt: "MYPAGE 2건 manual verify PASS·보드 아카이브. SECURITY-COOKIE-SESSION-01 — OD·profile 쿠키·단위 검증·verify runbook 스캐폴드(전항 pending)."
categories: [deVlog]
tags: [planet645, spring, mypage, credits, security, session, cookie, verify, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-21 09:00:00 +0900
last_modified_at: 2026-05-21 22:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** [2026-05-20](2026-05-20-developer-log.md)에서 implement·runbook까지 끝낸 **마이페이지 크레딧·내 정보 내비** 2건을 manual verify로 닫고 보드 아카이브. 이어 Billing 다음 보안 밴드 **`SECURITY-COOKIE-SESSION-01`** — 세션 쿠키·타임아웃 profile 정렬·design-delta·verify 스캐폴드(`/verify-next`는 앱 기동 후).

## 1. 오늘 목표

### Planet645

- [x] `MYPAGE-CREDIT-PAGE-01` · `MYPAGE-MYINFO-NAV-01` — manual verify **PASS**
- [x] `SECURITY-COOKIE-SESSION-01` — `/task-flow-route` → `delta` · OD-S1~S5 · profile 쿠키 · `CookieSessionSecurityTest`
- [x] `09-security-privacy.md` §1.4 · `auth-oauth.md` 동기화

### 개발 운영

- [x] verify run record — `runs/2026-05-20-MYPAGE-*-01.md` 최종 PASS
- [x] `MYPAGE-*` 보드 → `archive-history-index.md` + `archive/archive-history-2026-05-24.md`
- [x] `cookie-session-design-delta.md` · `SECURITY-COOKIE-SESSION-01-runbook.md` + `runs/2026-05-21-…` · 보드 `flowProfile` 연결

---

## 2. 주요 변경 사항

### 2.1 마이페이지 manual verify PASS (Planet645)

| 태스크 | Run record | 판정 |
|--------|------------|------|
| `MYPAGE-CREDIT-PAGE-01` | `runs/2026-05-20-MYPAGE-CREDIT-PAGE-01.md` | **PASS** (C01–C08 PASS, C09/C10/B01 skip) |
| `MYPAGE-MYINFO-NAV-01` | `runs/2026-05-20-MYPAGE-MYINFO-NAV-01.md` | **PASS** (N01–N09 PASS, B01 skip) |

환경: `local` · dev-login · `BASE=http://localhost:18080` (runbook과 동일).

| 태스크 | implement SoT (05-20) | verify SoT |
|--------|----------------------|------------|
| `MYPAGE-CREDIT-PAGE-01` | `GET /mypage/credits`, `GET /api/v1/credits/history`, 드로어 → `/mypage/credits` | `artifact/ops/verify/MYPAGE-CREDIT-PAGE-01-runbook.md` |
| `MYPAGE-MYINFO-NAV-01` | `GET /mypage/info`, `mypage-myinfo-*` fragment, 서브 `myinfoActiveKey` | `artifact/ops/verify/MYPAGE-MYINFO-NAV-01-runbook.md` |

워크스페이스·Docker 이미지 동기화 후 `docker compose` rebuild·재기동. 1차 verify는 미동기화로 FAIL → 재실행에서 통과.

### 2.2 마이페이지 보드 아카이브 (개발 운영)

| 태스크 | completedAt (UTC) | 주간 파일 |
|--------|-------------------|-----------|
| `MYPAGE-CREDIT-PAGE-01` | `2026-05-21T02:45:00Z` | `archive/archive-history-2026-05-24.md` |
| `MYPAGE-MYINFO-NAV-01` | `2026-05-21T02:46:00Z` | 동일 |

Active 비움 · 다음 백로그 선두: `PHASE22-BILLING-SANDBOX-01`(220).

### 2.3 SECURITY 워크플로우 (개발 운영)

| 항목 | 판정 |
|------|------|
| `flowProfile` | **`delta`** — CSRF 직후 세션 축 정렬, full PRD·spike 생략 |
| 게이트 | 세션·쿠키 정책 변경 → implement 전 OD 확정 |

### 2.4 SECURITY 설계 OD-S1~S5 (Planet645)

| ID | 결정 |
|----|------|
| **OD-S1** | prod `SameSite=Strict` |
| **OD-S2** | `local` `Secure=false` · `prod` `Secure=true` |
| **OD-S3** | `server.servlet.session.timeout=30m` |
| **OD-S4** | `maximumSessions(1)` · `maxSessionsPreventsLogin(false)` — 신규 로그인 시 이전 세션 만료 |
| **OD-S5** | 저장소 문서 → **in-memory `HttpSession`** (`SPRING_SESSION` 표기 정정) |

SoT: `artifact/design/security/cookie-session-design-delta.md`.

### 2.5 SECURITY profile 쿠키 구현 (Planet645)

| 파일 | 내용 |
|------|------|
| `application.properties` | `timeout=30m` · `cookie.http-only=true` |
| `application-local.properties` | `secure=false` · `same-site=lax` |
| `application-prod.properties` | `secure=true` · `same-site=strict` |
| `SecurityConfig` | 변경 없음 — `migrateSession` · `maximumSessions(1)` as-built 유지 |

**자동 검증:** `CookieSessionSecurityTest` — `ServerProperties`로 local/prod 플래그·공통 30m (`./gradlew test` green).

### 2.6 SECURITY 문서 delta (Planet645)

| 문서 | 반영 |
|------|------|
| `09-security-privacy.md` §1.4 | in-memory 세션 · profile별 쿠키 · concurrent session |
| `artifact/as-built/auth-oauth.md` | 세션 쿠키·`SecurityConfig` 요약 |
| `cookie-session-design-delta.md` | OD 확정 · 검증 표 T-SESS-0~4 |

### 2.7 SECURITY verify 스캐폴드 (개발 운영)

| Verify ID | Runbook | 시나리오 |
|-----------|---------|----------|
| `SECURITY-COOKIE-SESSION-01` | `artifact/ops/verify/SECURITY-COOKIE-SESSION-01-runbook.md` | S01–S06 (`BASE=18080`) |

Run: `artifact/ops/verify/runs/2026-05-21-SECURITY-COOKIE-SESSION-01.md` — 전항 `pending`. S05 동시 세션·S06 prod `dev-login` 404는 브라우저/prod 기동 시 실행 또는 `skip`.

---

## 3. 문제와 해결

### Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| MYPAGE 1차 verify FAIL | 워크스페이스·Docker 이미지 미동기화 | 동기화·rebuild 후 CREDIT·MYINFO **PASS** |
| §1.4 `SPRING_SESSION` table | 설계·as-built 불일치 | OD-S5: in-memory `HttpSession`으로 정정 |
| prod `SameSite` 미명시 | properties 없음 | prod `strict` · local `lax` 분리 |
| MockMvc `Set-Cookie` 검증 실패 | dev-login·lazy session | `ServerProperties` profile 테스트로 대체 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- CSRF(`Strict`+토큰) 다음 축은 **쿠키 attribute + session fixation/concurrency** — profile 파일로 prod/local을 분리하면 로컬 HTTP dev-login과 prod HTTPS 기준선을 한 태스크에서 닫을 수 있다.
- concurrent session은 **OAuth 로그인 이벤트**가 있어야 재현된다 — runbook S05를 manual·skip 허용으로 두는 편이 맞다.

---

## 5. 다음 액션

> SoT: `artifact/ops/tasks/board.md`.

### 5.1 미완

#### Planet645

- [ ] `/verify-next` — `SECURITY-COOKIE-SESSION-01` S01–S04 (`docker compose up` 후)
- [ ] S05 브라우저 동시 세션(선택) · S06 로컬 prod-profile `dev-login` 404
- [ ] verify Pass 후 `SECURITY-COOKIE-SESSION-01` 보드 아카이브

#### 개발 운영

- [ ] verify run `2026-05-21-SECURITY-COOKIE-SESSION-01.md` 최종 **PASS** 기록
- [ ] 후속 `SECURITY-OAUTH2-HARDENING-01` (redirect·state·nonce)
- [ ] 다음 백로그: `PHASE22-BILLING-SANDBOX-01`(220) 또는 보안 밴드(235+)

### 5.2 완료 (당일 — [§2](#2-주요-변경-사항))

| §2 | 항목 |
|----|------|
| [2.1](#21-마이페이지-manual-verify-pass-planet645) | MYPAGE CREDIT·MYINFO verify PASS |
| [2.2](#22-마이페이지-보드-아카이브-개발-운영) | MYPAGE 보드 아카이브 2건 |
| [2.3](#23-security-워크플로우-개발-운영) | SECURITY `delta` 워크플로우 |
| [2.4](#24-security-설계-od-s1s5-planet645) | OD-S1~S5 |
| [2.5](#25-security-profile-쿠키-구현-planet645) | profile 쿠키·`CookieSessionSecurityTest` |
| [2.6](#26-security-문서-delta-planet645) | security 문서 delta |
| [2.7](#27-security-verify-스캐폴드-개발-운영) | SECURITY verify 스캐폴드 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` |
| 보드·verify | `artifact/ops/tasks/board.md`, `artifact/ops/verify/` |
| 설계 | `artifact/design/security/cookie-session-design-delta.md` |
| 환경 표기 | **로컬 prod-profile 회귀** (staging 용어 없음) |
| 전일 일지 | [2026-05-20-developer-log.md](2026-05-20-developer-log.md) |
