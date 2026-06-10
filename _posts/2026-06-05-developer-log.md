---
title: "개발일지 — 2026-06-05 (UI Kit design-guide-v03 · 먹색 수채화)"
excerpt: "디자인 컨셉 v03 변경 · 먹색 테두리 + 수채화 사선 붓칠 + Gaegu 폰트 · 포인트 컬러 체계"
categories: [deVlog]
tags: [planet645, ui-kit, design, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-05 17:31:04 +0900
last_modified_at: 2026-06-05 17:31:04 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 디자인 컨셉 v03로 변경 · 먹색(#36322b) 거친 테두리 + 수채화 사선 붓칠 + Gaegu 폰트 · 포인트 컬러(노랑·빨강·초록·파랑) · design-guide-v03.html 공개.

## 1. 오늘 목표

### Planet645 (UI Lab)

- [x] **디자인 컨셉 v03** — 먹색 테두리 + 수채화 손그림 + Gaegu · 포인트 컬러 체계

### 개발 운영

_(해당 없음)_

---

## 2. 주요 변경 사항

### 2.1 디자인 컨셉 v03 (UI Lab)

| 항목 | 값 |
|------|-----|
| Commit | `8cc376c` · 디자인컨셉변경 |
| 컨셉 | v02(거친 선·해칭) → **v03(먹색 + 수채화)** |
| 테두리 | #36322b(먹색) · `border-radius` 불규칙 |
| 채움 | 반투명 수채화 사선 붓칠(Gaegu 폰트의 곡선미·손그림감) |
| 폰트 | **Gaegu**(권게글꼴, Google Fonts) · 400, 700 weight |
| 배경 | #efe9da(톤 다운·손그린 종이감) |
| 포인트 컬러 | 노랑(주)·빨강(경고/강조)·초록·파랑(보조) |

#### 산출물

| 파일 | 역할 |
|------|------|
| `design/planet645-ui-kit/design-guide-v03.html` | 인터랙티브 가이드 · 색상·버튼·입력·카드·아이콘 미리보기 |
| `design/planet645-ui-kit/planet645-sketch.js` | 브러시/스케치 효과 생성기 · 사선 붓칠·불규칙 border-radius |
| `design/planet645-ui-kit/tokens.css` | CSS 변수 · 색상·간격·반경·효과 |

#### 디자인 특징

- **먹색 테두리:** 모든 UI 컴포넌트(버튼·카드·입력)는 `border: 1.5px solid #36322b` + 불규칙 border-radius(`18px 14px 20px 16px / 16px 20px 14px 18px` 등)로 손그린 느낌
- **수채화 붓칠:** CSS 변수 `--muk`(먹색), `--paper`, `--cream`으로 반투명 배경 · 사선 무늬 또는 다층 해칭(JavaScript로 생성)
- **Gaegu 폰트:** 손글씨 감성 · 헤더·버튼·라벨 전역 적용
- **포인트:** 제한된 컬러 팔레트 · 노랑 강조(CTA), 빨강(에러/경고), 초록·파랑(정보)

### 2.2 design-guide-v03.html 구조

| 섹션 | 내용 |
|------|------|
| 색상·토큰 | 먹색, 종이색(#fffef9), 크림(#fdfbf4), 보조색(#6b665c) · 포인트 컬러 4종 |
| 버튼 | primary, secondary, outline 예시 · `border-radius` 불규칙성 · 그림자(4px 5px, 투명도 12%) |
| 카드 | `.stage` 클래스 · 크림 배경 · 먹색 테두리 · 패딩 24px 26px |
| 텍스트 | h1(34px), h2(21px), body(Gaegu) 계층 |
| 입력 | 텍스트 필드·checkbox 예시 · 먹색 테두리 일관성 |

---

## 3. 문제와 해결

_(해당 없음)_

---

## 4. 배운 점

- 거친 손그림 톤은 **단색 테두리(먹색) + 불규칙 border-radius + 수채화 배경**의 3층이 필요 · flat 디자인만으로는 표현 불가
- Gaegu 손글씨 폰트는 **헤더·라벨·버튼 전역**에 적용하면 일관된 톤이 나옴 · 본문 텍스트는 보조 폰트와 혼합 고려
- 포인트 컬러 제한(4색)은 오히려 명확한 **정보 계층**(주 액션·경고·정보·보조)을 만듦

---

## 5. 다음 액션

### 5.1 미완

#### Planet645 (UI Lab)

- [ ] design-guide-v03.html · React 컴포넌트 동기화 (포인트 컬러 CSS·Gaegu 폰트 로드)
- [ ] 랜딩·결과·대시보드 화면에 v03 스타일 반영 (promote 진행 중 통합)

#### 개발 운영

_(해당 없음)_

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| 디자인 컨셉 v03 | §2.1 · §2.2 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| UI Lab | `com.mockdan.planet645-ui-lab/` (자체 Git) |
| 환경 표기 | 로컬 개발(design-guide-v03.html 정적 미리보기) |
| 전일 일지 | [2026-06-04-developer-log.md](2026-06-04-developer-log.md) |
