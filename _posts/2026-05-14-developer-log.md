---
title: "개발일지 — 2026-05-14"
excerpt: "약관·동의 SoT(PRD·설계 v0.2.2·DOCS_INDEX) 일지 반영 + 이행 plan 초안(`artifact/plan/terms-consent-implementation.md`) — Phase A–G·검증 매트릭스."
date: 2026-05-14 12:00:00 +0900
last_modified_at: 2026-05-15 12:00:00 +0900
categories: [deVlog]
tags: [planet645, terms, consent, prd, design, artifact, 개발일지]
toc: true
toc_sticky: true
---

# 개발일지 — 2026-05-14

> [어제 2026-05-13 일지](/2026-05-13-developer-log)에서 마이페이지·스모크·Playwright 회귀를 정리했고, 본 일지 **§7**에 약관·동의 SoT(PRD·설계 v0.2.2·인덱스)를 모았다. 같은 주제의 **이행 계획(plan)** 은 `artifact/plan/terms-consent-implementation.md` 로 추가했다. 코드 변경은 없고 산출물은 문서다.

---

## 1. 오늘 목표

### Planet645

- [x] 약관·동의 SoT 문서 링크·요약을 일지 §7에 정리(전일 작업을 올바른 날짜 문서로 이관)
- [x] 약관·동의·개정 PRD·설계 SoT에 맞춘 **이행 plan** 초안 작성
- [ ] plan에 적은 Phase A(G) 실제 구현 — **후속 세션**

### 개발 운영

- [x] 개발일지(`_posts`) 갱신 — 본 문서

---

## 2. 주요 변경 사항

### 2.1 Planet645 — 약관 이행 plan 초안

워크스페이스 루트 기준 **`artifact/plan/terms-consent-implementation.md`** 를 추가했다. 상위 문서와의 대응은 아래와 같다.

| 산출물 | 경로 | 역할 |
|--------|------|------|
| PRD | `artifact/prd/terms-consent.md` | 항목·필수/선택·개정 구분값·수용 기준 |
| 설계 | `artifact/design/terms-consent/01-schema-revision-guards.md` | 스키마·가드·라우트·Redis 게이트·배치 |
| **Plan (신규)** | `artifact/plan/terms-consent-implementation.md` | Phase A–G(마이그레이션 → 도메인 → Redis → 라우트·UI → 재동의·배치), 테스트·수동 감사, 설계 §10.4 TBD에 대한 **권장 초안**(예: `IMPLICIT`는 배치 반영 전까지 동의 row 비갱신·전면 차단은 `EXPLICIT`만) |

보드에 별도 에픽을 둘지는 미정이며, plan 본문 §6에 `board.xml` 연동 권장만 적어 두었다. PRD §9 표의 Plan 행은 아직 파일 경로로 갱신하지 않았다(원하면 한 줄 링크 추가로 정합 가능).

### 2.2 개발 운영 문서

_(코드·보드·CLAUDE 변경 없음 — 일지만 추가)_

---

## 3. 문제와 해결

### Planet645

_(해당 없음 — 문서 작성만 수행)_

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- PRD·설계가 이미 SoT로 고정된 뒤에는, **plan** 에서 “설계 TBD를 구현 팀이 닫을 선택지”를 한 줄이라도 박아 두면 스프린트 착수 시 논의 비용이 줄어든다.
- 약관 개정은 **필수/선택이 아니라 `revision_consent_mode`(EXPLICIT / IMPLICIT)** 만 본다는 규칙이 설계·plan 모두에 관통하므로, 테스트 케이스도 그 축으로 매트릭스를 짜는 편이 안전하다.

---

## 5. 다음 액션

### Planet645

- [ ] `terms-consent-implementation.md` Phase A부터 순차 구현(Flyway/Liquibase·엔티티·가드)
- [ ] 설계 §10.4 **D1**(IMPLICIT 배치 전 가드 동작) 팀 결정 확정 후 설계에 역주입
- [ ] PRD §9 “Plan” 행에 `terms-consent-implementation.md` 링크 반영(선택)
- [ ] `.claude/tasks/board.xml`에 약관 이행 에픽 태스크 신설 또는 Phase17 G절 하위로 분해(선택)

### 개발 운영

- [ ] 없음(또는 다음 일지에서 회귀·보드 상태만 갱신)

---

## 6. 기준

이 일지는 워크스페이스 `CLAUDE.md`의 개발일지 경로 규칙(`jungmockdan.github.com/_posts/`, Jekyll 프론트 매터)을 따른다. 본문 Planet645 / 개발 운영 분리는 워크스페이스 `.cursor/rules/jekyll-devlog-github-pages.mdc`와 맞춘다.

---

## 7. 약관·동의 문서 (SoT)

### Planet645

- **PRD:** [`artifact/prd/terms-consent.md`](../../artifact/prd/terms-consent.md) — 개정 **구분값**(EXPLICIT/IMPLICIT), 마이페이지 **동의 조회 전용**, 재동의 **전용 페이지**, 약관 전문·보관 정책 등.
- **설계:** [`artifact/design/terms-consent/01-schema-revision-guards.md`](../../artifact/design/terms-consent/01-schema-revision-guards.md) **v0.2.2** — `term_id` FK, `terms` **`(term_key, term_version)` UNIQUE**, MariaDB **temporal** + JPA 엔티티 분리, 라우트 **`/terms/view`**(전문·공개)·**`/terms-agreements`**(동의 조회·인증)·**`/terms-agreement`**(재동의), **고시 DB 조회 → Redis** 게이트, 배치·동시 로그인·중복 POST 등. 약관 **본문·목록·E(제3자) 행 수**는 **운영 데이터/mock**으로 두고 개발 블로커에서 제외.
- **인덱스:** [`artifact/DOCS_INDEX.md`](../../artifact/DOCS_INDEX.md)에 PRD·설계 링크 반영.

### 개발 운영

- **코드 변경:** 없음(문서만). 이행·Redis TTL·plan 본문은 §2.1 및 후속 세션.
