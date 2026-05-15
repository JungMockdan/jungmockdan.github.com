---
title: "개발일지 — 2026-05-15 (TERMS-CONSENT v1·FOLLOWUP·Flyway·보드·E2E)"
date: 2026-05-15 12:00:00 +0900
last_modified_at: 2026-05-15 23:45:00 +0900
categories: [deVlog]
tags: [planet645, spring, terms-consent, flyway, playwright, board, phase17, docker, redis]
excerpt: "TERMS-CONSENT v1·FOLLOWUP 완료(Spring temporal·재동의 E2E)·보드 MD SoT·스모크/E2E 스펙·문서 §8-10·Playwright 25/1·V16·V25 체크섬"
toc: true
toc_sticky: true
---

## 1. 오늘 목표

### Planet645

- [x] 약관·동의 에픽(`TERMS-CONSENT-EPIC-01`) 진행·v1 범위 정리.
- [x] Flyway V26 `user_terms_agreement` 시스템 버저닝 DDL 및 JPA 읽기 전용 매핑.
- [x] `artifact/plan/phase17-product-engineering-backlog.md` §G SoT 한 줄·변경 이력 반영.
- [x] (저녁) Docker 기준 `npm run test:e2e` **25 passed / 1 skipped** 회귀 및 원인(포트·Redis·동의 행) 코드 반영.
- [x] **`TERMS-CONSENT-FOLLOWUP-01`**: MariaDB `FOR SYSTEM_TIME ALL` 조회·REST·Flyway V27·재동의 Playwright·문서·보드 아카이브.
- [x] **Flyway V16**: `user_subscriptions`에 V11과 중복되던 `ADD INDEX` 제거(깨끗한 `compose down -v` 기동 시 Duplicate key 방지).

### 개발 운영

- [x] 태스크 보드 **XML → Markdown**: `artifact/ops/tasks/board.md`(·`archive-history.md`) SoT, `board.xml`은 레거시 도구용.
- [x] 보드 `metadata/lastUpdated`·`TERMS-CONSENT-EPIC-01` / **`TERMS-CONSENT-FOLLOWUP-01`** 아카이브·후속 백로그 분리.
- [x] 동일 일자 개발일지 본문 갱신(스테이징 묶음: PRD·설계·플랜·phase17 §G·`smoke/` 스펙 정합).

---

## 2. 주요 변경 사항

### 2.1 Planet645 — 약관·동의(TERMS-CONSENT)

- **Flyway V25**: `terms` 확장(`TERM-001` 등), `user_terms_agreement.term_key`·유니크, `terms_policy_revision`, `terms_consent_event`. `term_key` 단일 유니크 드롭은 `INFORMATION_SCHEMA` 기반 동적 `DROP INDEX`로 이식성 보강.
- **Flyway V26·이력 조회(FOLLOWUP)**: `user_terms_agreement`에 MariaDB `SYSTEM VERSIONING`(`row_start`/`row_end`). JPA는 **현재 행**만 단일 `@Id` + `row_start`/`row_end` 읽기 전용. **이력:** `UserTermsAgreementTemporalRepository` + `GET /api/v1/profile/terms-agreement-temporal?termKey=…`(MariaDB 외는 빈 배열).
- **Flyway V27**: `TERM-001` 두 번째 시행 행(E2E·`testuser-e2e-reconsent` stale `term_id` 시드).
- **앱**: 재동의 가드·`/terms/view`(공개)·`DevLoginSeeder` 시드 사용자 동의 행 동기화(E2E·dev-login과 충돌 방지).
- **회귀**: `./startup-and-smoke-test.sh --build` 기준 API 스모크 **31/31**; Playwright는 재동의 스펙 추가 전 스모크 러너 **24 passed / 1 skipped** → 단독 `npm run test:e2e` **25 passed / 1 skipped**(§2.4).
- **문서(FOLLOWUP 클로저)**: PRD **§8 수용 기준 10**(본인 temporal API); 설계 **§3.4** V26·JPA·`UserTermsAgreementTemporalRepository` 이행 문구; `terms-consent-implementation.md` 상태 **v1 이행 완료**·§6·변경 이력; phase17 **§G** SoT·§H 이력(스테이징·커밋 대상).

### 2.2 개발 운영 — 보드·백로그

- **보드 SoT**: `artifact/ops/tasks/board.md` — `TERMS-CONSENT-EPIC-01`·**`TERMS-CONSENT-FOLLOWUP-01`** 모두 **Archived**(`lastUpdated` 2026-05-15T16:45:00Z). `Backlog`에는 `PHASE17-02-FOLLOWUP-MKT-SEND-CHECK` 등만 유지.
- **이관**: `.claude/tasks/board.xml` 내용을 `artifact/ops/tasks/` MD로 집결(`convert_xml_to_md.py`); XML은 레거시 도구용으로만 유지.

