---
title: "개발일지 — 2026-05-19 (선택 약관 철회·보드 정리)"
excerpt: "TERMS-OPTIONAL-CONSENT-WITHDRAW-01: optional-terms 동의·철회, TERM-004 철회 시 마케팅 채널 OFF. Spring PR #11·워크스페이스 보드 아카이브 #16 merge."
categories: [deVlog]
tags: [planet645, spring, terms, mypage, marketing, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-19 10:00:00 +0900
last_modified_at: 2026-05-19 22:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` 구현·PR merge. 저녁에는 `board.md`에 PHASE17 prod validate·회귀 태스크 복원 및 `executionOrder` 밴드 재정렬, `MYPAGE-MYINFO-NAV-01` 착수 전 IA 결정 메모 정리.

## 1. 오늘 목표

### Planet645

- [x] `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` — optional-terms 동의·철회 UX + 서비스
- [x] 단위 테스트 (`TermsConsentServiceTest`, `ProfileEditControllerTest`)
- [x] Spring PR [#11](https://github.com/JungMockdan/com.mockdan.life-saver-lotto/pull/11) merge

### 개발 운영

- [x] 보드 Active에서 태스크 아카이브 · [2026-05-18 일지](/2026-05-18-developer-log) 다음 액션 정합
- [x] 워크스페이스 PR [#16](https://github.com/JungMockdan/com.mockdan.life-saver-lotto-workspace/pull/16) merge
- [ ] GitHub Pages push (05-18·05-19 일지)

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
| `TERMS-MINOR-AGE-DECLINE-01` | `110` | `230` → 약관 밴드 |
| `MYPAGE-CREDIT-PAGE-01` | `200` | `210` → 마이페이지 밴드 |
| `MYPAGE-MYINFO-NAV-01` | `210` | 선행: CREDIT-PAGE |
| `PHASE22-BILLING-SANDBOX-01` | `220` | … |

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

> **SoT:** 완료 태스크는 `artifact/ops/tasks/archive-history-index.md`. PHASE17 회귀·스키마 drift는 `board.md` Backlog **`105`/`106`**에 복원됨(본 세션).

### Planet645 — 완료 (이전 세션·아카이브)

- [x] **`PHASE17-02-FOLLOWUP-MKT-SEND-CHECK`** — [05-18 §2.8](/2026-05-18-developer-log#28-저녁--수동-verify--phase17-02-followup-mkt-send-check-planet645) · 아카이브 `2026-05-19T00:20:00Z`
- [x] `NOTIF-MKT-SEND-GUARD-01` · `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` — [05-18·05-19 일지](/2026-05-18-developer-log) §5

### Planet645 — 미완

- [ ] optional-terms → marketing-channels **UI 수동 스모크**
- [ ] Google OAuth 재로그인 — 대시보드 프로필·통계
- [ ] 스키마 rename PR (Spring + ML)
- [ ] **`PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01`** (선행)
- [ ] **`PHASE17-LOCAL-PROD-REGRESSION-01` 본선** — 05-13 1차 시도(validate 차단) 이후 재개
- [ ] `MYPAGE-CREDIT-PAGE-01` (보드 Backlog `200`)
- [ ] `MYPAGE-MYINFO-NAV-01` — design 단계 IA 결정(§2.4) 후 implement

### 개발 운영

- [x] `board.md` Backlog — PHASE17 복원(`105`/`106`) · executionOrder 밴드 재정렬(§2.3)
- [x] `phase17-product-engineering-backlog.md` §C.5 SoT 경로 정합
- [x] `mypage-drawer.spec.ts` as-built 카피 정합
- [ ] GitHub Pages push (05-18·05-19 일지)

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto/` · merge `73ec8a5` (PR #11) |
| 회귀 | 스테이징 없음 → **로컬 prod-profile** |
| 보드 SoT | `artifact/ops/tasks/board.md` — Active 비어 있음 (2026-05-19) |
