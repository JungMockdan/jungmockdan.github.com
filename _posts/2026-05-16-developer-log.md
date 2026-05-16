---
title: "개발일지 — 2026-05-16 (Billing Sub B·C·WSL 기동·TERMS·E2E)"
excerpt: "Phase 2.2 Billing 문서·Sub B/C IMPLEMENT(completion handler·웹훅·Mock·PRO_MONTHLY). WSL Docker/nvm·Flyway V16/V26·Security SSR·TERMS view. gradle 345·스모크 37·E2E 23/3."
date: 2026-05-16 12:00:00 +0900
last_modified_at: 2026-05-17 14:30:00 +0900
categories: [deVlog]
tags: [planet645, billing, webhook, prd, design, artifact, commercialization, playwright, e2e, flyway, hibernate, docker, wsl, nvm, security, terms-consent, pro-monthly, completion-handler, 개발일지]
toc: true
toc_sticky: true
---

> [이전 2026-05-15 일지](/2026-05-15-developer-log)에서 TERMS-CONSENT v1·E2E 25/1을 마친 뒤, **오전** Phase 2.2 Billing 문서 정리, **저녁** WSL 기동·TERMS/E2E 회귀, **심야** Billing Sub B·C 1차 IMPLEMENT까지 진행했다.

---

## 1. 오늘 목표

### Planet645 — 문서 (오전)

- [x] as-built 대비 Phase 2.2 PRD·PLAN·DESIGN delta(FR-11, `userId` UUID, 미처리 웹훅, earn 멱등 키, 플랜 SoT 등)
- [x] `billing-integration-spec.md`·V3 PRD·`as-built/billing-entitlement.md` 정합
- [x] V3 단일 PRD → `artifact/prd/commercialization/` 분할(안 A) + 구 파일 스텁

### Planet645 + 개발 운영 — WSL·기동·E2E (저녁)

- [x] WSL Docker `permission denied` → `wsl --shutdown` 후 `docker` 그룹 반영
- [x] WSL Node **nvm**(Windows npm CRLF 회피)·Playwright **ubuntu24.04** 플랫폼 우회
- [x] 루트 `.env`(더미 OAuth)·Flyway **V16** 실패 이력 복구·**SecurityConfig** SSR 302
- [x] **`GET /terms/view`** — TERMS-CONSENT 스모크·E2E green
- [x] Flyway **V26** `user_terms_agreement` 시스템 버저닝(MariaDB 4124 회피)
- [x] 기동 시 `t_win_numbers` N+1 제거(warm-up·`JOIN FETCH`)
- [x] 루트 `playwright.config.ts`·`global-setup` 302 버그 수정
- [x] `./startup-and-smoke-test.sh --keep-running` — API **37/37**, Playwright **23 passed / 3 skipped**

### Planet645 — Billing IMPLEMENT (심야)

- [x] Sub B·C 1차: `BillingPaymentCompletionService`·웹훅·Mock checkout·`PRO_MONTHLY`·gradle **345 passed**
- [x] 스모크 6건 추가(`smoke/spec/smoke-tests.json`)

### 개발 운영

- [x] `artifact/ops/tasks/board.md` — `PHASE22-BILLING-MVP-01` Agent notes·Sub B/C 체크리스트
- [x] 본 개발일지

---

## 2. 주요 변경 사항

### 2.1 Phase 2.2 문서 — delta·PLAN·DESIGN·V3 분할

as-built와 어긋나던 항목을 PRD·PLAN·DESIGN에 명시했다.

