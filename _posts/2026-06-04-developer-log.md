---
title: "개발일지 — 2026-06-04 (평가·ML B · React promote 4화면)"
excerpt: "평가·ML B · React promote · Lab UI Kit·거친 스케치 질감(design-guide-v02)"
categories: [deVlog]
tags: [planet645, evaluate, security, ml, ui-kit, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-04 08:48:06 +0900
last_modified_at: 2026-06-04 22:40:13 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 평가 M1~M4(OCR 제외) · ML B1~B3 · React promote 4화면 · Lab UI Kit + **거친 선·채색** · 랜딩 flow·CTA.

## 1. 오늘 목표

### Planet645

- [x] M1 verify · M2 LLM · M3 QR · M4 CTA — §2.1~§2.4
- [x] ML-RANKING-INTERNALS-01~03 — §2.5~§2.7
- [x] UI React Lab · promote 4화면 — §2.9~§2.11
- [x] Lab UI Kit · 거친 스케치 질감 — §2.12
- [x] React 랜딩 user flow · SVG·CTA — §2.13

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

### 2.9 UI React Lab · 캐릭터 SVG (Planet645)

| 산출 | 경로 |
|------|------|
| Lab 앱 | `com.mockdan.planet645-ui-lab/` (자체 Git) |
| SVG SoT | Lab `design/planet645-svg-assets/` |
| 랜딩 mock | `CharacterAsset` · `PlanetLogo` |

### 2.10 개발 운영 — Lab 에셋 동기화

| 산출 | 경로 |
|------|------|
| 스크립트 | `npm run sync:svg` · `sync_character_assets.py` (PNG) |

### 2.11 React promote 4화면 (Planet645)

OQ-7 B · verify runbook **스킵** · `UI-REACT-LAB-01` · `UI-REACT-PROMOTE-01` 아카이브.

| URL | Spring | Lab |
|-----|--------|-----|
| `/` | `LandingController` forward | `LandingPage.tsx` |
| `/recommend/result` | forward + `GET /api/v1/ui/recommend/result` | `RecommendResultPage.tsx` |
| `/dashboard` | forward + `GET /api/v1/ui/dashboard` | `DashboardPage.tsx` |
| `/evaluate` | GET forward · POST 유지 | `EvaluateInputPage.tsx` (QR 후속) |

공통: `planet645-react/` static · `npm run build:spring` · `App.tsx` pathname 라우팅 · 갭 `artifact/design/ui/lab/*-thymeleaf-gap-2026-06-04.md`.

### 2.12 Lab UI Kit — design-guide-v02 (Planet645)

`artifact/design/design-guide-v02.png` 기준 스케치 UI 프리미티브(캐릭터 SVG와 분리). v02 **거친 선·거친 채색** 반영.

| 산출 | 경로 |
|------|------|
| SoT | `design/planet645-ui-kit/` |
| 서빙 | `public/planet645-ui-kit/` · `npm run sync:ui-kit` |
| 토큰·컴포넌트 | `tokens.css` · `ui-kit.css` · **`sketch-surfaces.css`** |
| 거친 질감 | 다층 해칭(`--p645-fill-scribble-*`) · `::before` 흔들 테두리 · `::after` 그레인 |
| SVG 필터 | `index.html` `#p645-sketch-wobble` · `LottoBallShell` 교차 해칭·`feDisplacementMap` |
| 아이콘 | `icons/*.svg` · 스프라이트 · stroke 두께 상향 |
| React | `src/ui-kit/` · `UiKitIcon` · `UiKitLottoBall` · `/design-kit` |
| 문서 | `docs/UI-KIT.md` |

`sync:assets` = SVG + UI kit · 랜딩 `sketch-box`·CTA `p645-btn` 동일 톤.

### 2.13 React 랜딩 — user flow · SVG · UI Kit CTA (Planet645)

`user-flow-v1.md` · Spring `LandingController` / `GET /start`와 동기화.

| 산출 | 내용 |
|------|------|
| 인증 | `GET /api/v1/ui/entry-target` · Lab `ensureEntryPath` · CTA는 서버 `/start` 위임 |
| SVG | `manifest.ts` — `import.meta.env.BASE_URL` (`/planet645-react/…`) |
| UI Kit | 랜딩 **시작하기** → `p645-btn--primary` + `UiKitIcon`(`p645-planet`) |
| UX | 헤더 캐릭터 3종·CTA 위 복실 제거 |
| Spring | `AnonymousAuthenticationToken` `/start` → `/login` · `build:spring` 반영 |

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

### 3.3 Planet645 — Spring embed SVG 404

| 문제 | 원인 | 해결 |
|------|------|------|
| `planet645-logo-icon.svg` 404 | Lab 경로 `/planet645-svg-assets` — Vite `base` `/planet645-react/` 미반영 | `BASE_URL` 접두 + `build:spring` |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- hard negative는 near-miss만으로도 랭킹 쿼리(회차 풀) 변별이 커진다; 풀분포는 추천 23풀과 **분포 정합**만 맞추는 프록시로 시작 가능.
- isotonic은 API 필드는 그대로 두고 점수 간격만 보정 — M1 카피 동기화 없이 배포 가능.
- curl SSR manual `line[]` 검증 시 Spring 배열 바인딩 쉼표 분할 주의.
- React promote는 Thymeleaf 레이아웃(TNB) 없이 `PageShell` 링크로 시작; 데이터는 `/api/v1/ui/*` + 세션.
- UI Kit은 `design/` SoT → `public/` 동기화로 URL 공유; **flat 단색만으로는 v02 톤이 안 나와** 해칭+pseudo 테두리+그레인 레이어가 필요.
- post-auth 분기(로그인·프로필·온보딩)는 클라이언트 `/api/v1/me` 추측보다 **`/start`·`entry-target`** 서버 위임이 SoT와 맞다.

---

## 5. 다음 액션

### 5.1 미완

#### Planet645

- [ ] `FEAT-MY-NUMBER-EVAL-01` — M3 **OCR** (후속)
- [ ] React promote 잔여 — TNB shell · `/evaluate` QR jsQR 이식

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
| UI React Lab · SVG | §2.9 |
| Lab 에셋 sync | §2.10 |
| React promote 4화면 | §2.11 |
| Lab UI Kit · 거친 스케치 | §2.12 |
| 랜딩 user flow · SVG · CTA | §2.13 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| ML 백로그 | `artifact/plan/ml-vs-service-diversification-6month-backlog.md` |
| 환경 표기 | 로컬 prod-profile 회귀 |
| 전일 일지 | [2026-06-03-developer-log.md](2026-06-03-developer-log.md) |
