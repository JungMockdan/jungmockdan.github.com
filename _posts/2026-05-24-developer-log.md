---
title: "개발일지 — 2026-05-24 (Billing dormant · HTTP 보안 헤더)"
excerpt: "직접결제 중단·광고 BM — Billing dormant. prod HTTP 헤더 verify PASS."
categories: [deVlog]
tags: [planet645, billing, security, spring, verify, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-24 09:00:00 +0900
last_modified_at: 2026-05-24 22:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** **BM 전환** — 당분간 **직접결제·PG 없음**, Billing **dormant** · 수익 축은 **광고·미션 REWARD**(후속 에픽). prod-profile **HTTP 보안 헤더** verify **PASS** · 보드 아카이브.

## 1. 오늘 목표

### Planet645

- [x] Billing·구독 **Mock 수준 마무리** — prod 결제·mock checkout 차단
- [x] prod HTTP 보안 헤더 baseline (`SECURITY-HTTP-HEADERS-01`)
- [x] `./gradlew test` green

### 개발 운영

- [x] `board.md` · `roadmap-current.md` — Phase 2.2 **deferred**, Sandbox/Refund cancelled
- [x] HTTP 헤더 design delta · verify runbook · run record · 보드 아카이브

---

## 2. 주요 변경 사항

### 2.1 BILLING-DORMANT-01 — 결제·구독 dormant (Planet645)

**제품 결정:** **직접결제·PG 입점 중단**(토스 등 사전 문의 **하지 않음**). 당분간 **광고(AdSense)·미션 REWARD**로 BM — Mock PG·Flyway는 **재개용으로만** 코드 보존(`app.billing.*` off).

| 플래그 | 기본 | 역할 |
|--------|------|------|
| `app.billing.enabled` | `false` | checkout API · webhook · mock completion · `isPaidUser` |
| `app.billing.checkout-ui.enabled` | `false` | `/subscribe` · bottom nav · myinfo · upsell |

| 영역 | 동작 (dormant) |
|------|----------------|
| `BillingGate` | `requireEnabled()` → 503 ProblemDetail |
| `SubscribeController` | `@ConditionalOnProperty(checkout-ui)` — off 시 라우트 미등록 |
| Flyway **V3** | `user_subscriptions` 전원 FREE · SUBSCRIPTION `credit_entitlement` 0화 |

**검증:** `BillingDormantIntegrationTest` + billing 단위·통합 보강.

### 2.2 SECURITY-HTTP-HEADERS-01 — prod HTTP 보안 헤더 (Planet645)

**제품 결정:** v1은 **enforce CSP 없음** · **CSP Report-Only**로 위반만 관찰 · **HSTS 후속** · **local 변경 없음** · **Swagger prod off**.

| 항목 | prod 동작 |
|------|-----------|
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Content-Security-Policy-Report-Only` | aspirational 정책 (인라인·CDN 위반 DevTools 관찰) |
| springdoc | `api-docs` / `swagger-ui` **disabled** |

**구현:** `app.security.http-headers.enabled=true` (`application-prod.properties`) → `ProdHttpSecurityHeaders` + `SecurityConfig`. MockMvc `HttpSecurityHeadersLocalTest` / `HttpSecurityHeadersProdTest` (T-HDR-0~3).

**후속:** design delta §9 F-HDR-1~10 (CSP enforce·nonce·HSTS·ML/LLM 헤더 등).

### 2.3 보드·로드맵 · SoT 정리 (개발 운영)

| 항목 | 변경 |
|------|------|
| `PHASE22-BILLING-SANDBOX-01` · `REFUND-01` | **cancelled** |
| `SECURITY-HTTP-HEADERS-01` | design → implement → verify **PASS** → **archived** |
| `SECURITY-COOKIE-SESSION-01` | verify PASS (당일 아카이브) |

SoT: [`http-headers-design-delta.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/design/security/http-headers-design-delta.md), [`SECURITY-HTTP-HEADERS-01-runbook.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/ops/verify/SECURITY-HTTP-HEADERS-01-runbook.md).

---

## 3. 문제와 해결

### 3.1 Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| MockMvc prod 헤더 미적용 | `@Nested` + profile 조건 타이밍 | `app.security.http-headers.enabled` property + top-level test class 분리 |
| CSP/Referrer `HttpSecurity.headers()` 미반영 | Spring Security writer 조합 | `HeaderWriterFilter` 앞 커스텀 필터로 Report-Only·Referrer 주입 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- **결제 코드**와 **당장의 BM**은 분리 — dormant로 PG 재개 옵션만 남기고, 당분간 운영 초점은 **광고·미션** 쪽.
- CSP는 **Report-Only → 정리 → enforce** 단계가 Thymeleaf SSR에 현실적.

---

## 5. 다음 액션

> SoT: `artifact/ops/tasks/board.md`.

### 5.1 미완

#### Planet645

- [ ] Ad·미션·약관(광고·추적) 에픽 — **후속 스프린트**(5/26 verify 등)

#### 개발 운영

- [ ] (선택) 사업자등록 — 광고·서비스 운영; **통신판매업·PG**는 직접결제 재개 시
- [ ] (선택) prod OAuth 로그인 후 onboarding/recommend **브라우저** CSP Report-Only 위반 spot (S07)

### 5.2 완료 (당일 — 상세 §2)

| §2 | 항목 |
|----|------|
| 2.1 | BILLING-DORMANT-01 |
| 2.2 | SECURITY-HTTP-HEADERS-01 — implement + verify PASS |
| 2.3 | 보드·SoT · HTTP 헤더 아카이브 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| HTTP 헤더 | `com.mockdan.life-saver-lotto` — `ProdHttpSecurityHeaders`, `application-prod.properties` |
| Verify | [`runs/2026-05-24-SECURITY-HTTP-HEADERS-01.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/ops/verify/runs/2026-05-24-SECURITY-HTTP-HEADERS-01.md) |
| 환경 표기 | **로컬 prod-profile 회귀** (`localhost:18080`, compose override) |
| 전일 일지 | [2026-05-22-developer-log.md](2026-05-22-developer-log.md) |