| 주제 | 문서 반영 요약 |
|------|----------------|
| 웹훅 서명 | secret 설정 시 `X-Toss-Signature` 누락·불일치 **모두 400** — as-built는 헤더 없이 200 가능 → 2.2 수정 대상 |
| 결제 성공 | `WebhookService`가 `activatePaidSubscription` 미호출 → completion handler 단일화(DESIGN §10·Sub B) |
| 멱등 | ledger `EARN:SUBSCRIPTION:{orderId}`, `earn` 오버로드 권장 |
| 플랜·가격 | `PRO_MONTHLY`·9900 KRW SoT vs `SubscribeController` 혼재 — PLAN §5·DESIGN §6.1.1 |
| 스모크 ID | `BILLING-WH-S4-SIG-MISS`, `BILLING-WH-UNKNOWN-USER` 등 PLAN §6.2 |

**경로**

| 층 | 파일 |
|----|------|
| Sprint 8 PRD delta | [`phase22-billing-mvp-prd-delta.md`](../../artifact/prd/phase22-billing-mvp-prd-delta.md) |
| 시나리오 PLAN | [`phase22-billing-mvp-scenarios.md`](../../artifact/plan/phase22-billing-mvp-scenarios.md) |
| DESIGN delta | [`phase22-billing-mvp-design-delta.md`](../../artifact/design/billing-subscription/phase22-billing-mvp-design-delta.md) |
| 결제 통합 | [`billing-integration-spec.md`](../../artifact/design/billing-subscription/billing-integration-spec.md) |
| V3 프로그램 | [`commercialization/README.md`](../../artifact/prd/commercialization/README.md) |

- **`billing-integration-spec.md`**: §4.2 FR-10/11 분기, §7 `POST /webhook/toss/payment`
- **V3 `commercialization/` 분할(안 A)**: `v3-overview.md`, `v3-sprint-roadmap.md`, `identity-membership-prd.md`, `billing-subscription-entitlement-prd.md` — 구 [`membership-billing-subscription-plan.md`](../../artifact/prd/membership-billing-subscription-plan.md)는 deprecated 스텁만 유지
- **「V3」**: 2026-04-06 PRD **문서 세대 라벨**(Sprint 7–9 상용화 묶음). API/릴리스 버전·Flyway `V3~V7`과 혼동 주의
- `DOCS_INDEX`·`project-roadmap`·`board`·`as-built/billing-entitlement` 링크 갱신

### 2.2 WSL·Flyway·Security·약관 뷰

| 증상 | 원인 | 조치 |
|------|------|------|
| `docker.sock` permission denied | `docker` 그룹 미반영 | `wsl --shutdown` 후 재접속 |
| `env: bash\r: No such file` | Windows Node shim CRLF | **nvm** Linux Node |
| Playwright ubuntu26 미지원 | Ubuntu 26.04 WSL | `PLAYWRIGHT_HOST_PLATFORM_OVERRIDE=ubuntu24.04-x64` |
| Compose `.env not found` | `spring-app.env_file` | 루트 `.env`(더미 OAuth) |
| Flyway V16 실패 마이그레이션 | V11과 중복 `ADD INDEX`·`success=0` 잔존 | V16을 **컬럼만**으로 축소·실패 행 삭제 후 재기동 |
| SSR 경로 **401** (스모크 31/37) | `LoginUrlAuthenticationEntryPoint` 미등록 경로 | `ssrLoginRedirectMatcher()`로 `/dashboard` 등 → **302 `/login`** |
| `GET /terms/view` **404** | 설계만 있고 미구현 | 공개 약관 페이지(`permitAll`) |

- **Flyway V26**: `system_time` 분리 ALTER → `ADD PERIOD FOR SYSTEM_TIME` + `ADD SYSTEM VERSIONING` 단일 `ALTER`(MariaDB 4124). 이미 versioned면 skip
- **회귀(저녁)**: API 스모크 **37/37**; `npm run test:e2e` **23 passed / 3 skipped**

### 2.3 Spring — 기동 시 당첨 번호 N+1

- **증상**: `ApplicationReadyEvent` warm-up 시 `SELECT … FROM t_win_numbers WHERE base_id=?` 회차 수만큼 반복
- **조치** (`com.mockdan.life-saver-lotto`): `TWinBaseInfo.numbers` 역방향 매핑 제거, `TWinNumbersRepository`에 `join fetch n.baseInfo` `@Query`, `app.lotto.draw-statistics.warmup-on-startup` 설정·로그

