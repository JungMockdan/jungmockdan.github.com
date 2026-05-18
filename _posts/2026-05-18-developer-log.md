---
title: "개발일지 — TNB·내 정보 드로어 병합 및 선택 약관 사후 동의"
excerpt: "드로어·설정 허브 병합 후, 마이페이지에서 마케팅 미동의 시 수신 채널을 켤 수 없던 UX 공백을 optional-terms 플로우로 해소했다."
categories: [deVlog]
tags: [planet645, spring, thymeleaf, tnb, mypage, merge, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-18 12:00:00 +0900
last_modified_at: 2026-05-18 21:30:00 +0900
---

## 1. 오늘 목표

### Planet645

- [x] 대시보드 UI가 예전처럼 보이는 원인 조사(롤백 vs 미병합 브랜치)
- [x] `feature/tnb-myinfo-drawer-settings` → `main` 병합
- [x] 앱 기동 실패(`Ambiguous mapping` on `/mypage/profile`) 해결
- [x] `TERMS-OPTIONAL-CONSENT-MYPAGE-01` — 마이페이지 선택 약관(마케팅) 사후 동의 UX

### 개발 운영

- [x] 수동 merge 진행분 폐기 후 깨끗한 상태에서 재병합
- [ ] `main` 원격 push(필요 시)

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

### 2.5 개발 운영

_(해당 없음 — 오늘 작업은 저장소 병합·기동 수정 중심)_

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

### Planet645

- [ ] 로컬: 마케팅 미동의 → optional-terms 동의 → marketing-channels 채널 ON 스모크
- [ ] `PHASE17-02-FOLLOWUP-MKT-SEND-CHECK` — 동의+채널 조합 발송 가드 수동 점검
- [ ] `MYPAGE-DRAWER-01` 잔여(크레딧 상세 페이지 등)
- [ ] 로컬 prod-profile 회귀(`PHASE17-LOCAL-PROD-REGRESSION-01`)

### 개발 운영

- [x] `artifact/ops/tasks/board.md` — `TERMS-OPTIONAL-CONSENT-MYPAGE-01` → `review_pending`
- [ ] PRD `terms-consent.md` §4.5 가입 후 동의 경로 문서 delta

## 6. 기준

- 저장소: `com.mockdan.life-saver-lotto/` (Spring Boot, port 8080)
- 환경 정책: 스테이징 없음 — 회귀는 **로컬 prod-profile**
- dev-login: `local` + `app.dev-login.enabled=true`에서만 (`/dev/login?as=user|admin`)
