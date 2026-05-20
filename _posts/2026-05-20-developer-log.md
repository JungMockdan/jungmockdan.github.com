---
title: "개발일지 — 2026-05-20 (로그아웃 제로베이스)"
excerpt: "AUTH-LOGOUT-ZERO-01 — POST+CSRF·API logout·LOGOUT 감사·Referer redirect. 보드 아카이브·SECURITY-GLOBAL-CSRF-01 선행 해제."
categories: [deVlog]
tags: [planet645, spring, identity, security, logout, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-20 16:00:00 +0900
last_modified_at: 2026-05-20 16:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 로그아웃을 PRD·design·plan·Spring 구현까지 한 사이클로 맞춤 — 드로어 POST+CSRF, `POST /api/v1/auth/logout`, 현재 세션만 무효화, `IdentityAuditLog.LOGOUT`, 공개 Referer 복귀.

## 1. 오늘 목표

### Planet645

- [x] `AUTH-LOGOUT-ZERO-01` — design · plan · implement · 단위·MockMvc 검증

### 개발 운영

- [x] 보드 Active 정리 · 아카이브 · `SECURITY-GLOBAL-CSRF-01` blockedBy 해제

---

## 2. 주요 변경 사항

### 2.1 `AUTH-LOGOUT-ZERO-01` — 로그아웃 제로베이스 (Planet645)

as-built는 GET `/logout` 링크·고정 `logoutSuccessUrl("/")`·감사 없음·API 미구현이었다. PRD Q1–Q8(2026-05-18)과 design(OD-1·OD-2 확정)에 맞춰 v1을 구현했다.

| 영역 | v1 동작 |
|------|---------|
| **SSR** | 드로어·`reconsent` — `POST /logout` + CSRF hidden |
| **CSRF** | `requireCsrfProtectionMatcher` — **POST `/logout` only** (전역 CSRF 아님) |
| **세션** | 현재 브라우저 1건 무효화 (`09-security-privacy` 「전 세션」과 의도적 delta) |
| **리다이렉트** | same-origin Referer path가 공개 allowlist면 복귀, else `/` |
| **API** | `POST /api/v1/auth/logout` → 204 (CSRF 면제) |
| **감사** | `IdentityAuditLog` `LOGOUT` — 저장 실패해도 로그아웃 성공 |

**코드 SoT:** `LogoutService`, `IdentityLogoutHandler`, `PostLogoutRedirectService`, `AuthLogoutController`, `SecurityConfig`, `fragments/tnb.html`, `terms/reconsent.html`.

**검증:** `AuthLogoutSecurityTest`, `LogoutServiceTest`, `PostLogoutRedirectServiceTest` · `./gradlew test` green.

**문서:** `artifact/design/user-identity/auth-logout-flow.md`, `artifact/plan/auth-logout-implementation.md`.

### 2.2 보드 · 백로그 현행화 (개발 운영)

| 항목 | 내용 |
|------|------|
| 아카이브 | `AUTH-LOGOUT-ZERO-01` → `archive/archive-history-2026-05-24.md`, index 39건 |
| Active | _(없음)_ |
| 다음 밴드 | `SECURITY-GLOBAL-CSRF-01` (`116`) — SSR POST 전역 CSRF 확장, 선행 태스크 완료로 `blockedBy` 해제 |
| `backlogRunOrder` | `116` → `200` → … |

---

## 3. 문제와 해결

### Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| GET `/logout` 북마크 | logout matcher POST only | GET은 logout 필터 미적용(404) — design에 명시 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 로그아웃 CSRF는 **matcher로 한 엔드포인트만** 켜도 PRD Q3-A를 만족한다 — 마이페이지 POST 일괄 CSRF는 `SECURITY-GLOBAL-CSRF-01`로 분리하는 편이 안전하다.
- Referer-only redirect는 Safari 등 Referer 누락 시 `/` 폴백이 기본 UX다.

---

## 5. 다음 액션

> 보드 SoT: `artifact/ops/tasks/board.md`.

### 5.1 미완

| Task ID | 항목 |
|---------|------|
| `SECURITY-GLOBAL-CSRF-01` | SSR POST 전역 CSRF — design → implement → governance |
| `MYPAGE-CREDIT-PAGE-01` | 크레딧 전용 페이지 (200) |
| `MYPAGE-MYINFO-NAV-01` | 내 정보 허브·서브nav (210) |

### 5.2 완료 (당일 — 상세는 §2)

| §2 | 항목 |
|----|------|
| 2.1 | `AUTH-LOGOUT-ZERO-01` |
| 2.2 | 보드 아카이브 · CSRF 후속 태스크 정리 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` |
| PRD·design | `artifact/prd/auth-logout.md`, `artifact/design/user-identity/auth-logout-flow.md` |
| 보드 | Active 없음 · 다음 `116` `SECURITY-GLOBAL-CSRF-01` |