### 2.4 Playwright E2E (워크스페이스)

| 항목 | 내용 |
|------|------|
| `playwright.config.ts` | `testDir: ./smoke/e2e`, `globalSetup`, `SPRING_BASE` 기본 18080 |
| `global-setup.ts` | dev-login **302**를 `res.ok()` 실패로 오인 → **`status < 400`** 으로 성공 판정, `auth.json`·`auth-reconsent.json` 생성 |
| `auth-storage.ts` | auth 파일 경로 워크스페이스 루트 기준 통일 |
| 스펙 | `.settings-list`, `약관 동의 내역` 등 현 UI 반영; TNB 드로어·dev-login `POST /onboarding`은 **의도 skip 3건** |

### 2.5 Billing Sub B·C IMPLEMENT

PLAN [`phase22-billing-mvp-scenarios.md`](../../artifact/plan/phase22-billing-mvp-scenarios.md) §5·§6.1 — 결제 성공 SoT = **`POST /webhook/toss/payment`** + Mock checkout §0.1 A+B.

| 컴포넌트 | 변경 |
|----------|------|
| `BillingPaymentCompletionService` | Payment → `activatePaidSubscription` → `earn` (`EARN:SUBSCRIPTION:{orderId}`) |
| `WebhookService` | 파싱·위임; `PRO_MONTHLY` 가드·unknown user → handler |
| `TossWebhookController` | secret 시 signature **400**; unknown user **400** |
| `TossMockBillingProviderAdapter` | redirect 직전 in-process completion (S6) |
| `SubscribeController` | `planId=pro` → `PRO_MONTHLY` 9900; `standard` alias |
| `EntitlementService` | `earn(..., idempotencyKey)` 오버로드 |
| `BillingController` | `POST …/payments/succeeded` **@Deprecated** |

| PLAN | 검증 |
|------|------|
| S1·S2·S3·S7 | `BillingPaymentCompletionServiceTest` |
| S4·S5·unknown user | `TossWebhookControllerTest` |
| S6 | `TossMockBillingProviderAdapterTest`·`SubscribeControllerTest` |
| 스모크 | `BILLING-WH-S3`·`S5`·`S7`·`UNKNOWN-USER` 6건 |

**회귀:** `./gradlew test` **345 passed**(1 skipped). 보드 `PHASE22-BILLING-MVP-01` Sub B·C 1차 체크, verify는 billing 스모크 S1/S2/S8·서명 S4 잔여.

**IMPLEMENT에서 고정한 규칙**

- 신규 플로우는 **웹훅 only**; 레거시 `payments/succeeded` deprecated
- S7: `planCode` ≠ `PRO_MONTHLY` → HTTP 200 `processed`, DB 무변경(PG 재시도 폭주 방지)
- 멱등: Payment `idempotency_key = orderId` + ledger 이중 방어

---

## 3. 문제와 해결

| 문제 | 원인 | 해결 |
|------|------|------|
| 단일 PRD 파일명이 Billing만 강조 | Identity·Entitlement·스프린트까지 한 파일에 혼재 | `commercialization/` 슬라이스 + API/DB는 design SoT로 위임 |
| `V3` 의미 불명확 | 문서 세대명 vs Flyway 버전 혼동 | Sprint 7–9 상용화 PRD 세대명으로 정의; delta(`phase22-*`)가 Sprint 8 구현 SoT |
| E2E **9 skipped** | `global-setup`이 dev-login **302**를 실패 처리 | `status < 400` 성공 판정 → `auth.json` 생성 |
| `playwright test` No tests found | 루트 config 없음 | `playwright.config.ts` 추가 |
| 기동 로그 Hibernate SQL flooding | warm-up N+1 | §2.3 bulk JOIN; 로컬 `logging.level.org.hibernate.SQL=INFO` |
| mypage E2E 실패 | 템플릿·TNB와 assertion 불일치 | 실제 UI 기준 갱신; 미구현은 skip 사유 명시 |
| `startup-and-smoke-test.sh` Permission denied | `scripts/*.sh` **644** | `find … -exec chmod 755 {} \;` (메인만 777로는 자식 스크립트 실패) |
| 스모크 billing S1/S2/S8 미검증 | 시드 user·webhook-secret 전제 | verify 잔여(§5) |

