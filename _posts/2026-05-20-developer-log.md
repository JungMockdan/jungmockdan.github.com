---
title: "개발일지 — 2026-05-20 (PHASE17 Step 5 재검증·약관 verify)"
excerpt: "온보딩·선택 약관 수정 반영 후 prod Step 5 재검증, 미성년 decline 수동 verify, 로그아웃 design 착수."
categories: [deVlog]
tags: [planet645, spring, terms, oauth, prod-profile, verify, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-20 10:00:00 +0900
last_modified_at: 2026-05-20 10:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 오늘 목표](#1-오늘-목표)
  - [Planet645](#planet645)
  - [개발 운영](#개발-운영)
- [2. 주요 변경 사항](#2-주요-변경-사항)
  - [Planet645](#planet645-1)
  - [개발 운영](#개발-운영-1)
- [3. 문제와 해결](#3-문제와-해결)
  - [Planet645](#planet645-2)
  - [개발 운영](#개발-운영-2)
- [4. 배운 점](#4-배운-점)
- [5. 다음 액션](#5-다음-액션)
  - [5.1 미완](#51-미완)
  - [5.2 완료 (당일 — 상세는 §2)](#52-완료-당일--상세는-2)
- [6. 기준](#6-기준)

<!-- /code_chunk_output -->


> **하루 요약:** `TERMS-MINOR-AGE-DECLINE-01` 수동 verify 완료·보드 아카이브. D6(가입 전 decline) 경로에서 social·감사 없음이 as-built와 일치함을 확인.

## 1. 오늘 목표

### Planet645

- [ ] `PHASE17-LOCAL-PROD-REGRESSION-01` — Step 5 **prod** `:8080` 수동 OAuth UI 재검증 (M1-4 온보딩 스킵·M4 알림/약관 가드 — [05-19 §2.13–2.14](/2026-05-19-developer-log#214-온보딩-스킵--선택-약관-화면-planet645) 수정 반영 후)
- [x] `TERMS-MINOR-AGE-DECLINE-01` — 수동 verify **done** ([`TERMS-MINOR-AGE-DECLINE-01-manual-verify.md`](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md))
- [ ] `AUTH-LOGOUT-ZERO-01` — `artifact/design/user-identity/auth-logout-flow.md` 초안 (PRD ✅)

### 개발 운영

- [ ] 보드 `PHASE17-LOCAL-PROD-REGRESSION-01` Step 5·verify SoT와 일지 §2 동기화
- [ ] (선택) 약관 PRD 0a 결정(05-19→20) — design/plan 반영 여부 점검

---

## 2. 주요 변경 사항

### 2.1 `TERMS-MINOR-AGE-DECLINE-01` 수동 verify

- SoT: [`artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md`](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md) — **verify done**
- **D6 경로:** OAuth 직후·프로필 제출 전 decline → `social_accounts`·`identity_audit_log` 없음 = as-built (D4/D5 skip)
- **N2:** `JSESSIONID`만 삭제 시 `/login`(쿼리 없음) — N3와 동일. 재현은 dev-login → `/profile/setup` → decline → `login?error=no_pending_profile`
- OAuth 반복용 초기화: [`TERMS-MINOR-AGE-DECLINE-01-reset-oauth-test-account.sql`](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-reset-oauth-test-account.sql)
- 보드: `TERMS-MINOR-AGE-DECLINE-01` → [`archive-history-2026-05-24.md`](../../artifact/ops/tasks/archive/archive-history-2026-05-24.md)

### Planet645

_(당일 verify만 — 코드 변경 없음)_

### 개발 운영

- verify §6 최종 Pass · 보드·아카이브 인덱스 갱신

---

## 3. 문제와 해결

### Planet645

_(해당 없음)_

### 개발 운영

- **N2 vs N3:** 세션 쿠키만 제거하면 Security가 먼저 `/login`으로 보냄 — decline API의 `no_pending_profile`과는 다른 진입
- **OAuth 초기화 SQL:** §2 hard-delete 시 `testuser-e2e-user`에 묶인 Google social까지 삭제되면 dev-login 시드 identity 깨짐 → bootRun 재기동 또는 §3 시드 이메일 분리 사용

---

## 4. 배운 점

- as-built에서 **social 계정 생성**은 OAuth 콜백 직후가 아니라 `linkSocialAccount`(프로필 제출 또는 EXACT_MATCH) 시점이다 — decline verify는 이 전제를 전제로 D4/D5를 skip한다.

---

## 5. 다음 액션

> 잔여 출처: [05-19 §5](/2026-05-19-developer-log#5-다음-액션) · 보드 [`artifact/ops/tasks/board.md`](../../artifact/ops/tasks/board.md).

### 5.1 미완

| Task ID | 항목 |
|---------|------|
| `PHASE17-LOCAL-PROD-REGRESSION-01` | Step 5 prod — M1-4·M4 재검증 |
| `AUTH-LOGOUT-ZERO-01` | design → plan → implement |
| `MYPAGE-CREDIT-PAGE-01` | design (백로그 `200`) |
| `MYPAGE-MYINFO-NAV-01` | design — [05-19 결정 메모](/2026-05-19-developer-log#25-mypage-myinfo-nav-01-착수-전-결정-메모-planet645--문서만) 참고 |

### 5.2 완료 (당일 — 상세는 §2)

| Task ID | 항목 |
|---------|------|
| `TERMS-MINOR-AGE-DECLINE-01` | 수동 verify Pass · 보드 아카이브 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` |
| 회귀 | **로컬 prod-profile** (스테이징 없음) |
| 보드 SoT | `artifact/ops/tasks/board.md` — Active `(none)`, Backlog `115`→`250` |
| 선행 (05-19) | validate drift 완료 · Step 3–4·6 green · Step 5 prod M1-4·M4 결함 → 온보딩·optional-terms sync 수정 |
