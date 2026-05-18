---
title: "개발일지 — TNB·내 정보 드로어 병합 및 선택 약관 사후 동의"
excerpt: "드로어 병합·선택 약관 사후 동의에 이어, /profile/setup 만 18세 이하 decline·PRD/design 역정합·수동 verify 가이드 handoff까지 정리했다."
categories: [deVlog]
tags: [planet645, spring, thymeleaf, tnb, mypage, merge, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-18 12:00:00 +0900
last_modified_at: 2026-05-19 12:00:00 +0900
---

## 1. 오늘 목표

### Planet645

- [x] 대시보드 UI가 예전처럼 보이는 원인 조사(롤백 vs 미병합 브랜치)
- [x] `feature/tnb-myinfo-drawer-settings` → `main` 병합
- [x] 앱 기동 실패(`Ambiguous mapping` on `/mypage/profile`) 해결
- [x] `TERMS-OPTIONAL-CONSENT-MYPAGE-01` — 마이페이지 선택 약관(마케팅) 사후 동의 UX
- [x] `AUTH-LOGOUT-ZERO-01` — 로그아웃 제로베이스 PRD 확정(Q1–Q8) 및 평가 보완 메모
- [x] `TERMS-MINOR-AGE-DECLINE-01` — implement + PRD/design/plan 역정합
- [ ] `TERMS-MINOR-AGE-DECLINE-01` — 수동 verify ([가이드](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md))

### 개발 운영

- [x] 수동 merge 진행분 폐기 후 깨끗한 상태에서 재병합
- [ ] `main` 원격 push(필요 시) — §5.3 개발 운영

## 2. 주요 변경 사항

### 2.1 UI — TNB·내 정보 드로어 (Planet645)

`main`에는 텍스트 `로그인`·대시보드 「알림」 버튼이 남아 있었고, 의도한 UI는 **미병합 브랜치** `feature/tnb-myinfo-drawer-settings`에만 있었다.

병합 후 반영된 UX:

| 영역 | 변경 |
|------|------|
| TNB | 프로필 아이콘(내 정보 드로어) + 벨 아이콘(메시지함), 브랜드 `플래넷645` |
| 대시보드 헤더 | 「알림」 텍스트 버튼 제거, 「새 추천」만 유지 |
| `/mypage` | 구 `index.html` 대신 **설정 메뉴 허브** (`settings-menu.html`) |
| 드로어 | 크레딧·플랜 뱃지, 프로필 수정·메시지함·설정 바로가기 |

관련 파일 예: `fragments/tnb.html`, `static/css/design-system.css`, `static/js/main.js`, `ui/AuthenticatedLayoutModelAdvice.java`, `ui/MypageController.java`.

### 2.2 병합 전략 — 백엔드·DB (Planet645)

- **UI·드로어·dev-login**: feature 브랜치 쪽 채택
- **약관 가드·Phase 2.2 billing·만 19세 정책**: `main` 유지 (`TermsConsentGuardInterceptor`, `TermsConsentService` 등)
- **Flyway**: V1 baseline에 V25–V27이 이미 포함되어 있어, feature의 `V25`–`V27`·삭제된 `V16` 마이그레이션 파일은 **제거**(중복 적용 방지)
- feature와 `main`에 겹치던 terms/dev-login **중복 클래스** 정리(동일 테이블 JPA 엔티티 이중 등록 방지)

병합 커밋: `f268887` — `Merge branch 'feature/tnb-myinfo-drawer-settings' into main`

### 2.3 프로필 수정 라우트 중복 제거 (Planet645)

기동 시 오류:

```text
Ambiguous mapping: POST [/mypage/profile]
  profileEditController#saveProfile  vs  mypageController#saveProfile
```

- **원인**: 병합으로 `ProfileEditController`(main, `ProfileEditRequest`)와 `MypageController`(feature, `ProfileSetupRequest`)가 동일 경로에 매핑
- **해결**: `ProfileEditController` 삭제. 템플릿 `content/mypage/profile.html`은 이미 `profileSetupRequest` + `fragments/profile-setup-fields`를 사용 → `MypageController` 단일 소유
- `MypageController.saveProfile`에 `MinimumAgeNotMetException` 처리 추가
- 테스트: `ProfileEditControllerTest`를 `MypageController` 프로필 엔드포인트 기준으로 수정

후속 PR: `feat/terms-consent-profile-guard-2026-05-18` (`32b627c`) — 약관 프로필 가드·테스트 보강 포함.

### 2.4 선택 약관 사후 동의 — `TERMS-OPTIONAL-CONSENT-MYPAGE-01` (Planet645)

**배경:** 가입 시 TERM-004(마케팅) 미선택 사용자는 `/mypage/settings/marketing-channels`에서 채널 UI가 비활성화만 되고, 동의로 이어지는 전용 진입이 없었다.

**구현 요약:**

| 항목 | 내용 |
|------|------|
| 라우트 | `GET/POST /mypage/settings/optional-terms` (`/terms-agreement` 재동의와 역할 분리) |
| 서비스 | `TermsConsentService.findOptionalTermsPendingConsent`, `applyOptionalTermConsents` — 필수·기존 동의 행은 유지, 선택 약관만 upsert |
| 이벤트 | `TermsConsentSource.MARKETING_SETTINGS` |
| IA | 설정 허브(`mypage-settings-menu`)·TNB 드로어·마케팅 수신 화면 CTA 「선택 약관 동의하기」 |
| 검증 | `TermsConsentServiceTest`, `ProfileEditControllerTest`, E2E 미인증 가드 |

**플로우:** 마케팅 수신 수단(미동의) → 선택 약관 동의 → 동의 저장 → 수신 수단에서 채널 ON.

### 2.6 만 18세 이하 decline — `TERMS-MINOR-AGE-DECLINE-01` (Planet645)

**배경:** `/profile/setup`은 생년월일 `AgePolicy` 검증만 있고, 「만 18세 이하」 자진 신고·소셜 정리 UX가 없었다.

**제품 결정:** 재로그인 허용 · `/profile/setup`만 · 안내 `login?info=minor_ineligible`.

**구현 요약:**

| 항목 | 내용 |
|------|------|
| API | `POST /profile/setup/decline-minor` — `MinorAgeDeclineService`, 세션 정리, `SecurityContextLogoutHandler` |
| UI | `setup.html` — 「만 19세 이상입니다」+ `ProfileAgeGuard` + 「만 18세 이하입니다」(`window.confirm`) |
| 로그인 | `login.html` — `info=minor_ineligible` info 배너 |
| 감사 | `IdentityAuditLog.MINOR_AGE_DECLINED` (identity 있을 때) |
| 테스트 | `ProfileSetupControllerTest`, `MinorAgeDeclineServiceImplTest` |

**문서 역정합:** `terms-consent.md` §3.1 C · `02-future-minor-age-declaration-flow.md` · plan §9 Phase H · 보드 `review_pending`.

**Handoff:** 수동 verify SoT — [`artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md`](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md) (다른 세션에서 §3.2–3.4 실행).

### 2.5 로그아웃 제로베이스 — `AUTH-LOGOUT-ZERO-01` (Planet645)

**배경:** as-built는 드로어 GET `/logout`, 재동의 POST, Spring `logoutSuccessUrl("/")` 고정 등이 혼재. 상위 PRD의 `POST /api/v1/auth/logout`은 미구현.

**절차:** 결정 질문(Q1–Q8) 선행 → [`artifact/prd/auth-logout.md`](../../artifact/prd/auth-logout.md) 확정 → PRD 평가 → §14 보완 메모.

**확정 결정 요약:**

| ID | 결정 |
|----|------|
| Q1 | SSR + `POST /api/v1/auth/logout` |
| Q2 | 현재 브라우저 세션만 |
| Q3 | POST + CSRF |
| Q4 | 즉시 로그아웃(확인 모달 없음) |
| Q5 | 공개 Referer면 복귀, else `/` |
| Q6 | v1 진입: 드로어 푸터만 |
| Q7 | `IdentityAuditLog` `LOGOUT` 필수 |
| Q8 | v1 세션만; OAuth revoke는 design 검토만 |

**PRD §14 design 필수 메모 (D-MEMO):** Referer UX·CSRF 전역 disable 정합·reconsent v1 회귀 유지·allowlist 동기화·identity 주체. **implement 전 D-MEMO-1·2·4 Closed.**

**보드:** `artifact/ops/tasks/board.md` — `prd` 페이즈 done, `design` planned.

### 2.6 개발 운영

_(해당 없음 — §2.5 문서 작업)_

## 3. 문제와 해결

### 3.1 대시보드 UI가 “롤백된 것 같다”

| | |
|---|---|
| **문제** | TNB에 로그인 텍스트·대시보드 「알림」 버튼이 다시 보임 |
| **원인** | 롤백이 아니라 `feature/tnb-myinfo-drawer-settings`가 **PR 없이 main에 미병합** |
| **해결** | 브랜치 확인 후 `main`에 merge, 충돌 해결 및 중복 마이그레이션·terms 스택 정리 |

### 3.2 수동 merge 중단 후 재작업

| | |
|---|---|
| **문제** | 로컬 `main`에 미완료 merge·수정 파일 잔존 |
| **원인** | 사용자 수동 병합 시도와 자동 merge 산출물 혼재 |
| **해결** | `git reset --hard origin/main` → feature 브랜치 **재 merge** |

### 3.3 ApplicationContext 기동 실패

| | |
|---|---|
| **문제** | `securityFilterChain` 생성 중 `Ambiguous mapping` on `POST /mypage/profile` |
| **원인** | `ProfileEditController` vs `MypageController` 이중 매핑 |
| **해결** | `ProfileEditController` 제거, `MypageController`만 유지 |

### 개발 운영

_(해당 없음)_

## 4. 배운 점

- UI 회귀로 보이는 현상은 **git 롤백**뿐 아니라 **장기 미병합 feature 브랜치**일 수 있다 — `main` vs `feature/*` diff를 먼저 본다.
- Flyway V1 squash 이후 feature 브랜치의 `V25`–`V27` 파일은 **baseline과 중복**될 수 있어 merge 시 제거해야 한다.
- Thymeleaf 폼 객체(`profileSetupRequest` vs `profileEditRequest`)와 컨트롤러를 맞춘 뒤, **중복 `@RequestMapping`은 기동 전에** 잡아야 한다(Spring이 Security 설정 단계에서 mapping introspection으로 실패).

## 5. 다음 액션

SoT: [`artifact/ops/tasks/board.md`](../../artifact/ops/tasks/board.md) (Active 없음 · Backlog 8건, `AUTH-LOGOUT-ZERO-01` 포함). 완료 태스크: [`archive-history-index.md`](../../artifact/ops/tasks/archive-history-index.md).

### 5.1 오늘 완료·아카이브 (태스크 ID)

| Task ID | 상태 | 비고 |
|---------|------|------|
| `TERMS-OPTIONAL-CONSENT-MYPAGE-01` | archived | 선택 약관 사후 동의 · Spring PR #7 |
| `TERMS-OPTIONAL-CONSENT-WITHDRAW-01` | archived | 동의·철회 단일 화면 · 2026-05-19 |
| `NOTIF-MKT-SEND-GUARD-01` | archived | 마케팅 발송 가드 · Spring PR #10 |
| `PHASE17-02-FOLLOWUP-MKT-SEND-CHECK` | archived | dev 훅 시나리오 1–4 수동 verify |
| `NOTIF-CONSENT-BOUNDARY-01` | archived | 약관·알림 경계 PRD/design/plan |

TNB·드로어 병합 본체는 에픽 `MYPAGE-DRAWER-01`(PRD) 기준이며, 보드에는 **후속** ID로 분리되어 있다.

### 5.2 잔여 업무 — Planet645 (board Backlog)

보드 `executionOrder`는 유지. 아래 **착수 순**은 **선행 태스크 없음**을 전제로, 범위·as-built 재사용·외부 연동·설계 미확정 항목을 기준으로 **처리하기 쉬운 것부터** 정렬했다.

| 착수 순 | Task ID | board Order | 우선순위 | 요약 | 쉬운 이유(요지) |
|--------|---------|-------------|----------|------|----------------|
| 0 | [`AUTH-LOGOUT-ZERO-01`](../../artifact/ops/tasks/board.md) | 200 | **high** | 로그아웃 제로베이스 · PRD ✅ → design/plan | PRD·Q1–Q8 확정 · §14 D-MEMO design 선행 |
| 1 | [`TERMS-MINOR-AGE-DECLINE-01`](../../artifact/ops/tasks/board.md) | 230 | medium | 「만 18세 이하」decline · **verify 대기** | implement·문서 **done** · [수동 verify](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md) |
| 2 | [`MYPAGE-CREDIT-PAGE-01`](../../artifact/ops/tasks/board.md) | 210 | medium | 드로어 크레딧 → **상세·이력 SSR** | balance/history API·드로어 잔액 **as-built** · 페이지+링크 1곳 집중 |
| 3 | [`MYPAGE-MYINFO-NAV-01`](../../artifact/ops/tasks/board.md) | 220 | medium | 내 정보 **허브·서브nav** | `mypage-settings-menu` **패턴 복제** 가능 · 대상 화면·라우트 **다수**로 2번보다 넓음 |
| 4 | [`PROFILE-ADDRESS-SEARCH-01`](../../artifact/ops/tasks/board.md) | 260 | low | 프로필 주소검색 API 연동 | 공급자·CSP·키·마이그레이션 **선택지** design 선행 |
| 5 | [`PHASE22-BILLING-SANDBOX-01`](../../artifact/ops/tasks/board.md) | 240 | medium | Toss Payments Sandbox | 실키·웹훅·서명·프로파일 분기 · PG 연동 |
| 6 | [`PHASE22-BILLING-REFUND-01`](../../artifact/ops/tasks/board.md) | 250 | low | 환불·영수증 API·UI | MVP **비범위** 후속 · API·웹훅·UI 면적 큼 |
| 7 | [`IDENTITY-CI-FREE-ABUSE-01`](../../artifact/ops/tasks/board.md) | 270 | low | CI 앵커·무료 혜택 1인 1회 | PASS/NICE·법무·Entitlement 교차 · 범위 최대 |

**착수 큐 (선행 무시):**  
`AUTH-LOGOUT-ZERO-01`(design) → `TERMS-MINOR-AGE-DECLINE-01` → `MYPAGE-CREDIT-PAGE-01` → `MYPAGE-MYINFO-NAV-01` → `PROFILE-ADDRESS-SEARCH-01` → `PHASE22-BILLING-SANDBOX-01` → `PHASE22-BILLING-REFUND-01` → `IDENTITY-CI-FREE-ABUSE-01`

> 보드 Agent notes의 선행(예: MYINFO↔CREDIT, Sandbox 키)은 **착수 순 결정에 반영하지 않음**. 실제 착수 시 키·라우트 확정이 막히면 해당 항목만 건너뛰고 다음 순번으로 진행.

### 5.3 잔여 업무 — verify·문서 (보드 외)

Backlog보다 **먼저** 끝내기 쉬운 항목(코드 변경 없음·짧은 회귀). §5.2 착수 **이전** 또는 병렬로 처리.

| 착수 순 | 항목 | 연관 Task ID | 메모 |
|--------|------|----------------|------|
| 0a | [ ] PRD delta: `terms-consent.md` §4.5·§6.1 | `TERMS-OPTIONAL-CONSENT-MYPAGE-01` | 문서만 · 아카이브 design 잔여 |
| 0b | [ ] 로컬 스모크: optional-terms → marketing-channels ON | `TERMS-OPTIONAL-CONSENT-MYPAGE-01` | 구현 완료 · 수동 1회 |
| 0c | [ ] 로컬 **prod-profile** 회귀 | `PHASE17-LOCAL-PROD-REGRESSION-01` | plan/CLAUDE SoT · board 미등재 |
| 0d | [ ] `TERMS-MINOR-AGE-DECLINE-01` 수동 verify | 동일 | [`TERMS-MINOR-AGE-DECLINE-01-manual-verify.md`](../../artifact/ops/verify/TERMS-MINOR-AGE-DECLINE-01-manual-verify.md) §3.2–3.4 |

### Planet645 (추가)

- [ ] `AUTH-LOGOUT-ZERO-01` — `artifact/design/user-identity/auth-logout-flow.md` (§14 D-MEMO-1~5 반영)
- [ ] `artifact/plan/auth-logout-implementation.md`

### 개발 운영

- [x] `artifact/ops/tasks/board.md` — `TERMS-OPTIONAL-CONSENT-MYPAGE-01` 등 당일 완료 태스크 아카이브 이관
- [x] `artifact/prd/auth-logout.md` — PRD 확정 + §14 평가 보완 메모
- [ ] PRD `terms-consent.md` §4.5 delta (§5.3 표 참조)
- [ ] `main` 원격 push(필요 시) — §1 개발 운영

## 6. 기준

- 저장소: `com.mockdan.life-saver-lotto/` (Spring Boot, port 8080)
- 환경 정책: 스테이징 없음 — 회귀는 **로컬 prod-profile**
- dev-login: `local` + `app.dev-login.enabled=true`에서만 (`/dev/login?as=user|admin`)