---

## 4. 배운 점

- **Phase 2.2**(로드맵 Phase)와 **V3**(PRD 묶음 이름)는 축이 다르다. 구현 SoT는 `phase22-*` delta다.
- 상용화 PRD는 얇게 두고, 계약·DDL은 `artifact/design/**`에 두는 편이 유지보수에 낫다.
- Playwright **`res.ok()`는 3xx를 성공으로 보지 않는다** — dev-login·OAuth는 status 범위를 명시할 것.
- `@EntityGraph`만으로 OneToOne bulk가 안 잡히면 **`join fetch` JPQL**이 더 확실하다.
- E2E auth 파일은 **워크스페이스 루트 절대 경로**로 `global-setup`과 스펙을 맞출 것.
- 웹훅 only + S7 무변경 200 + 멱등 이중 방어는 PG 재시도·중복 결제를 동시에 막는 패턴이다.

---

## 5. 다음 액션

### Planet645

- [ ] verify: `./startup-and-smoke-test.sh` billing 스모크 green — `BILLING-WH-S1-DONE`·`S2-DUP`·`S8-RENEW`(시드 `users.user_id`)
- [ ] 로컬 prod-profile: `billing.toss.webhook-secret` 설정 후 `BILLING-WH-S4-SIG-*`
- [ ] (선택) `commercialization/README.md`에 「V3 = 문서 세대명」 한 줄
- [ ] (선택) Playwright `BILLING-S6-SUBSCRIBE-ME`
- [ ] TNB 드로어·메시지함 UI 구현 시 `mypage-drawer.spec.ts` skip 해제
- [ ] dev-login 없이 온보딩 POST E2E — OAuth codegen 또는 SocialAccount 시드

### 개발 운영

- [ ] `./startup-and-smoke-test.sh` — `scripts/*.sh` **755**·Docker `docker` 그룹 확인 후 재실행
- [x] `./gradlew test` **345 passed**(2026-05-16)
- [x] API 스모크 **37/37**·Playwright **23/3**(billing 스모크 추가 전 저녁 회귀)

**회귀 명령**

```bash
cd ~/workspace/com.mockdan.life-saver-lotto-workspace
source ~/.bashrc   # nvm
find . -maxdepth 3 -name '*.sh' -type f -exec chmod 755 {} \;
./startup-and-smoke-test.sh --keep-running
npm run test:e2e
cd com.mockdan.life-saver-lotto && ./gradlew test
```

---

## 6. 기준

- **저장소 경계**: 워크스페이스 git = `artifact/`, `smoke/`, `jungmockdan.github.com/`; Spring·Flyway = `com.mockdan.life-saver-lotto/`(루트 `.gitignore`, 별도 repo).
- **Billing SoT**: [`phase22-billing-mvp-prd-delta.md`](../../artifact/prd/phase22-billing-mvp-prd-delta.md) · PLAN · DESIGN delta · [`billing-integration-spec.md`](../../artifact/design/billing-subscription/billing-integration-spec.md).
- **보드**: [`artifact/ops/tasks/board.md`](../../artifact/ops/tasks/board.md) — `PHASE22-BILLING-MVP-01`(Sub B·C 1차 완료, verify in-progress).
- **개발일지 경로**: 워크스페이스 `CLAUDE.md` — `jungmockdan.github.com/_posts/`, `YYYY-MM-DD-developer-log.md`.
