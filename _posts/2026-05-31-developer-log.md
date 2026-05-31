---
title: "개발일지 — 2026-05-31 (동행복권 정합 · 문자 시드 · flow-up)"
excerpt: "동행복권 정합·/info/draw645. 문자 시드(text-seed-v1) 미리보기 범위 A·verify PASS. 추천 v1 unit/verify 아카이브·보드 flow-up."
categories: [deVlog]
tags: [planet645, spring, thymeleaf, ui, dhlottery, recommend, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-31 17:31:58 +0900
last_modified_at: 2026-05-31 21:48:53 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** [동행복권 소개](https://m.dhlottery.co.kr/lt645/intro) 정합 → **랜딩 · 볼색 · `/info/draw645`**. **문자 시드**(`text-seed-v1`) — 문장→결정적 6번호 미리보기·verify T01–T07 PASS. **5/17–5/30 flow-up** — 보드 Backlog `262`–`271` ([§2.3](#23-개발-운영--후속-flow-up-517530)).

## 1. 오늘 목표

### Planet645

- [x] 동행복권 vs Planet645 도메인·UI 정합 점검
- [x] 랜딩 솔직 엔터테인 톤 · 로또볼 공식 구간 · `/info/draw645`
- [x] 문자 시드 미리보기 범위 A (`FEAT-TEXT-SEED-PREVIEW-A-01`)

### 개발 운영

- [x] 5/17–5/30 일지 §5 후속 flow-up · 보드 Backlog 재등록
- [x] 추천 v1 unit · verify runbook PASS → 아카이브

---

## 2. 주요 변경 사항

### 2.1 동행복권 정합 점검 (Planet645)

| 영역 | 판정 | 비고 |
|------|------|------|
| 당첨 등수 · 번호 범위 · 크롤·스케줄 | 일치 | `LottoOfficialRankCalculator`, 1~45, 토 20:00/20:35~ |
| 서비스 성격 | 일치 | 비판매 · 엔터테인 · LLM 도박성 필터 |
| 랜딩 카피 · 로또볼 · 기초 정보 UX | 불일치/부재 | → [§2.2](#22-정합-보완-구현-planet645) |

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
| 코드 | `TextSeedNumberExtractor` · `RecommendController` · 단위·컨트롤러 테스트 green |

### 2.4 개발 운영 — 후속 flow-up (5/17–5/30)

범위: [5/17](2026-05-17-developer-log.md)–[5/30](2026-05-30-developer-log.md) 일지 §5 · `board.md` · `archive-history-2026-06-01.md`.

| 지표 | 상태 |
|------|------|
| 구현 → 아카이브 | ✅ `RECOMMEND-NUMBERS-V1-01` · `RECOMMEND-V1-UNIT-01` · `RECOMMEND-V1-VERIFY-01` |
| §5 → 보드 | ✅ Backlog `262`–`271` ([`board.md`](../../artifact/ops/tasks/board.md)) |
| 문자 시드 | ✅ `FEAT-TEXT-SEED-PREVIEW-A-01` implement·verify — [`FEAT-TEXT-SEED-PREVIEW-A-01-runbook.md`](../../artifact/ops/verify/FEAT-TEXT-SEED-PREVIEW-A-01-runbook.md) T01–T07 PASS (`8081`; `8080` 구빌드 preview 500) |

**판정:** 큰 기능·후속 verify는 아카이브까지 연결. **문자 시드**는 범위 A 완료·`review_pending`.

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

---

## 5. 다음 액션

### 5.1 미완

#### Planet645

- [ ] `INFO-DRAW645-UX-01` — detail 확률 라벨 · prizeLabel · info 링크
- [ ] `PROFILE-ADDRESS-VERIFY-01` — 주소 수동 verify
- [ ] `IDENTITY-CI-PASS-01` — PASS/NICE · CI E2E
- [ ] (선택) `RECOMMEND-V1-E2E-01` · `AD-MISSION-BETA-01`
- [ ] (선택) 문자 시드 **범위 B** — 고정 번호 채우기

#### 개발 운영

- [ ] `SECURITY-SECRETS-CI-01` — secrets-guard CI
- [ ] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| 동행복권 정합 점검 | [§2.1](#21-동행복권-정합-점검-planet645) |
| 랜딩 · 볼색 · `/info/draw645` | [§2.2](#22-정합-보완-구현-planet645) |
| 문자 시드 미리보기 A · verify PASS · 보드 아카이브 | [§2.3](#23-문자-시드-미리보기--범위-a-planet645) |
| flow-up · Backlog · 추천 v1 unit/verify 아카이브 | [§2.4](#24-개발-운영--후속-flow-up-517530) |

---

## 6. 기준

- 공식 SoT: [동행복권 로또6/45 소개](https://m.dhlottery.co.kr/lt645/intro)
- Planet645: 번호 추천·통계 **엔터테인먼트**(복권 판매·당첨금 지급 아님)
- 문자 시드: [`text-seed-base45-preview-spec.md`](../../artifact/design/recommendation-ml/text-seed-base45-preview-spec.md) · verify [`runs/2026-05-31-FEAT-TEXT-SEED-PREVIEW-A-01.md`](../../artifact/ops/verify/runs/2026-05-31-FEAT-TEXT-SEED-PREVIEW-A-01.md)
- Flow-up SoT: `artifact/ops/tasks/board.md` · `archive/archive-history-2026-06-01.md`
