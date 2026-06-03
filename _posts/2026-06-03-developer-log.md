---
title: "개발일지 — 2026-06-03 (문자 시드 B · select 인라인)"
excerpt: "문자 시드 B — 인라인 미리보기·좌/우클릭 칩·금칙·POST PRG(URL 무 phrase) · verify PASS."
categories: [deVlog]
tags: [planet645, recommend, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-03 08:29:37 +0900
last_modified_at: 2026-06-03 21:34:33 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 문자 시드 **범위 B** — select step1 인라인 미리보기, 좌클릭 고정/우클릭 제외 칩, 금칙·로그 마스킹, **POST+PRG**로 URL에 문장 미노출. preview 템플릿 제거·verify PASS.

## 1. 오늘 목표

### Planet645

- [x] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · select 인라인 · 고정/제외 칩

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

---

## 3. 문제와 해결

### Planet645

_(해당 없음)_

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 범위 A 스펙 1차 UI( select 인라인 )가 B에서야 완성됐다 — 독립 URL은 redirect만 남기는 편이 UX·유지보수에 유리하다.
- 시드 6개 vs 고정 최대 5 → **전체 자동 populate**보다 칩 선택 UI가 제약과 맞는다.
- URL 노출은 GET query보다 **POST+세션 flash**가 맞다 — 접근 로그 마스킹과 병행.

---

## 5. 다음 액션

### 5.1 미완

#### 개발 운영

**2026-06-02 이월**

- [ ] (후속) OWASP stage 2 — `failBuildOnCVSS=7` (Netty 42582 등 잔여 HIGH 정리 후)

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| `FEAT-TEXT-SEED-PREVIEW-B-01` — 인라인 · 칩 UX · 금칙 · POST PRG | §2.1–§2.2 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| 환경 표기 | 로컬 prod-profile 회귀 (staging 표기 없음) |
| 전일 일지 | [2026-06-02-developer-log.md](2026-06-02-developer-log.md) |
| text-seed SoT | `artifact/design/recommendation-ml/text-seed-base45-preview-spec.md` §11·§12 |
