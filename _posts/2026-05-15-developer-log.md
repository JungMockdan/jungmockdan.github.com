---
title: "개발일지 — 2026-05-15"
excerpt: "약관·동의 NFR·제품 결정 확정(PR드 §10·§4.3·§6.2·§8·§9) — 플랜 D4·Phase C·보드 `TERMS-CONSENT-EPIC-01`; 코드 변경 없음."
date: 2026-05-15 12:00:00 +0900
last_modified_at: 2026-05-15 12:00:00 +0900
categories: [deVlog]
tags: [planet645, terms, consent, prd, nfr, board, artifact, 개발일지]
toc: true
toc_sticky: true
---

# 개발일지 — 2026-05-15

> [전일 2026-05-14 일지](/2026-05-14-developer-log)에서 약관 이행 plan 초안을 올렸다. 본 일지는 **제품·NFR 결정**을 PRD·이행 플랜·`board.xml`에 반영한 날짜로 정리한다. **애플리케이션 코드 변경은 없다.**

---

## 1. 오늘 목표

### Planet645

- [x] 약관·동의 **v1 NFR·제품 결정**을 PRD·플랜에 박제(D1·유예~배치 전 UX·감사 IP/UA·동시 기기·Redis 소스·운영 롤백 등)
- [x] PRD §9 Plan 행을 이행 플랜 파일 경로로 정합
- [x] `.claude/tasks/board.xml`에 약관 이행 **에픽** `TERMS-CONSENT-EPIC-01` 추가
- [ ] Phase A(G) 구현 — **후속 세션**

### 개발 운영

- [x] `_posts` 개발일지 본 문서 작성

---

## 2. 주요 변경 사항

### 2.1 Planet645 — PRD·플랜·보드 (문서만)

| 결정 요약 | 반영 위치 |
|-----------|-----------|
| `IMPLICIT`: 유예 만료 후 **배치 전까지 동의 row `N` 유지**(전면 차단 없음); 그 구간 **비차단** “개정 반영 처리 중” 등 **안내·배너** | PRD §4.3, 플랜 §2 D1 |
| 감사 이벤트에 **클라이언트 IP 전체·User-Agent 전문** 저장 | PRD §6.2, §8 수용 기준 9번, §10 |
| 감사 **무결성**: 앱 수준 **append-only**(+권한 분리); DB WORM 등은 v1 비요구 | PRD §10 |
| 동시 기기 동의 변경: **최종 작성자 우선**만(409·낙관적 락 UI 비요구) | PRD §10 |
| 가용성·성능·알림 채널: v1 **수치 목표 없음**, 관측 후 보강 | PRD §10 |
| Redis 게이트 소스: **1행 집계 테이블**(고시 커밋 시 갱신) SoT | 플랜 §2 D4, Phase C |
| Plan 행 링크 | PRD §9 → [`artifact/plan/terms-consent-implementation.md`](../../artifact/plan/terms-consent-implementation.md) |
| 보드 에픽 | `TERMS-CONSENT-EPIC-01` (DesignDocReference에 PRD·설계·플랜 링크) |

워크스페이스 루트 기준 수정 파일: `artifact/prd/terms-consent.md`, `artifact/plan/terms-consent-implementation.md`, `.claude/tasks/board.xml`.

### 2.2 개발 운영 문서

_(GitHub Pages·Jekyll 설정 변경 없음. `board.xml`의 `metadata/lastUpdated`만 갱신.)_

---

## 3. 문제와 해결

### Planet645

_(해당 없음 — 결정 수용 후 문서 역주입만 수행)_

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 질문지에 **번호로 답이 오면**(1-A, 9-a 등) PRD §10처럼 “질문 목록”을 **“v1 확정” 절로 갈아끼우는** 편이, 이후 구현·감사 점검 시 재질문을 막는다.
- 플랜의 **Phase C 소스**를 “쿼리 또는 테이블”로 두면 첫 PR에서 범위가 흔들리므로, **D4로 한 줄 고정**해 두는 것이 이행에 유리하다.

---

## 5. 다음 액션

### Planet645

- [ ] `TERMS-CONSENT-EPIC-01` 기준으로 Phase A(마이그레이션·`TERM-xxx`) 착수
- [ ] 설계 `01-schema-revision-guards.md` §10.4와 본문 용어(EXPLICIT/IMPLICIT vs PRD 표현) **역주입** 필요 여부 점검
- [ ] `terms_consent_event` 스키마·마이그레이션에 **IP·UA 컬럼** 반영(설계·DDL과 정합)

### 개발 운영

- [ ] 문서·보드 변경분 **워크스페이스 `main` 커밋·푸시**(별도 요청 시)

---

## 6. 기준

이 일지는 워크스페이스 `CLAUDE.md`의 개발일지 경로(`jungmockdan.github.com/_posts/`)와 Jekyll 프론트 매터 규칙을 따른다. 본문 Planet645 / 개발 운영 분리는 워크스페이스 `.cursor/rules/jekyll-devlog-github-pages.mdc`와 맞춘다.

---

## 7. SoT 링크 (당일 정합)

### Planet645

- **PRD:** [`artifact/prd/terms-consent.md`](../../artifact/prd/terms-consent.md) — §4.3 `IMPLICIT` 유예~배치 전, §6.2 IP·UA, **§10 v1 NFR 확정**, §9 Plan 경로.
- **이행 플랜:** [`artifact/plan/terms-consent-implementation.md`](../../artifact/plan/terms-consent-implementation.md) — §2 D1·D4·NFR 요약, Phase C 소스 확정, 보드 `TERMS-CONSENT-EPIC-01` 명시.
- **설계(참조):** [`artifact/design/terms-consent/01-schema-revision-guards.md`](../../artifact/design/terms-consent/01-schema-revision-guards.md)

### 개발 운영

- **보드:** 워크스페이스 `.claude/tasks/board.xml` — `TERMS-CONSENT-EPIC-01`.
