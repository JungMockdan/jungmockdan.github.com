---
title: "개발일지 — 2026-06-03 (문자 시드 B · 내 번호 평가 M1)"
excerpt: "문자 시드 B verify PASS · 내 번호 평가 PRD·설계 확정 · M1 구현(/evaluate·공유 카드·PREMIUM 연동) · 런타임·Thymeleaf 수정."
categories: [deVlog]
tags: [planet645, recommend, evaluate, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-03 08:29:37 +0900
last_modified_at: 2026-06-03 23:41:19 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 문자 시드 **범위 B** verify PASS. **내 번호 평가받기** PRD·설계 확정 후 **M1** — `evaluation` 모듈 · `/evaluate` · `evaluation-report-card` · PREMIUM 결과 「이 번호 평가」. E2E 중 `RequestParam`·Thymeleaf `T()` 샌드박스 이슈 수정.

## 1. 오늘 목표

### Planet645

- [x] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B (인라인 미리보기 · 좌/우클릭 칩 · POST PRG · 금칙·로그 마스킹 · verify·보드 아카이브)
- [x] `FEAT-MY-NUMBER-EVAL-01` — **설계** — PRD 확정(OQ-1~7) · `01-evaluation-module` · `02-input-and-report-ui` · `04-slip-capture` · QR 실측 파서
- [x] `FEAT-MY-NUMBER-EVAL-01` — **M1 구현** — STANDALONE `/evaluate` · 공유 fragment · PREMIUM `PremiumEvaluationAdapter`

### 개발 운영

- [ ] (후속) OWASP stage 2 — `failBuildOnCVSS=7` (Netty 42582 등 잔여 HIGH 정리 후)

---

## 2. 주요 변경 사항

### 2.1 문자 시드 범위 B — select 인라인 (Planet645)

| 산출 | 경로 |
|------|------|
| 컨트롤러 | `RecommendController` — 세션 flash · POST preview · 레거시 GET redirect |
| UI | `select.html` — 인라인 칩 · 좌/우클릭 오버레이 |
| 제거 | `text-seed-preview.html` |
| Design | `text-seed-base45-preview-spec.md` §11·§12 |
| Verify | `FEAT-TEXT-SEED-PREVIEW-B-01` T01–T05 PASS |
| 보드 | `FEAT-TEXT-SEED-PREVIEW-B-01` archived |

| 동작 | 내용 |
|------|------|
| 미리보기 | `POST /recommend/select/text-seed/preview` → PRG `GET /recommend/select` |
| 고정/제외 | 왼쪽 클릭 ✓ 고정 · 오른쪽 클릭 ✕ 제외 (입력칸 동기화) |
| 레거시 | `GET /recommend/text-seed/preview` → 세션 flash 후 select (**URL 무 phrase**) |

### 2.2 문구 정책 · 로그 (Planet645)

| 산출 | 경로 |
|------|------|
| 금칙 | `TextSeedPhrasePolicy` · `recommendation/text-seed-blocklist.txt` |
| 로그 | `TextSeedQueryMaskingFilter` — query `phrase`/`q` → `[redacted]` |
| UI 안내 | 미저장 · POST · 금칙 문구 (`select.html`) |

### 2.3 내 번호 평가받기 — PRD·설계 (Planet645)

| 산출 | 경로 |
|------|------|
| PRD **확정** | `artifact/prd/my-number-evaluation.md` — OQ-1~7 |
| 모듈·UI 설계 | `artifact/design/my-number-evaluation/01-evaluation-module.md` · `02-input-and-report-ui.md` |
| 캡처 설계 | `artifact/design/my-number-evaluation/04-slip-capture.md` |
| 고객 카피 | `artifact/prd/numbers-recommendation-customer-guide.md` §14 |

| 항목 | 결정 |
|------|------|
| 포지셔닝 | 번호 추천과 분리 — 「내가 고른/산 번호 평가」 |
| 공유 모듈 | `NumberEvaluationService` — STANDALONE + PREMIUM |
| M1 리포트 | 줄별 ML 참고 점수 · Historical 비목표 |
| 캡처 | QR 1순위 · 붙여넣기 · OCR 폴백(M3) |

### 2.4 내 번호 평가 — M1 구현 (Planet645)

| 산출 | 경로 |
|------|------|
| 도메인 | `com.mockdan.lifesaverlotto.evaluation.*` — `NumberEvaluationServiceImpl` · `MlEvaluationScorer` · 슬립 파서 |
| DB | `V8__user_number_evaluation.sql` · `UserNumberEvaluation` |
| UI | `EvaluateController` — `/evaluate` · `input` · `report` · `history` |
| 공유 카드 | `fragments/evaluation-report-card.html` · `scoreHelpBtnPattern` |
| PREMIUM | `PremiumEvaluationAdapter` · `recommend/result.html` 「이 번호 평가」 |
| 진입 | `dashboard.html` · `tnb.html` |
| 테스트 | slip 단위 3 · `PremiumEvaluationAdapterTest` · `RecommendControllerTest` 보강 |

| 화면 | 동작 |
|------|------|
| EVAL-01 | 1~5게임 직접 입력·붙여넣기 → ML 채점 |
| EVAL-02 | fragment 리포트 · 다줄 슬립 메타 |
| REC-PREMIUM | 선정 카드 + 평가 카드(ML 참고 점수 분리 표시) |

### 2.5 ML 강화 vs 서비스 다각화 — 전략 백로그 (Planet645)

ML 역할을 **후보 풀 랭킹 + 줄별 `mlScore`**로 고정하고, 투자 비중 **제품 다각화 70% / ML 내실화 30%**.

| 산출 | 경로 |
|------|------|
| 전략 | `artifact/plan/ml-vs-service-diversification-6month-backlog.md` |
| 트랙 A | `FEAT-MY-NUMBER-EVAL-01` M1~M4 |
| 트랙 B (low) | `ML-RANKING-INTERNALS-01~03` |

---

## 3. 문제와 해결

### 3.1 Planet645 — M1 런타임

| 문제 | 원인 | 해결 |
|------|------|------|
| `GET /evaluate` 500 · `IllegalArgumentException` (Integer 파라미터) | `@RequestParam` 이름 미지정 · `-parameters` 미반영 빌드 | `EvaluateController`에 `name="gameCount"` 등 명시 |
| `ParseException` · `T()` forbidden (`input` 78행) | Thymeleaf 샌드박스에서 정적 호출 금지 | `lineLabels` 모델 주입 · `history`는 `createdAtDisplay` |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 범위 A 스펙 1차 UI( select 인라인 )가 B에서야 완성됐다 — 독립 URL은 redirect만 남기는 편이 UX·유지보수에 유리하다.
- 시드 6개 vs 고정 최대 5 → **전체 자동 populate**보다 칩 선택 UI가 제약과 맞는다.
- URL 노출은 GET query보다 **POST+세션 flash**가 맞다 — 접근 로그 마스킹과 병행.
- Thymeleaf(Spring 6) 템플릿에서는 `T(StaticClass)` 대신 **컨트롤러/서비스에서 포맷·라벨을 모델로** 넘기는 편이 안전하다.

---

## 5. 다음 액션

### 5.1 미완

> **이월:** [2026-06-02-developer-log.md](2026-06-02-developer-log.md) §5.1 (OWASP) · 당일 M1 완료 후 후속만 유지

#### Planet645

**2026-06-03 후속**

- [ ] `FEAT-MY-NUMBER-EVAL-01` — M2 `EvaluationCommentaryService` · PREMIUM LLM 해설
- [ ] `FEAT-MY-NUMBER-EVAL-01` — M3 QR·OCR 캡처 (`04-slip-capture.md`)
- [ ] `FEAT-MY-NUMBER-EVAL-01` — M4 남은 숫자로 추천 CTA
- [ ] (low) `ML-RANKING-INTERNALS-01~03` — ML 내실화 · 예측 강화 아님
- [ ] (verify) M1 — `/evaluate` · PREMIUM `/recommend/result` 평가 카드 · 로컬 prod-profile 스모크

#### 개발 운영

**2026-06-02 이월**

- [ ] (후속) OWASP stage 2 — `failBuildOnCVSS=7` (Netty 42582 등 잔여 HIGH 정리 후)

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| `FEAT-TEXT-SEED-PREVIEW-B-01` | §2.1–§2.2 |
| `FEAT-MY-NUMBER-EVAL-01` PRD·설계 | §2.3 |
| `FEAT-MY-NUMBER-EVAL-01` M1 구현 + 런타임 수정 | §2.4 · §3.1 |
| ML vs 다각화 전략 백로그 | §2.5 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| 환경 표기 | 로컬 prod-profile 회귀 (staging 표기 없음) |
| 전일 일지 | [2026-06-02-developer-log.md](2026-06-02-developer-log.md) |
| 평가 SoT | `artifact/design/my-number-evaluation/02-input-and-report-ui.md` |
| text-seed SoT | `artifact/design/recommendation-ml/text-seed-base45-preview-spec.md` §11·§12 |
