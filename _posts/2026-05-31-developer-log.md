---
title: "개발일지 — 2026-05-31 (동행복권 정합 · CI anchor · flow-up)"
excerpt: "동행복권 정합·draw645 UX·문자 시드 A·주소 verify PASS. CI anchor·FREE_SIGNUP duplicate gate verify PASS. external API inventory SoT."
categories: [deVlog]
tags: [planet645, spring, thymeleaf, ui, dhlottery, recommend, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-31 17:31:58 +0900
last_modified_at: 2026-05-31 22:40:19 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** [동행복권 소개](https://m.dhlottery.co.kr/lt645/intro) 정합 → **랜딩 · 볼색 · `/info/draw645` · UX 후속**. **문자 시드 A** · **주소 verify** · **CI anchor** verify PASS. **external API inventory** SoT. Backlog `264`–`271` ([§2.4](#24-개발-운영--후속-flow-up-517530)).

## 1. 오늘 목표

### Planet645

- [x] 동행복권 vs Planet645 도메인·UI 정합 점검
- [x] 랜딩 솔직 엔터테인 톤 · 로또볼 공식 구간 · `/info/draw645`
- [x] 문자 시드 미리보기 범위 A (`FEAT-TEXT-SEED-PREVIEW-A-01`)
- [x] draw645 UX 후속 (`INFO-DRAW645-UX-01`)
- [x] 프로필 주소 verify (`PROFILE-ADDRESS-VERIFY-01`)
- [x] CI anchor · FREE_SIGNUP duplicate gate (`IDENTITY-CI-PASS-01`)

### 개발 운영

- [x] 5/17–5/30 일지 §5 후속 flow-up · 보드 Backlog 재등록
- [x] 추천 v1 unit · verify runbook PASS → 아카이브
- [x] 문자 시드 범위 B 보드 등록 (`FEAT-TEXT-SEED-PREVIEW-B-01`)
- [x] external API inventory SoT (`external-integrations.md`)

---

## 2. 주요 변경 사항

### 2.1 동행복권 정합 점검 (Planet645)

| 영역 | 판정 | 비고 |
|------|------|------|
| 당첨 등수 · 번호 범위 · 크롤·스케줄 | 일치 | `LottoOfficialRankCalculator`, 1~45, 토 20:00/20:35~ |
| 서비스 성격 | 일치 | 비판매 · 엔터테인 · LLM 도박성 필터 |
| 랜딩 카피 · 로또볼 · 기초 정보 UX | 불일치/부재 | → [§2.2](#22-정합-보완-구현-planet645) · [§2.5](#25-draw645-ux-후속-planet645) |

### 2.2 정합 보완 구현 (Planet645)

| 항목 | 변경 |
|------|------|
| **랜딩** `landing.html` | 「당첨을 보장하지 않는 엔터테인먼트」·재미용 카피 · `/info/draw645` 링크 |
| **볼 색** `lotto-ball.html`, `design-system.css` | 1–10 / 11–20 / 21–30 / 31–40(회색) / 41–45 |
| **기초 정보** `GET /info/draw645` | `InfoController` · `content/info/draw645.html` · `/info/**` permitAll |

### 2.3 문자 시드 미리보기 — 범위 A (Planet645)

문장 입력 → **항상 같은 6번호(1~45)** · ML·크레딧·세션 무관. SoT: [`text-seed-base45-preview-spec.md`](../../artifact/design/recommendation-ml/text-seed-base45-preview-spec.md).

| 항목 | 내용 |
|------|------|
| 알고리즘 | `text-seed-v1` — NFC → SHA-256 → 바이트 `% 45 + 1` (6개 유일·오름차순) |
| Golden | `planet645` → `6,9,21,26,35,44` · `안녕` → `8,19,24,26,30,33` |
| API/UI | `GET /recommend/text-seed/preview` · `text-seed-preview.html` · select 1단계 `<details>` |
| verify | [`FEAT-TEXT-SEED-PREVIEW-A-01-runbook.md`](../../artifact/ops/verify/FEAT-TEXT-SEED-PREVIEW-A-01-runbook.md) T01–T07 PASS (`8081`) |

### 2.4 개발 운영 — 후속 flow-up (5/17–5/30)

범위: [5/17](2026-05-17-developer-log.md)–[5/30](2026-05-30-developer-log.md) 일지 §5 · [`board.md`](../../artifact/ops/tasks/board.md) · [`archive-history-2026-06-01.md`](../../artifact/ops/tasks/archive/archive-history-2026-06-01.md).

| 지표 | 상태 |
|------|------|
| 구현 → 아카이브 | ✅ `RECOMMEND-NUMBERS-V1-01` · `RECOMMEND-V1-UNIT-01` · `RECOMMEND-V1-VERIFY-01` · `FEAT-TEXT-SEED-PREVIEW-A-01` · `INFO-DRAW645-UX-01` |
| §5 → 보드 | ✅ Backlog `264`–`271` — `FEAT-TEXT-SEED-PREVIEW-B-01`(271) 신규 |

### 2.5 draw645 UX 후속 (Planet645)

`INFO-DRAW645-UX-01` — 기초 정보 페이지는 선행 완료, **노출·라벨·대시보드**만 후속.

| 항목 | 변경 |
|------|------|
| **링크** | `login.html` · `fragments/site-footer.html` · `auth_layout` 푸터 |
| **detail** | 미사용 `probability` 「당첨 확률」 카드 제거 |
| **대시보드** | `LottoDrawDetail.findTopByOrderByLtEpsdDesc` → `prizeLabel` (`{회차}회 1등 {금액}원`) |

### 2.6 프로필 주소 verify (Planet645)

`PROFILE-ADDRESS-VERIFY-01` — Daum 팝업 경로 대신 `/mypage/profile` POST·재조회·DB 수동 verify.

| 항목 | 내용 |
|------|------|
| verify | [`PROFILE-ADDRESS-VERIFY-01-runbook`](../../artifact/ops/verify/PROFILE-ADDRESS-VERIFY-01-runbook.md) A01–A05 **PASS** ([run](../../artifact/ops/verify/runs/2026-05-31-PROFILE-ADDRESS-VERIFY-01.md)) |

### 2.7 개발 운영 — external API inventory SoT

사업자등록·계약 **일괄 처리** 전까지 Tier A/B/I로 외부 연동 목록·갈아끼우기 경계 정리.

| 항목 | 내용 |
|------|------|
| 문서 | [`external-integrations.md`](../../artifact/as-built/external-integrations.md) · `as-built/README` · `06-integrations-and-config` cross-link |
| 정책 | Tier **A**(지금 OK: 동행복권·Google OAuth·Daum postcode·OpenAI dev) · **B**(PG·NICE·AdSense 등 batch 후) |

### 2.8 CI anchor · FREE_SIGNUP gate (Planet645)

`IDENTITY-CI-PASS-01` — scaffold 후속. CI 원문 미저장 · SHA-256(`CI_ANCHOR_SALT` + CI)만 `user_ci_anchor`.

| 항목 | 내용 |
|------|------|
| 플로우 | `/identity/verification/start` · NICE callback scaffold · 로컬 `/dev/ci-verify?ciToken=…` |
| Entitlement | `grantFreeSignupCredits` — 앵커 필수 · 동일 CI 다른 계정 → duplicate 차단 |
| 가드 | `scripts/check-ci-anchor-guard.sh` · default `CI_ANCHOR_ENABLED=false` |
| verify | [`IDENTITY-CI-PASS-01-runbook`](../../artifact/ops/verify/IDENTITY-CI-PASS-01-runbook.md) C01–C06 **PASS** ([run](../../artifact/ops/verify/runs/2026-05-31-IDENTITY-CI-PASS-01.md)) |
| 잔여 | NICE `EncodeData` decrypt — vendor SDK 후속 |

---

## 3. 문제와 해결

### 3.1 랜딩 「당첨 확률」 (Planet645)

| 문제 | 해결 |
|------|------|
| 당첨 확률 상승 암시 | 비보장·재미용 톤 ([§2.2](#22-정합-보완-구현-planet645)) |

### 3.2 로또볼 구간 (Planet645)

| 문제 | 해결 |
|------|------|
| 1–9 구간·31–40 주황 vs 공식 | 1–10…41–45 · 회색 ([§2.2](#22-정합-보완-구현-planet645)) |

### 3.3 문자 시드 로컬 포트 (Planet645)

| 문제 | 해결 |
|------|------|
| `8080` preview 500 | 최신 `bootRun` **`8081`** 에서 T01–T07 PASS ([§2.3](#23-문자-시드-미리보기--범위-a-planet645)) |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 도메인·크롤은 정합; **카피·볼 색·참조 UX**가 정보 서비스 빈틈이었다.
- **보드 clear 후** devlog §5만으로는 후속이 끊긴다 — flow-up + Backlog 재등록이 필요.
- 재미 기능(문자 시드)은 **추천 파이프라인과 분리**하면 범위 A를 빠르게 닫을 수 있다.
- **Tier A/B**로 외부 연동을 나누면 사업자등록 전에도 Mock·flag off만 유지하면서 갈아끼울 수 있다.

---

## 5. 다음 액션

### 5.1 미완

#### Planet645

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [ ] (선택) `RECOMMEND-V1-E2E-01` · `AD-MISSION-BETA-01`

#### 개발 운영

- [ ] `SECURITY-SECRETS-CI-01` — secrets-guard CI
- [ ] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| 동행복권 정합 점검 | [§2.1](#21-동행복권-정합-점검-planet645) |
| 랜딩 · 볼색 · `/info/draw645` | [§2.2](#22-정합-보완-구현-planet645) |
| 문자 시드 미리보기 A · verify PASS | [§2.3](#23-문자-시드-미리보기--범위-a-planet645) |
| flow-up · Backlog · 아카이브 | [§2.4](#24-개발-운영--후속-flow-up-517530) |
| draw645 UX (`INFO-DRAW645-UX-01`) | [§2.5](#25-draw645-ux-후속-planet645) |
| 프로필 주소 verify PASS | [§2.6](#26-프로필-주소-verify-planet645) |
| external API inventory SoT | [§2.7](#27-개발-운영--external-api-inventory-sot) |
| CI anchor · FREE_SIGNUP gate PASS | [§2.8](#28-ci-anchor--free_signup-gate-planet645) |

---

## 6. 기준

- 공식 SoT: [동행복권 로또6/45 소개](https://m.dhlottery.co.kr/lt645/intro)
- Planet645: 번호 추천·통계 **엔터테인먼트**(복권 판매·당첨금 지급 아님)
- 문자 시드: [`text-seed-base45-preview-spec.md`](../../artifact/design/recommendation-ml/text-seed-base45-preview-spec.md) · verify [`runs/2026-05-31-FEAT-TEXT-SEED-PREVIEW-A-01.md`](../../artifact/ops/verify/runs/2026-05-31-FEAT-TEXT-SEED-PREVIEW-A-01.md)
- Flow-up SoT: `artifact/ops/tasks/board.md` · `archive/archive-history-2026-06-01.md`
- External API: [`external-integrations.md`](../../artifact/as-built/external-integrations.md)
