---
title: "개발일지 — 2026-05-24 (Billing dormant · PG 보류)"
excerpt: "PG 입점 보류 — Mock 결제·구독 UI dormant(BILLING-DORMANT-01). 플래그·V3 FREE 리셋·431 tests green. Sandbox/Refund 보드 cancelled, 다음 SECURITY-COOKIE-SESSION-01."
categories: [deVlog]
tags: [planet645, billing, subscription, spring, verify, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-24 09:00:00 +0900
last_modified_at: 2026-05-24 18:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** PG 입점 승인 불확실로 **실 PG·구독 노출 중단**, Mock completion·스키마는 **dormant 보존**. `app.billing.enabled` / `checkout-ui.enabled` 이중 플래그, Flyway V3 전원 FREE, 크레딧 부족 UX 중립화. 보드 Sandbox·Refund **cancelled** — 다음 밴드 **235** 세션 쿠키.

## 1. 오늘 목표

### Planet645

- [x] Billing·구독 **Mock 수준 마무리** — prod 결제·mock checkout 차단
- [x] 구독 UI 숨김 + local 설정으로만 재노출
- [x] FREE 크레딧 정책(A) 유지 · 전원 FREE DB 리셋(V3)
- [x] `./gradlew test` green (431 passed)

### 개발 운영

- [x] `board.md` · `roadmap-current.md` — Phase 2.2 **deferred**, Sandbox/Refund cancelled
- [x] 코드 리뷰 — RecommendControllerTest assertion·billing disabled 테스트 보강

---

## 2. 주요 변경 사항

### 2.1 BILLING-DORMANT-01 — 결제·구독 dormant (Planet645)

**제품 결정:** PG 입점 승인 전까지 **실 PG 연동 보류**. Mock PG·completion handler·Flyway 스키마는 **재개용으로 유지**.

| 플래그 | 기본 | 역할 |
|--------|------|------|
| `app.billing.enabled` | `false` | checkout API · webhook · mock completion · `isPaidUser` |
| `app.billing.checkout-ui.enabled` | `false` | `/subscribe` · bottom nav · myinfo · upsell |

| 영역 | 동작 (dormant) |
|------|----------------|
| `BillingGate` | `requireEnabled()` → 503 ProblemDetail |
| `SubscribeController` | `@ConditionalOnProperty(checkout-ui)` — off 시 라우트 미등록 |
| `GET /api/v1/subscriptions/me` | billing off → **FREE** |
| `POST /api/v1/billing/**` | **503** |
| `TossWebhookController` | billing off → **503** |
| `CreditBatchServiceImpl` | 월간 구독 reload skip · **free 만료 배치 유지** |
| Flyway **V3** | `user_subscriptions` 전원 FREE · SUBSCRIPTION `credit_entitlement` 0화 · `payment` 보존 |

**UI:** 구독 탭 제거(3탭) · 크레딧 부족 → `/mypage/credits` · 알림 copy 「크레딧·서비스 안내」.

**local mock 재개:** `application-local.properties` 주석 — 두 플래그 **함께** `true`.

**검증:** `BillingDormantIntegrationTest` + billing 단위·통합 보강 · **431 tests passed**.

### 2.2 보드·로드맵 정리 (개발 운영)

| 항목 | 변경 |
|------|------|
| `PHASE22-BILLING-SANDBOX-01` · `PHASE22-BILLING-REFUND-01` | **cancelled** — PG 재개 시 FOLLOWUP |
| `roadmap-current.md` Phase 2.2 | **deferred** (PG 입점 후) |
| Active 다음 | `SECURITY-COOKIE-SESSION-01` (235) |

SoT: `artifact/ops/tasks/board.md`, `artifact/plan/roadmap-current.md`.

---

## 3. 문제와 해결

### Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| 전체 test 1 fail | 크레딧 부족 메시지 변경 후 assertion 미갱신 | `RecommendControllerTest` 기대값 갱신 |
| prod mock PRO 우회 | Mock adapter가 checkout 시 completion | `BillingGate` 다층 차단 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- PG 심사와 **코드 완성도**는 분리 가능 — **dormant + 플래그**로 베타(무료)와 유료 재개 경로를 동시에 유지.
- UI·결제 플래그 **분리**로 local에서만 mock UX 재현 가능.

---

## 5. 다음 액션

> SoT: `artifact/ops/tasks/board.md`.

### 5.1 미완

#### Planet645

- [ ] `SECURITY-COOKIE-SESSION-01` — design → implement (Billing 다음 235)
- [ ] 토스페이먼츠 가맹 **사전 문의**(carry — [2026-05-22](2026-05-22-developer-log.md) §5.1)
- [ ] Payletter 또는 PortOne **동일 문의**(Plan B)

#### 개발 운영

- [ ] 사업자등록(722000·724000) · 통신판매업 신고
- [ ] 도메인·호스팅 방향 결정
- [ ] `/verify-next` — `SECURITY-COOKIE-SESSION-01` (앱 기동 후)

### 5.2 완료 (당일 — 상세 §2)

| §2 | 항목 |
|----|------|
| 2.1 | BILLING-DORMANT-01 — 플래그·UI·V3·테스트 |
| 2.2 | 보드 cancelled · roadmap deferred |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Billing dormant | `com.mockdan.life-saver-lotto` — `BillingGate`, V3 migration |
| 보드 | `artifact/ops/tasks/board.md` — 다음 235 |
| 환경 표기 | **로컬 prod-profile 회귀** |
| 전일 일지 | [2026-05-22-developer-log.md](2026-05-22-developer-log.md) |
