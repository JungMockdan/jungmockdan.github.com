---
title: "개발일지 — 2026-06-04 (평가 M1~M4 · ML 트랙 B 완료)"
excerpt: "평가 M1~M4(OCR 제외) · OWASP stage 2 · ML B1~B3(NDCG·hard neg·isotonic mlScore)"
categories: [deVlog]
tags: [planet645, evaluate, security, ml, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-04 08:48:06 +0900
last_modified_at: 2026-06-04 12:54:42 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 내 번호 평가 **M1 verify~M4**(OCR 제외) · OWASP **stage 2** · ML 내실화 **B1~B3** 완료(오프라인 NDCG · hard negative · isotonic mlScore).

## 1. 오늘 목표

### Planet645

- [x] M1 verify · M2 LLM · M3 QR · M4 CTA — §2.1~§2.4
- [x] ML-RANKING-INTERNALS-01~03 — §2.5~§2.7

### 개발 운영

- [x] OWASP stage 2 — §2.8
- [x] ML 트랙 B 보드 아카이브 — §2.8

---

## 2. 주요 변경 사항

### 2.1 M1 verify (Planet645)

| 산출 | 경로 |
|------|------|
| Runbook | `artifact/ops/verify/FEAT-MY-NUMBER-EVAL-M1-VERIFY-01-runbook.md` |
| Run | `artifact/ops/verify/runs/2026-06-04-FEAT-MY-NUMBER-EVAL-M1-VERIFY-01.md` |

| 시나리오 | 판정 |
|----------|------|
| S01 STANDALONE paste→report | PASS |
| S02 붙여넣기 2게임 | PASS |
| S03b 단위·컨트롤러 회귀 | PASS |

### 2.2 M2 LLM commentary (Planet645)

| 산출 | 경로 |
|------|------|
| Spring | `evaluation/commentary/EvaluationCommentaryService*` · `EvaluateController` |
| LLM | `POST /evaluate-comment` · `core/commentary.py` |

PREMIUM 구독 ACTIVE만 해설 · 크레딧 미차감(OQ-1).

### 2.3 M3 QR 캡처 (Planet645)

| 산출 | 내용 |
|------|------|
| `input.html` | manual · paste · **qr** · jsQR |
| OCR | 후속(Q1.A) |

### 2.4 M4 추천 CTA (Planet645)

| 산출 | 경로 |
|------|------|
| Design delta | `artifact/design/recommendation-ml/slip-exclude-recommend-delta.md` |
| UI | CTA → `/recommend/select?excludeNumbers=…` |

### 2.5 ML-RANKING-INTERNALS-01 (Planet645)

| 산출 | 경로 |
|------|------|
| 오프라인 평가 | `com.mockdan.life-saver-lotto-ml/core/offline_eval.py` |
| 리포트 | `reports/ml_offline_eval.json` · `.md` |

시간 분할 · 분류 vs NDCG@k/MRR 분리.

### 2.6 ML-RANKING-INTERNALS-02 (Planet645)

| 산출 | 내용 |
|------|------|
| `core/negative_sampling.py` | 회차당 음성 3종: random · near-miss(5/6) · pool-distribution |
| `model_training.py` | 양성:음성 1:3 · `hard_mixed_v1` |

풀분포는 당첨선 구간 프로필 프록시(ML 단독, 사용자 DB 미연동).

### 2.7 ML-RANKING-INTERNALS-03 (Planet645)

| 산출 | 내용 |
|------|------|
| `core/calibration.py` | test 구간 isotonic fit |
| `scoring.py` | `predict_proba` 후 보정 · `phase7-xgboost-v1.1-calibrated` |
| `models/score_calibrator.pkl` | 추론 시 로드(없으면 raw proba) |

평가 카드 UI 카피 변경 없음(0~1 상대 점수 유지).

### 2.8 OWASP stage 2 · 보드 (개발 운영)

| 항목 | 값 |
|------|-----|
| Gate | `failBuildOnCVSS=7` · Netty `4.1.135.Final` |
| SoT | `artifact/runbook/security/dependency-cvss-baseline.md` |
| 아카이브 | `ML-RANKING-INTERNALS-01~03` → `archive-history-2026-06-08.md` |

---

## 3. 문제와 해결

### 3.1 Planet645 — verify curl manual 입력

| 문제 | 원인 | 해결 |
|------|------|------|
| POST `/evaluate` 200 + formError | Spring `String[] line` 쉼표 분할 | verify는 paste 모드 |

### 3.2 Planet645 — ML 전량 재학습 환경

| 문제 | 원인 | 해결 |
|------|------|------|
| host `model_training` DB 실패 | `mariadb`/`xgboost` 미설치 | 코드·단위테스트 green; DB 연동 시 `python3 -m core.model_training`로 xgb+calibrator 동시 갱신 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- hard negative는 near-miss만으로도 랭킹 쿼리(회차 풀) 변별이 커진다; 풀분포는 추천 23풀과 **분포 정합**만 맞추는 프록시로 시작 가능.
- isotonic은 API 필드는 그대로 두고 점수 간격만 보정 — M1 카피 동기화 없이 배포 가능.
- curl SSR manual `line[]` 검증 시 Spring 배열 바인딩 쉼표 분할 주의.

---

## 5. 다음 액션

### 5.1 미완

#### Planet645

- [ ] `FEAT-MY-NUMBER-EVAL-01` — M3 **OCR** (Tesseract.js · 후속)

#### 개발 운영

_(해당 없음)_

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| M1 verify | §2.1 |
| M2 LLM | §2.2 |
| M3 QR | §2.3 |
| M4 CTA | §2.4 |
| ML B1 NDCG | §2.5 |
| ML B2 hard neg | §2.6 |
| ML B3 calibration | §2.7 |
| OWASP stage 2 · 보드 | §2.8 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| ML 백로그 | `artifact/plan/ml-vs-service-diversification-6month-backlog.md` |
| 환경 표기 | 로컬 prod-profile 회귀 |
| 전일 일지 | [2026-06-03-developer-log.md](2026-06-03-developer-log.md) |
