---
title: "개발일지 — 2026-05-21 night (세션 쿠키·보안 기준선)"
excerpt: "SECURITY-COOKIE-SESSION-01 — design-delta·profile 쿠키 정렬·단위 검증·verify runbook 스캐폴드. 로컬 prod SameSite=Strict."
categories: [deVlog]
tags: [planet645, spring, security, session, cookie, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-21 22:00:00 +0900
last_modified_at: 2026-05-21 22:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약(night):** Billing 다음 보안 밴드 **`SECURITY-COOKIE-SESSION-01`** — 세션 쿠키·타임아웃을 profile별로 명시하고 design-delta·as-built를 맞춤. 수동 verify runbook·run record 스캐폴드 완료 — `/verify-next`는 앱 기동 후.

## 1. 오늘 목표

### Planet645

- [x] `SECURITY-COOKIE-SESSION-01` — `/task-flow-route` → `delta` 프로필
- [x] OD-S1~S5 확정 · design-delta · `application-*.properties` · `CookieSessionSecurityTest`
- [x] `09-security-privacy.md` §1.4 · `auth-oauth.md` 동기화

### 개발 운영

- [x] `artifact/design/security/cookie-session-design-delta.md`
- [x] verify split: `SECURITY-COOKIE-SESSION-01-runbook.md` + `runs/2026-05-21-…`
- [x] 보드 `flowProfile` · `VerifyDocReference` 연결

---

## 2. 주요 변경 사항

### 2.1 워크플로우 라우팅 (개발 운영)

| 항목 | 판정 |
|------|------|
| `flowProfile` | **`delta`** — CSRF 직후 세션 축 정렬, full PRD·spike 생략 |
| 게이트 | 세션·쿠키 정책 변경 → implement 전 OD 확정 |

### 2.2 설계 결정 OD-S1~S5 (Planet645)

| ID | 결정 |
|----|------|
| **OD-S1** | prod `SameSite=Strict` |
| **OD-S2** | `local` `Secure=false` · `prod` `Secure=true` |
| **OD-S3** | `server.servlet.session.timeout=30m` |
| **OD-S4** | `maximumSessions(1)` · `maxSessionsPreventsLogin(false)` — 신규 로그인 시 이전 세션 만료 |
| **OD-S5** | 저장소 문서 → **in-memory `HttpSession`** (`SPRING_SESSION` 표기 정정) |

SoT: 워크스페이스 `artifact/design/security/cookie-session-design-delta.md`.

### 2.3 구현 — profile 쿠키 매트릭스 (Planet645)

| 파일 | 내용 |
|------|------|
| `application.properties` | `timeout=30m` · `cookie.http-only=true` |
| `application-local.properties` | `secure=false` · `same-site=lax` |
| `application-prod.properties` | `secure=true` · `same-site=strict` |
| `SecurityConfig` | 변경 없음 — `migrateSession` · `maximumSessions(1)` as-built 유지 |

**자동 검증:** `CookieSessionSecurityTest` — `ServerProperties`로 local/prod 플래그·공통 30m (`./gradlew test` green).

### 2.4 문서 delta (Planet645 · 개발 운영)

| 문서 | 반영 |
|------|------|
| `09-security-privacy.md` §1.4 | in-memory 세션 · profile별 쿠키 · concurrent session |
| `artifact/as-built/auth-oauth.md` | 세션 쿠키·`SecurityConfig` 요약 |
| `cookie-session-design-delta.md` | OD 확정 · 검증 표 T-SESS-0~4 |

### 2.5 verify 스캐폴드 (개발 운영)

| Verify ID | Runbook | 시나리오 |
|-----------|---------|----------|
| `SECURITY-COOKIE-SESSION-01` | `artifact/ops/verify/SECURITY-COOKIE-SESSION-01-runbook.md` | S01–S06 (`BASE=18080`) |

Run: `artifact/ops/verify/runs/2026-05-21-SECURITY-COOKIE-SESSION-01.md` — 전항 `pending`. S05 동시 세션·S06 prod `dev-login` 404는 브라우저/prod 기동 시 실행 또는 `skip`.

---

## 3. 문제와 해결

### Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
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

### Planet645

- [ ] `/verify-next` — S01–S04 (`docker compose up` 후)
- [ ] S05 브라우저 동시 세션(선택) · S06 로컬 prod-profile `dev-login` 404
- [ ] verify Pass 후 보드 `SECURITY-COOKIE-SESSION-01` 아카이브

### 개발 운영

- [ ] verify run `2026-05-21-…` 최종 `PASS` 기록
- [ ] 후속 `SECURITY-OAUTH2-HARDENING-01` (redirect·state·nonce)

---

## 6. 기준

- 환경 표기: **로컬 prod-profile 회귀** (staging 용어 없음).
- 일지에 Git push·PR·merge 기록 없음 — SoT는 보드·verify·`artifact/design/security/`.