### 2.3 Planet645 — Phase 17 통합 백로그

- **`artifact/plan/phase17-product-engineering-backlog.md` §G**: v1 SoT 한 줄(아카이브 에픽·이행 플랜·FOLLOWUP 연결). §H 변경 이력에 2026-05-15 행 추가.

### 2.4 Planet645 + 개발 운영 — Playwright E2E 보강(저녁 회귀)

- **`SPRING_BASE` 기본값**: `smoke/e2e/config.ts`·`global-setup.ts` 폴백을 `http://localhost:18080`으로 통일(`docker-compose` 호스트 포트 `SPRING_PORT`·`startup-and-smoke-test.sh`와 정합). 단독 `npm run test:e2e`가 8080으로 붙어 `ERR_CONNECTION_REFUSED` 나던 문제 제거.
- **재동의 POST 500**: Compose에 Redis 없이도 `TermsReconsentService`가 Lettuce 연결 실패 시 분산 락을 건너뛰도록 보강(재동의 E2E `terms-reconsent.spec.ts` green).
- **인증 E2E 안정화**: `GET /dev/e2e/sync-e2e-user-terms`(dev-login과 동일 활성 조건)로 `testuser-e2e-user`의 `user_terms_agreement`를 시행 약관 카탈로그에 맞춤; `global-setup`이 `dev/login?as=user` 직후 호출해 `auth.json` 저장 전에 동의 행을 재맞춤(`ddl-auto=update` 드리프트로 `/terms-agreement`·`/login`으로 빠지던 케이스 완화).
- **사용자 회귀**: Docker 재빌드·`up` 후 `npm run test:e2e` **25 passed / 1 skipped**(의도된 TNB 게스트 스펙 skip 유지).
- **문서**: `smoke/e2e/README.md` Base URL·`TERMS-CONSENT-FOLLOWUP-01` 스펙·`sync-e2e-user-terms`·`reset-reconsent-seed` 전제 정리.
- **재동의 E2E 시드**: `DevLoginSeeder.USER_ID_RECONSENT_PENDING`, `GET /dev/login?as=reconsent`, `GET /dev/e2e/reset-reconsent-seed`, `global-setup` → `auth-reconsent.json`, `terms-reconsent.spec.ts`.

### 2.5 Planet645 — 스모크·E2E 스펙(워크스페이스 git 스테이징)

워크스페이스 루트 git에는 `com.mockdan.life-saver-lotto/`가 `.gitignore`라 **Spring·Flyway 구현은 별도 저장소**에 두고, 아래는 FOLLOWUP 마감용 **스모크/E2E·문서** 묶음이다.

| 파일 | 내용 |
|------|------|
| `smoke/spec/smoke-tests.json` | `GET /terms/view` → 200 (`permitAll`) |
| `smoke/spec/e2e-tests.json`, `smoke/e2e/tests.json` | 공개 약관 `/terms/view`, `?termKey=TERM-001` 각 1건 |
| `smoke/e2e/terms-reconsent.spec.ts` | 대시보드 → `/terms-agreement` → 제출 → `/dashboard` |
| `smoke/e2e/global-setup.ts` | 기본 `SPRING_BASE` 18080, `as=reconsent`, `as=user` 직후 `sync-e2e-user-terms` |
| `smoke/e2e/README.md` | 재동의 E2E 전제·Base URL |
| `artifact/prd|design|plan/…`, `phase17-product-engineering-backlog.md` | §2.1 문서 클로저 |

### 2.6 Planet645 — Flyway V16 정리

- **`V16__add_auto_renew_to_user_subscriptions.sql`**: V11에서 이미 정의한 `idx_status`·`idx_expires_at`(및 `user_id` UNIQUE)와 겹치던 `ADD INDEX` 3줄 제거. 신규 DB 전체 마이그레이션 시 `Duplicate key name 'idx_status'` 방지.

---

## 3. 문제와 해결

| 문제 | 원인 | 해결 |
|------|------|------|
| Docker 스모크에서 `/terms/view` 404 | 호스트가 예전 Spring 이미지 사용 | `./startup-and-smoke-test.sh --build`로 Spring 이미지 재빌드 |
| Playwright 인증 스펙 다수 실패 | 새 가드로 dev-login 시드 사용자가 재동의 페이지에 머묾 | `DevLoginSeeder`에서 시행 중 약관에 맞춰 `user_terms_agreement` 재시드 |
| 단독 E2E가 8080으로 접속 실패 | Playwright 기본 `SPRING_BASE`가 compose 호스트 포트와 불일치 | `config.ts`·`global-setup` 기본을 **18080**으로 변경·`package.json` `test:e2e:local` 정합 |
| 재동의 제출 후 `/dashboard` 미도착(500) | Redis 빈은 있는데 호스트에 `6379` 없음 → 락 획득에서 예외 전파 | `TermsReconsentService`에서 Redis 예외 시 락 생략·429는 유지 |
| 인증 스펙 간헐 실패·`/login` | 로컬 `ddl-auto=update`로 시드 사용자 동의 행이 카탈로그와 어긋남 | `GET /dev/e2e/sync-e2e-user-terms` + global-setup 직후 호출 |
| 복합 PK `@IdClass` + `IDENTITY`/`@Generated` | Hibernate `CompositeValueGenerationException` | v1에서는 단일 `@Id` 유지 + V26 DDL·읽기 전용 temporal 컬럼 매핑으로 타협 |
| `docker compose down -v` 후 기동 시 V16 실패 | V11이 이미 `idx_status`/`idx_expires_at` 생성, V16이 동일 인덱스 재추가 | V16에서 중복 `ADD INDEX` 삭제, 컬럼 추가만 유지 |
| Flyway **V25 checksum mismatch** | DB에 적용된 V25 파일과 레포 V25 내용 불일치(예: `INFORMATION_SCHEMA` 기반 인덱스 드롭 보강 후) | 로컬은 볼륨 삭제 후 재마이그레이션 또는 `flyway repair` / 일시 `spring.flyway.repair-on-migrate=true` 후 제거; prod는 스키마 정합 확인 후 repair 여부 판단 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 약관 가드는 **로컬 E2E 시드 계정**까지 영향을 주므로, dev-login 시더와 동의 데이터를 함께 맞출 것.
- 스모크가 새 MVC 경로를 검증하려면 **Spring Docker 이미지 재빌드**가 필요할 수 있음(캐시된 JAR).
- MariaDB 시스템 버저닝은 DB PK가 `(id, row_end)`로 바뀌므로, **JPA PK 모델**은 Hibernate 제약과 `ddl-auto=validate`(향후 운영)까지 묶어 설계할 것.
- **선택 Redis**(락·캐시)는 “빈은 있는데 연결 불가”까지 고려해, 핵심 사용자 경로는 연결 실패 시에도 **DB 폴백**으로 끊기지 않게 둘 것.
- Playwright는 **호스트 포트·세션 파일**이 스크립트 회귀와 다르면 조용히 실패하므로, `SPRING_BASE` 기본값을 compose와 한줄에 맞추고 global-setup에서 **동의 행 재동기화** 한 번 더 거는 편이 안전하다.
- **이미 적용된 Flyway 스크립트를 고치면** 체크섬이 바뀐다. 개발 DB는 `repair`나 볼륨 초기화로 맞추고, 운영은 “DB 상태=새 스크립트 의도”가 확실할 때만 repair한다.
- **후속 마이그레이션(V16)**이 초기 DDL(V11)과 인덱스 이름을 **중복 정의**하면, 깨끗한 DB에서만 터지는 오류가 난다. 신규 `CREATE`와 `ALTER`를 같이 볼 것.

---

## 5. 다음 액션

### Planet645

- [x] Backlog **`TERMS-CONSENT-FOLLOWUP-01`**: UTA 이력 API·재동의 Playwright·문서 정합 — `artifact/ops/tasks/board.md` 아카이브·§2.4·§2.5 스펙 반영(2026-05-15).
- [ ] 마케팅 발송 수동 점검: Backlog **`PHASE17-02-FOLLOWUP-MKT-SEND-CHECK`**.

### 개발 운영

- [x] 보드 `artifact/ops/tasks/board.md` `lastUpdated` 및 `TERMS-CONSENT-FOLLOWUP-01` 아카이브·AgentNotes 정리.
- [ ] 원격/다중 개발자 환경에서 V16·V25 **체크섬 이미 적용 DB**가 있으면 `repair` 또는 운영 절차 문서 한 줄 보강 검토.

---

## 6. 기준

- 제품 SoT: `artifact/prd/terms-consent.md`(§8 수용 기준 10 포함), `artifact/design/terms-consent/01-schema-revision-guards.md`, `artifact/plan/terms-consent-implementation.md`.
- 보드 SoT: `artifact/ops/tasks/board.md` — **`TERMS-CONSENT-EPIC-01`**·**`TERMS-CONSENT-FOLLOWUP-01`** 모두 **Archived**(2026-05-15; FOLLOWUP: temporal API·재동의 E2E·문서·V27·`ProfileTermsTemporalApiTest`).
- Phase 17 통합: `artifact/plan/phase17-product-engineering-backlog.md` §G.
- **저장소 경계**: 워크스페이스 git = `artifact/`, `smoke/`, `jungmockdan.github.com/` 등; Spring 앱·Flyway = `com.mockdan.life-saver-lotto/`(루트 `.gitignore`, 별도 repo).
