---
title: "개발일지 — 2026-05-13"
excerpt: "MYPAGE-DRAWER-01 후속 — 스모크·E2E 회귀 + 로컬 전용 dev-login 백도어로 Playwright 인증 자동화. OAuth Provider 모킹(Option D)은 별도 작업으로 보류."
date: 2026-05-13 12:00:00 +0900
last_modified_at: 2026-05-13 15:10:00 +0900
categories: [deVlog]
tags: [planet645, smoke, e2e, playwright, security, 개발일지]
toc: true
toc_sticky: true
---

# 개발일지 — 2026-05-13

> [어제 2026-05-12 일지](/2026-05-12-developer-log)의 "5. 다음 액션 / Planet645"에서 미처리였던 **스모크·회귀 확인**을 진행했다. MYPAGE-DRAWER-01(TNB·내 정보 드로어·설정 허브·약관·크레딧)에서 추가된 라우트가 **인증 가드(302 → /login)**를 지키는지, 그리고 인증 세션이 있을 때 **허브·약관·드로어**가 렌더링되는지 자동화 회귀에 반영했다.

---

## 1. 오늘 목표

### Planet645

- [x] MYPAGE-DRAWER-01 라우트의 인증 가드 회귀를 `smoke/spec/smoke-tests.json`에 반영
- [x] MYPAGE-DRAWER-01 UI 회귀(드로어·허브·약관)를 Playwright 스펙으로 추가
- [x] `npx playwright test --list`로 새 스펙이 수집되는지 검증
- [x] Playwright 인증 화면 회귀 자동화 — **Option B**(로컬 전용 dev-login 백도어 + `globalSetup`) 구현
- [ ] Option D(OAuth provider WireMock 모킹) — 별도 작업으로 **보류**

### 개발 운영

- [x] 기존 스모크 인프라(`startup-and-smoke-test.sh`, `smoke_runner.py`) 변경 없이 스펙만 확장
- [x] auth 세션이 없는 환경에서도 회귀가 동작하도록 `auth.json` 옵셔널 패턴 유지
- [x] 보안 정책 게이트(G1) 승인 후 백도어 도입 — 운영 절대 차단(이중 가드)
- [x] PR 후 회귀 절차를 보드·Phase17 백로그에 반영(`PHASE17-LOCAL-PROD-REGRESSION-01`, §C.5 로컬 prod-profile)
- [x] '스테이징' 표현 정리 — 이 프로젝트는 staging 환경이 없음(1인 개발, 로컬→운영 2단계). 워크스페이스 `CLAUDE.md`·`AGENTS.md`에 환경 정책 명문화
- [x] Windows + WSL2 Hyper-V 동적 포트 예약 우회 — `docker-compose.yml` 호스트 포트 기본값 `58080` → `${SPRING_PORT:-18080}`
- [x] `/dashboard` 미인증 시 401 → 302 정정 — `SecurityConfig`에 `LoginUrlAuthenticationEntryPoint("/login")` 매처 추가
- [x] Playwright E2E 6건 회귀 — stale 4건 surgical 수정, 구조 불일치 1건 skip-with-reason, 실 회귀 1건 후속 태스크로 분리
- [x] `SECURITY-ENTRYPOINT-SSR-ROUTES-01` 마무리 — 잔여 SSR 4건(`/onboarding/**`, `/profile/setup/**`, `/subscribe/payment`, `/subscribe/success`)에 `LoginUrlAuthenticationEntryPoint("/login")` 매처 + 스모크 회귀 케이스 4건 추가. `./startup-and-smoke-test.sh --build` 결과 API 30/30 green (신규 4건 모두 302 PASS)
- [x] `PHASE17-LOCAL-PROD-REGRESSION-01` 1차 시도 — docker compose + `docker-compose.override.yml`(`SPRING_PROFILES_ACTIVE=prod`) 경로로 진행. Step 0(check-dev-login-guard.sh) green. Step 1 기동 시 `ddl-auto=validate` 가 엔티티 ↔ DB 컬럼 타입 미스매치 3건 연속 노출 → blocked 처리, 후속 `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` 신설
- [x] 보드 정합 — 백로그를 **소스 기준**으로 재검증해서 `MYPAGE-DRAWER-CREDIT-SUMMARY-01`(코드 단일 소스 `EntitlementService.getBalance()`로 이미 확정)·`SECURITY-ENTRYPOINT-SSR-ROUTES-01`(전 단계 마무리)을 `ArchivedTasks`로 이관. Playwright 환경 가드는 `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` 신규 백로그로 분리
- [x] `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` 구현·아카이브 — Cursor 샌드박스에 chromium 1217(약 285MB) 설치 + `scripts/check-playwright-browser.sh` 신설 + `startup-and-smoke-test.sh` pre-flight 연결. cache hit 7ms, cache miss 시 자동 install. 가드 반영 후 사용자 측 `./startup-and-smoke-test.sh --build` 재실행 — **API 스모크·Playwright 포함 전부 green** 확인
- [x] 보드 PHASE17 회귀 1차 시도 결과 반영 — `PHASE17-LOCAL-PROD-REGRESSION-01` `planned → blocked` + AgentNotes 회고 4줄, 후속 `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01`(executionOrder=109) 신설(discover/implement/verify 3-Phase, Done 체크리스트 3개, AgentNotes 3개), `metadata/lastUpdated` 갱신

## 2. 주요 변경 사항

### 2.1 API 스모크 — 인증 가드 6건 추가

MypageController(`/mypage`, `/mypage/settings/{terms,notifications,marketing-channels}`, `/mypage/profile`)와 NotificationController(`/notifications`)는 모두 `CurrentUserIdResolver.requireAppUserId(auth)`를 호출해 **미인증 시 302 → /login**으로 리다이렉트한다. 이 가드가 깨지지 않는지 매 배포에서 자동으로 잡히도록 스펙에 추가했다.

| 항목 | 내용 |
|---|---|
| 스펙 파일 | `smoke/spec/smoke-tests.json` |
| 새 케이스 | 6건 (`/mypage`, `/mypage/profile`, `/mypage/settings/terms`, `/mypage/settings/notifications`, `/mypage/settings/marketing-channels`, `/notifications`) |
| 기대 상태 | `302` |
| 리다이렉트 추적 | `"allowRedirect": false` — `smoke_runner.py`의 `_NoRedirectHandler` 사용 |
| 총 케이스 수 | 20 → 26 |

관련 파일:

- [`smoke-tests.json`][smokeTestsJson]
- [`smoke_runner.py`][smokeRunnerPy]
- [`MypageController`][mypageController]
- [`NotificationController`][notificationController]

---

### 2.2 E2E 회귀 — `mypage-drawer.spec.ts` 신설

기존 `phase15-dashboard.spec.ts`의 구성을 그대로 따랐다(미인증 redirect 회귀 + `auth.json` 옵셔널 가드). Playwright는 `playwright.config.ts`의 `testDir: './smoke/e2e'`로 자동 디스커버리되므로 등록 작업은 필요 없다.

| describe 블록 | 케이스 수 | 의도 |
|---|---|---|
| MYPAGE-DRAWER-01 인증 가드 (브라우저) | 6 | 미인증 상태에서 6개 라우트가 `/login` 또는 `/oauth2`로 빠지는지 URL 매칭 |
| MYPAGE-DRAWER-01 TNB 게스트 영역 (미인증) | 1 | `/login`에서 `.tnb-container` 노출, `.tnb-auth-guest .btn-login` 보임, `#tnbMypageDrawerOpen`은 **없음** |
| MYPAGE-DRAWER-01 내 정보 허브·드로어 (인증 세션 필요) | 3 | `auth.json` 있을 때만 실행 — `/mypage` 허브(`.settings-menu-page`), `/mypage/settings/terms`(`.terms-wrap`), TNB 드로어 토글(`#tnbMypageDrawerOpen` → `#mypageDrawerPanel` 노출) |

총 10건. `npx playwright test --list` 결과 25개 테스트 중 새로 추가된 10건이 정상 수집됐다.

관련 파일:

- [`mypage-drawer.spec.ts`][mypageDrawerSpec]
- [`phase15-dashboard.spec.ts`][phase15DashboardSpec]
- [`playwright.config.ts`][playwrightConfig]
- [`tnb.html`][tnbHtml]
- [`default_layout.html`][defaultLayoutHtml]
- [`settings-menu.html`][settingsMenuHtml]
- [`terms.html`][termsHtml]

---

### 2.3 Playwright 인증 자동화 — Option B (로컬 dev-login 백도어)

`mypage-drawer.spec.ts` / `phase15-dashboard.spec.ts` / `google-login.spec.ts`의 **인증 세션 필요** 케이스가 `auth.json` 부재로 모두 `test.skip()`되던 상태였다. 4가지 후보(A 실제 Google 자동화 / B dev 백도어 / C `@WithMockUser` / D OAuth provider 모킹)를 Feasibility Gate로 정리한 뒤 **Option B**를 채택해 구현했다.

| 구성 요소 | 위치 | 역할 |
|---|---|---|
| 컨트롤러 | `cmm/security/devlogin/DevLoginController` | `GET /dev/login?as=user\|admin` — `OAuth2AuthenticationToken`을 직접 만들어 세션에 주입(`HttpSessionSecurityContextRepository`) |
| 시드 | `cmm/security/devlogin/DevLoginSeeder` | `ApplicationRunner` — `testuser-e2e-user` / `testuser-e2e-admin` 두 명의 `User`·`Identity`를 idempotent하게 부트스트랩 |
| 이중 가드 | 두 클래스 동일 | `@Profile("!prod")` **AND** `@ConditionalOnProperty(name="app.dev-login.enabled", havingValue="true")` |
| 기본값 | `application.properties` | `app.dev-login.enabled=false` — 안전한 기본값 |
| 로컬 활성 | `application-local.properties` | `app.dev-login.enabled=true` — 로컬 전용 |
| Playwright | `smoke/e2e/global-setup.ts` | 테스트 실행 전 `/dev/login?as=user` → `auth.json`, `/dev/login?as=admin` → `auth-admin.json` 자동 생성. Spring 미기동 시 경고만 출력하고 통과 |
| 설정 연결 | `playwright.config.ts` | `globalSetup: './smoke/e2e/global-setup.ts'` 추가 |

세션 주입 핵심 로직(`DevLoginController.login`):

```java
OAuth2User delegate = new DefaultOAuth2User(authorities, attributes, "sub");
AuthenticatedSiteUser principal = new AuthenticatedSiteUser(delegate, userId);
OAuth2AuthenticationToken authToken = new OAuth2AuthenticationToken(
    principal, authorities, "dev-local"
);
SecurityContext context = SecurityContextHolder.createEmptyContext();
context.setAuthentication(authToken);
SECURITY_CONTEXT_REPO.saveContext(context, request, response);
```

`AuthenticatedSiteUser`를 그대로 사용하므로 `CurrentUserIdResolver.requireAppUserId(auth)`가 실제 OAuth 로그인과 동일하게 `appUserId`를 해석한다. 도메인 코드는 본 변경에 무관하다.

검증:

- `./gradlew compileJava` 통과
- `./gradlew compileTestJava` 통과 (기존 테스트 0 영향)
- `npx playwright test --list` 25 tests / 4 files — `globalSetup` 등록 후에도 디스커버리 정상

관련 파일:

- [`DevLoginController`][devLoginController]
- [`DevLoginSeeder`][devLoginSeeder]
- [`AuthenticatedSiteUser`][authenticatedSiteUser]
- [`CurrentUserIdResolver`][currentUserIdResolver]
- [`SecurityConfig`][securityConfig]
- [`application.properties`][appProps]
- [`application-local.properties`][appLocalProps]
- [`global-setup.ts`][globalSetupTs]
- [`playwright.config.ts`][playwrightConfig]

---

### 2.4 Option D 보류 — OAuth Provider 모킹

Option D(WireMock으로 Google `/.well-known/openid-configuration` · JWKS · token endpoint · ID 토큰 서명까지 모킹)는 **실제 OAuth 인증 흐름 자체**(`SiteOAuth2UserService` · `SiteOidcUserService` · `OAuth2LoginSuccessHandler` · `SocialUserProfileExtractor`)를 회귀할 유일한 자동화 경로다. 본 작업의 Option B는 그 흐름을 **우회**하므로 OAuth 흐름의 회귀 공백은 남는다.

비용·우선순위 차이로 본 세션에서는 보류하고 별도 작업으로 분리한다.

| 항목 | 내용 |
|---|---|
| 회귀 가치 | OAuth 사용자 서비스·성공 핸들러·프로파일 추출까지 포함 |
| 추가 작업 | WireMock 정적 응답·서명 키 페어·테스트 프로파일 설정·DB 시드 격리 |
| 의존성 | WireMock(또는 동등) 의존 추가 + CI 시간 증가 |
| 트리거 | OAuth 흐름 자체에 변경이 생기는 시점에 우선 착수 |

---

### 2.5 `startup-and-smoke-test.sh` 변경 없음

스모크 러너는 스펙(`smoke-tests.json`)을 인자로 받고, Playwright는 `testDir` 기반으로 새 스펙을 자동 수집하므로 셸 스크립트는 손대지 않았다. 다음 실행 순서·플래그도 그대로다.

```bash
./startup-and-smoke-test.sh             # Docker, 26개 API + 25개 E2E
./startup-and-smoke-test.sh --build     # 이미지 재빌드
./startup-and-smoke-test.sh --keep-running
PLAYWRIGHT_STORAGE_STATE=auth.json ./startup-and-smoke-test.sh  # 인증 세션 케이스까지
```

관련 파일:

- [`startup-and-smoke-test.sh`][startupSmokeSh]

---

### 2.6 보드 — PR 후 로컬 prod-profile 회귀 태스크 (+ '스테이징' 표현 정리)

2026-05-12 일지의 「스테이징 점검 태스크를 보드 정책에 맞게 반영」을 처리하다가 이 프로젝트에 **별도 staging 환경이 없다**는 사실을 확인했다(1인 개발, `application-local.properties` / `application-prod.properties` 2종, `docker-compose.yml` 단일, `.github/workflows`에 deploy 없음, `application-staging.properties` 0건). 아카이브 태스크 `PHASE17-01-FOLLOWUP-STAGING-CHECK`의 '스테이징'도 사실상 **로컬 prod-profile**을 가리켰던 회고를 AgentNotes에 남겼다.

이에 새 백로그 태스크를 `PHASE17-LOCAL-PROD-REGRESSION-01`(executionOrder=105, priority=high)로 정렬했다. 아카이브된 verify-only 패턴을 그대로 따르되 단계 수가 늘었다:

| Step | 의도 |
|---|---|
| 1. `./scripts/check-dev-login-guard.sh` | dev-login 토글이 `application-local.properties` 외에 없는지 사전 가드 |
| 2. `SPRING_PROFILES_ACTIVE=prod ./gradlew bootRun` | 로컬에서 운영 프로파일로 기동 |
| 3. `curl … /dev/login?as=user` → **404** | `@Profile("!prod")` 가드 자체가 검증 대상 |
| 4. API 스모크 | 26건 자동 |
| 5. Playwright | 미인증 가드 자동 green; 인증 spec은 의도된 skip |
| 6. 수동 Google 로그인 | TNB·드로어·허브·약관·알림 |
| 7. 증거 첨부 | 리포트·HTML·캡처를 PR/이슈에 |

prod-profile에서는 `DevLoginController`가 미등록되어 Playwright `globalSetup`이 dev-login 404를 받고 경고 후 skip하는 것이 **정상**이다(보안 가드 정합 자체가 회귀 대상). 인증 UI 자동 회귀가 필요한 일상 작업은 `local` 프로파일에서 그대로 동작한다.

부가로 **워크스페이스 환경 정책 한 줄을 명문화**했다: `CLAUDE.md`·`AGENTS.md`(심링크) 워크플로 절에 "로컬→운영 2단계만, '스테이징' 표현 미사용" 규칙 추가. Phase 17 백로그 상단과 §C.4·§C.5, `saturday-draw-night-pipeline.md` 장기 보완 절, `roadmap.xml` Phase 17 DoneCriteria의 단어도 같은 기준으로 정리했다.

관련 파일:

- [`board.xml`][boardXml]
- [`phase17-product-engineering-backlog.md`][phase17Backlog]
- [`saturday-draw-night-pipeline.md`][saturdayPipelineMd]
- [`roadmap.xml`][roadmapXml]
- [`CLAUDE.md`][CLAUDE]

---

### 2.7 오후 보강 — Docker 포트 회피 + `/dashboard` EntryPoint 보정

스모크 통합 실행 중에 발견한 두 건의 즉시 수정. 둘 다 surgical 한 줄/한 블록 변경.

**1) Docker 호스트 포트 18080으로 하향 (`docker-compose.yml`, `startup-and-smoke-test.sh`)**

`./startup-and-smoke-test.sh` 실행 시 Spring 컨테이너 기동 단계에서 다음 에러로 막혔다.

```text
Error response from daemon: ports are not available: exposing port TCP 0.0.0.0:58080 -> 127.0.0.1:0:
listen tcp 0.0.0.0:58080: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
```

`An attempt was made to access a socket in a way forbidden by its access permissions`는 Windows 전용 소켓 에러 코드 **WSAEACCES(10013)**. WSL 내부에서 `ss -tln`으로 확인했지만 58080은 비어 있었다. 원인은 Windows의 **Hyper-V/`winnat` 동적 포트 예약** — 기본 동적 포트 범위(49152–65535)의 일부 구간이 Hyper-V에 미리 잡혀 있고, Docker Desktop이 그 구간의 호스트 포트(58080)를 바인딩하려다 거부된다. `mariadb:3306`, `ml-service:8000`, `llm-service:8001`은 49152 미만이라 영향 없음.

| 항목 | Before | After |
|---|---|---|
| `docker-compose.yml` (spring-app `ports`) | `"58080:8080"` | `"${SPRING_PORT:-18080}:8080"` |
| `startup-and-smoke-test.sh` line 8 | `SPRING_PORT="${SPRING_PORT:-58080}"` | `SPRING_PORT="${SPRING_PORT:-18080}"` |

- 컨테이너 내부 포트(8080)는 그대로 — Spring 설정·Docker 내부 DNS·앱 코드 무영향.
- 두 파일이 동일한 `SPRING_PORT` 환경변수를 읽으므로 `.env`에 `SPRING_PORT=...` 한 줄로 통일 오버라이드 가능.
- `.github/workflows/e2e-smoke.yml`의 `SPRING_PORT: '54808'`은 Linux 러너라 위 예약 이슈가 없으므로 그대로 둠.

검증: `./startup-and-smoke-test.sh` 통과 — 4개 컨테이너 모두 health 정상, Spring app `up`.

**2) `/dashboard` 미인증 시 401 → 302 정정 (`SecurityConfig.java`)**

위 회귀 통과 직후 한 건 실패가 남았다.

```json
{
  "label": "PHASE15 Spring GET /dashboard (unauthenticated -> 302 redirect to login)",
  "expectedStatus": 302,
  "actualStatus": 401,
  "detail": "expected=302 actual=401"
}
```

`SecurityConfig.authorizeHttpRequests`에서 `/dashboard/**`는 `.authenticated()`로 보호되지만, `exceptionHandling`의 `defaultAuthenticationEntryPointFor` 매처 목록에 등재되지 않은 게 원인. 다른 SSR 경로(`/recommend/**`, `/notifications/**`, `/mypage/**`)는 `LoginUrlAuthenticationEntryPoint("/login")`로 명시 매핑되어 있고, `/api/v1/**`는 `HttpStatusEntryPoint(401)`로 명시 매핑되어 있다. `/dashboard/**`는 둘 다 매칭 실패 → `DelegatingAuthenticationEntryPoint`가 첫 등록 항목(`/api/v1/**` 쪽 401)으로 폴백한 것.

수정: 기존 매처 블록 뒤에 한 블록 추가.

```java
.defaultAuthenticationEntryPointFor(
    new LoginUrlAuthenticationEntryPoint("/login"),
    new AntPathRequestMatcher("/dashboard/**")
)
```

검증: `./startup-and-smoke-test.sh` 재실행 시 `total=26 passed=26 failed=0`(예상). 다른 라인 무변경.

관련 파일:

- [`docker-compose.yml`][dockerComposeYml]
- [`startup-and-smoke-test.sh`][startupSmokeSh]
- [`SecurityConfig`][securityConfig]

---

### 2.8 Playwright E2E 6건 회귀 — 분류·정리

`./startup-and-smoke-test.sh` 통합 실행 후 API 26/26 green, **E2E 19 passed / 6 failed**. 한 건씩 증거 모은 뒤 4종으로 분류해 처리.

| # | 케이스 | 분류 | 처리 |
|---|---|---|---|
| 1 | `google-login.spec.ts:18` 로그인 페이지 title | **A. 브랜드 리네이밍 stale** | 정규식 `Life Saver` → `Planet ?645`. `auth_layout.html:8`/`default_layout.html:9` 모두 `${pageTitle} ?: 'Planet 645'` 명시 — child `<title>로그인</title>`은 layout-dialect 머지에서 parent `th:text` 갱신에 덮임 |
| 2 | `google-login.spec.ts:78` 온보딩 POST → `/dashboard` | **D. 실 회귀** | 보드 `DEVLOGIN-ONBOARDING-FIX-01`로 분리. 본 절 §3.5 참고 |
| 3 | `mypage-drawer.spec.ts:52` 미인증 TNB | **B. 구조 불일치** | `test.skip()` + 사유 — `.tnb-auth-guest`는 `default_layout`에만 포함되고 공개 페이지는 모두 `auth_layout` 사용 → 미도달 코드 |
| 4 | `runner.spec.ts Landing page loads` title | **A. 브랜드 리네이밍 stale** | `tests.json:9` `assertTitleContains: "Life Saver"` → `"Planet ?645"` |
| 5 | `runner.spec.ts Recommend select page loads` | **C. tests.json 경계 위반** | `/recommend/**`는 `.authenticated()` → 미인증 302 → `/login`(form/button/.container 모두 없음). 일지 §3.1 패턴(인증 라우트는 tests.json에 두지 않음) 위배. `tests.json`에서 제거 |
| 6 | `runner.spec.ts Recommend result page loads` | **C. tests.json 경계 위반** | 동일. `[.result, .numbers, [th\:each]]` 셀렉터 자체도 Thymeleaf 속성을 가정한 잘못된 패턴. `tests.json`에서 제거 |

**구조 증거 (B 처리 근거)**

| 레이아웃 | TNB 포함 | 사용 페이지 |
|---|---|---|
| `default_layout.html` | ✅ (`fragments/tnb` include) | 인증 필요한 콘텐츠 페이지 16종 (`dashboard`, `mypage/**`, `recommend/**`, `subscribe/**`, `notifications/list`, `onboarding/features` 등) |
| `auth_layout.html` | ❌ | 공개 페이지 3종 (`/`, `/login`, `/profile/setup`) |

→ `tnb.html:115` `<div class="tnb-auth-guest" sec:authorize="!isAuthenticated()">`은 현재 어떤 라우트에서도 렌더링되지 않음. 테스트 의도(공개 페이지의 TNB 게스트 영역 회귀)는 유효하나 대상 페이지가 부재 — `test.skip()`로 보존해 향후 공개 `default_layout` 페이지 도입 시 재활성화.

관련 파일:

- [`tests.json`][e2eTestsJson]
- [`google-login.spec.ts`][googleLoginSpec]
- [`mypage-drawer.spec.ts`][mypageDrawerSpec]
- [`auth_layout.html`][authLayoutHtml]
- [`default_layout.html`][defaultLayoutHtml]
- [`tnb.html`][tnbHtml]

---

### 2.9 `SECURITY-ENTRYPOINT-SSR-ROUTES-01` 마무리 — 잔여 SSR 4건 + 보드 정합

§2.7에서 `/dashboard/**` 매처만 임시로 추가하고 동일 패턴의 잔여 SSR 라우트 4건은 보드 후속(`SECURITY-ENTRYPOINT-SSR-ROUTES-01`)으로 분리했었다. 사용자가 "개발일지 기준 말고 **소스 기준**으로 백로그 상태를 다시 확인하라"고 지시 — `SecurityConfig.java`·`smoke/spec/smoke-tests.json`·`templates/content/**`를 직접 SoT로 두고 대조하니 두 건의 어긋남을 발견했다.

**소스 vs 보드 대조 결과**

| Task | 보드 status (전) | 소스 사실 | 처리 |
|---|---|---|---|
| `MYPAGE-DRAWER-CREDIT-SUMMARY-01` | planned | `AuthenticatedLayoutModelAdvice:52-60` 드로어 모델·`CreditController:21-26` 잔액 API 둘 다 **`EntitlementService.getBalance(userId)`** 호출 → 코드 레벨 단일 소스 이미 확정 | **Archived 이관** |
| `SECURITY-ENTRYPOINT-SSR-ROUTES-01` | planned (전 Step) | `/dashboard/**`만 매처+스모크 반영, **잔여 SSR 4건** 미반영 | `in-progress`로 갱신 후 잔여 구현 → Archived |

**잔여 4건 구현**

`SecurityConfig.java` line 80 이후 `defaultAuthenticationEntryPointFor` 블록 4개 추가:

```java
.defaultAuthenticationEntryPointFor(
    new LoginUrlAuthenticationEntryPoint("/login"),
    new AntPathRequestMatcher("/onboarding/**")
)
.defaultAuthenticationEntryPointFor(
    new LoginUrlAuthenticationEntryPoint("/login"),
    new AntPathRequestMatcher("/profile/setup/**")
)
.defaultAuthenticationEntryPointFor(
    new LoginUrlAuthenticationEntryPoint("/login"),
    new AntPathRequestMatcher("/subscribe/payment")
)
.defaultAuthenticationEntryPointFor(
    new LoginUrlAuthenticationEntryPoint("/login"),
    new AntPathRequestMatcher("/subscribe/success")
)
```

→ `SecurityConfig`의 `.authenticated()` 9개 패턴이 전부 EntryPoint 매처에 명시 등재됨(API 1개 + SSR 8개).

**스모크 스펙 회귀 케이스 4건 추가** (`smoke/spec/smoke-tests.json`)

| URL | expectedStatus | allowRedirect |
|---|---|---|
| `/onboarding/features` | 302 | false |
| `/profile/setup` | 302 | false |
| `/subscribe/payment` | 302 | false |
| `/subscribe/success` | 302 | false |

**회귀 결과** (`./startup-and-smoke-test.sh --build`, 2026-05-13 13:50 KST)

| 영역 | 결과 |
|---|---|
| API 스모크 | **30/30 green** — 신규 4건 모두 302 PASS (`smoke/artifacts/startup-smoke-report.json`) |
| Playwright | 23/23 fail — **환경 이슈**(Cursor 샌드박스 chromium 미설치, `browserType.launch: Executable doesn't exist`). 본 변경 무관 → `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` 신규 backlog로 분리 |

**보드 변경 요약**

| Task | Active/Backlog → Archived | 비고 |
|---|---|---|
| `MYPAGE-DRAWER-CREDIT-SUMMARY-01` | Backlog → Archived | `completedAt=2026-05-11T10:00:00Z`(MYPAGE-DRAWER-01 완료 시점) |
| `SECURITY-ENTRYPOINT-SSR-ROUTES-01` | Backlog → Active → Archived | `completedAt=2026-05-13T04:50:00Z` |
| `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` | (신규) → Backlog | `startup-and-smoke-test.sh` pre-flight에 chromium 자동 설치 가드 추가 — 후속 |

관련 파일:

- [`SecurityConfig`][securityConfig]
- [`smoke-tests.json`][smokeTestsJson]
- [`AuthenticatedLayoutModelAdvice`][authenticatedLayoutModelAdvice]
- [`CreditController`][creditController]
- [`board.xml`][boardXml]

---

### 2.10 `PHASE17-LOCAL-PROD-REGRESSION-01` 1차 시도 — 회귀 가치 입증·blocked 처리

오후에 plan ([phase17_local_prod_regression_guide][phase17PlanMd]) 대로 **로컬 prod-profile 회귀**를 step-by-step 실행. 결과: 회귀 절차의 가치 자체는 즉시 입증됐으나(운영 부팅 차단 종류의 미스매치를 PR 머지 전 발견), Step 1에서 부팅이 막혀 본 태스크는 blocked 처리하고 후속 태스크로 분리했다.

**기동 경로 — `docker-compose.override.yml` 한 줄**

기존 [`docker-compose.yml`][dockerComposeYml] line 76 `SPRING_PROFILES_ACTIVE: local`을 직접 편집하지 않고, 워크스페이스 루트에 임시 override 파일을 만들어 compose 자동 머지에 위임:

```yaml
services:
  spring-app:
    environment:
      SPRING_PROFILES_ACTIVE: prod
```

`docker compose config | grep SPRING_PROFILES_ACTIVE` 로 머지 결과 확인(`prod`) 후 `docker compose up -d --build`. 컨테이너는 모두 healthy, Spring 로그도 `The following 1 profile is active: "prod"` 까지 정상.

**진행 결과**

| Step | 결과 |
|---|---|
| 0. `check-dev-login-guard.sh` | exit 0 — `application-local.properties`에만 `app.dev-login.enabled=true` |
| 1. docker compose up (prod override) | 컨테이너 기동 완료, prod profile active, **그러나 EntityManagerFactory 초기화에서 schema-validation 에러로 부팅 실패** |
| 2~6 | Step 1 차단으로 진행 불가 |

**1차 수정 (로컬 미커밋, 다음 PR로 분리 예정)**

`TWinBaseInfo.winAmountPerEach`의 Java 타입 `BigInteger`는 JPA 기본 매핑에서 SQL Types `NUMERIC(decimal(38,0))`이 되는데 DB 컬럼은 `bigint(20)`. 의도가 명확히 `bigint`였던 만큼 Java 타입을 `Long`으로 정렬하는 것이 의미적 변화 없는 정합 수정:

| 파일 | 변경 |
|---|---|
| [`TWinBaseInfo`][TWinBaseInfo] | `BigInteger winAmountPerEach` → `Long`, `columnDefinition = "int default 0"` / `"bigint default 0"` 2줄 제거(DB DEFAULT는 V2 migration이 이미 보장), orphan import 정리 |
| [`LotteryWinCrawler`][LotteryWinCrawler] | `base.setWinAmountPerEach(BigInteger.ZERO)` → `0L`, orphan import 정리 |
| [`LifeSaverLottoApplication`][lifeSaverLottoApplicationJava] | `parseBigIntegerCell` → `parseLongCell` (`new BigInteger(...)` → `Long.valueOf(...)`), 호출/`mergeBaseInfo` 시그니처 정리, orphan import 정리 |

검증: `./gradlew compileJava compileTestJava` 5s green, `./gradlew test --tests com.mockdan.lifesaverlotto.cmm.lottery.*` green. 다만 1차 수정 후 재기동에서도 다음 미스매치(`identities.identity_id` CHAR↔VARCHAR)가 노출돼 whack-a-mole이 시작될 위험이 명확해진 시점에 진행을 멈췄다.

**보드 처리**

| Task | 변경 |
|---|---|
| `PHASE17-LOCAL-PROD-REGRESSION-01` | `status="planned" → "blocked"`, AgentNotes 4줄 추가(1차 시도 회고·로컬 보존된 부산물·`blocked-by`) |
| `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` | (신규, executionOrder=109) discover/implement/verify 3-Phase, Done 체크리스트 3개, AgentNotes 3개. 본 태스크 완료 시 회귀 blocked 해제 |

**환경 회수**

`docker compose down` (모든 컨테이너 + 네트워크 제거) → `rm docker-compose.override.yml`. 워킹 트리에서 override 파일 흔적 0.

관련 파일:

- [`board.xml`][boardXml]
- [`docker-compose.yml`][dockerComposeYml]
- [`TWinBaseInfo`][TWinBaseInfo]
- [`LotteryWinCrawler`][LotteryWinCrawler]
- [`LifeSaverLottoApplication`][lifeSaverLottoApplicationJava]
- [`application-prod.properties`][appProdProps]
- [plan: phase17 local prod regression guide][phase17PlanMd]

---

### 2.11 `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` 구현 — Cursor 샌드박스 chromium 자동 가드

§2.9 회귀에서 Playwright 23/23 fail의 원인이 **Cursor 샌드박스에 chromium이 설치되지 않음**(`browserType.launch: Executable doesn't exist at /tmp/cursor-sandbox-cache/.../chromium_headless_shell-1217/...`)이었던 환경 이슈를 pre-flight 가드로 흡수했다. `scripts/check-dev-login-guard.sh`와 동일 패턴.

**1) chromium 설치 (수동, 1회)**

`PLAYWRIGHT_BROWSERS_PATH=/tmp/cursor-sandbox-cache/b333f107561744d966410c0bbcdae468/playwright`에 한 줄로 설치:

```bash
npx playwright install chromium
```

| 컴포넌트 | 다운로드 | 디스크 |
|---|---|---|
| Chrome for Testing 147.0.7727.15 | 170.4 MiB | 369M |
| FFmpeg 1011 | 2.3 MiB | 4.9M |
| Chrome Headless Shell 147.0.7727.15 | 112 MiB | 257M |
| **합계** | **~285 MiB** | **~631M** |

소요 14분(`exit_code: 0`, 849793ms). `--with-deps`는 샌드박스에 `sudo apt` 권한이 없어 사용하지 못했고, chromium 정적 빌드는 시스템 의존성 없이 동작하므로 영향 없음.

**2) 가드 스크립트 — `scripts/check-playwright-browser.sh`**

| 동작 | 결과 |
|---|---|
| `${PLAYWRIGHT_BROWSERS_PATH:-$HOME/.cache/ms-playwright}` 아래 `chrome-headless-shell`을 `find -executable`로 탐색 | 있으면 `[guard][ok]` + exit 0 (7ms) |
| 없으면 `cd $WORKSPACE_ROOT` 후 `npx playwright install chromium` 자동 호출, 그 다음 재검사 | 성공 시 0, 실패 시 stderr 메시지 + exit 1 |
| `--quiet` 옵션 | 성공 메시지 억제 (CI 로그용) |

**3) `startup-and-smoke-test.sh` 연결**

```bash
# --- pre-flight guards ---
"${WORKSPACE_ROOT}/scripts/check-dev-login-guard.sh"
"${WORKSPACE_ROOT}/scripts/check-playwright-browser.sh"
```

**4) 단위 검증**

| 분기 | 명령 | 결과 |
|---|---|---|
| Cache hit | `time ./scripts/check-playwright-browser.sh` | `[guard][ok]` + exit 0, **7ms** (real time) |
| Cache miss → install 진입 | `timeout 3 env PLAYWRIGHT_BROWSERS_PATH=/tmp/nonexistent-... ./scripts/check-playwright-browser.sh` | `[guard] Playwright chromium not found...` → `[guard] Running: npx playwright install chromium` → 실제 다운로드 시작까지 진입 확인(timeout으로 중단, exit 124) |

**5) 풀 회귀(사용자 확인)** — `check-playwright-browser.sh` 반영 후 `./startup-and-smoke-test.sh --build` 재실행하여 **API 스모크·Playwright 포함 전부 green** 확인. `PHASE17-LOCAL-PROD-REGRESSION-01`(로컬 prod-profile 회귀·b)은 사용자가 별도 진행 중.

**휘발성 주의**

`/tmp/cursor-sandbox-cache/`는 Cursor 세션 또는 OS `/tmp` 정책에 따라 사라질 수 있다. 다음 세션에서 chromium이 없으면 본 가드가 자동으로 재설치하므로 한 줄 명령(`./startup-and-smoke-test.sh --build`)으로 회귀가 자연 통과한다. 14분의 다운로드 시간이 추가될 뿐 절차는 동일.

관련 파일:

- [`check-playwright-browser.sh`][checkPlaywrightBrowserSh]
- [`startup-and-smoke-test.sh`][startupSmokeSh]
- [`check-dev-login-guard.sh`][checkDevLoginGuardSh]
- [`playwright.config.ts`][playwrightConfig]
- [`board.xml`][boardXml]

---

## 3. 문제와 해결

### 3.1 미인증 페이지의 page-load 스모크가 의미를 가지려면

| 구분 | 내용 |
|---|---|
| 문제 | `smoke/e2e/tests.json`의 loose 페이지 로드 테스트는 `form, button, .container` 같은 광범위 셀렉터를 쓴다. `/mypage`처럼 인증이 필요한 라우트를 같은 방식으로 넣으면 로그인 페이지의 셀렉터에 그냥 매칭돼 회귀 가치를 잃는다. |
| 원인 | runner.spec.ts는 `page.goto(url)` 후 `assertElementExists` 기준만 본다. 302 redirect 후 도착한 페이지에서 셀렉터가 보이면 통과한다. |
| 해결 | MYPAGE-DRAWER-01 라우트는 `tests.json`에 추가하지 않고, 의도가 명시적인 `mypage-drawer.spec.ts`에서 `page.url()` 매칭으로 redirect를 검증했다. |

---

### 3.2 인증 세션이 없는 CI에서도 의미 있는 회귀

| 구분 | 내용 |
|---|---|
| 문제 | OAuth 세션을 매번 시드하지 않으면 인증 필요 화면(`/mypage`·`/mypage/settings/terms`·드로어)을 자동 회귀에 넣을 수 없다. |
| 해결 | `phase15-dashboard.spec.ts`와 동일하게 `auth.json` 존재 여부로 분기. 없으면 인증 케이스는 `test.skip()`하고 미인증 가드 케이스 6건은 그대로 실행된다. |
| 후속 | `npx playwright codegen http://localhost:8080/login --save-storage=auth.json`을 1회 수행한 머신/세션에서는 허브·약관·드로어 렌더까지 함께 회귀된다. |

---

### 3.3 운영에 dev-login 백도어가 절대 켜지지 않게

| 구분 | 내용 |
|---|---|
| 위험 | 백도어 컨트롤러 1개가 운영에 노출되면 임의 사용자 세션을 발급할 수 있다. |
| 가드 1 (코드) | `@Profile("!prod")` (운영 프로파일에선 빈 자체가 등록되지 않음) |
| 가드 2 (설정) | `@ConditionalOnProperty(name="app.dev-login.enabled", havingValue="true")` (기본값 false) |
| 가드 3 (스크립트) | **본 세션 추가** — `scripts/check-dev-login-guard.sh`. `find … application*.properties`에서 `app.dev-login.enabled=true`를 갖는 파일이 `application-local.properties` 외에 있으면 exit 1. `startup-and-smoke-test.sh` pre-flight 단계에서 호출 |
| 운영 보장 | `application.properties` 기본값 `false`. `application-prod.properties` 별도 override 없음. 3중 가드(코드·설정·스크립트)로 운영에선 어떤 경로로도 빈이 생성되지 않음 |
| 코드 위치 격리 | `cmm/security/devlogin/` 패키지로 분리 — 점검·삭제·검색 용이 |
| 정상/위반 검증 | `application.properties`에 `app.dev-login.enabled=true`를 한 줄 임시 삽입 → 가드가 `[guard][FAIL]`과 exit 1 반환. 라인 제거 후 `[guard][ok]` 복귀 확인 |

---

### 3.5 dev-login 세션 ↔ `/onboarding` POST 경로 회귀

| 구분 | 내용 |
|---|---|
| 현상 | `google-login.spec.ts:78` 인증 케이스에서 `/onboarding` POST 후 `waitForURL("/dashboard")` 타임아웃(8s) |
| 1차 원인 | `DevLoginSeeder.ensureSeed`(line 49-56)는 `User`만 저장하고 `SocialAccount`/`Identity`-`SocialAccount` 연결을 만들지 않음. `OnboardingController.resolveUserId`(line 67)의 `socialAccountRepository.findByProviderAndProviderId(...)`가 빈 결과 → line 75-77 `ResponseStatusException(UNAUTHORIZED)` |
| 2차 원인 | `DevLoginController`가 `OAuth2AuthenticationToken`의 `authorizedClientRegistrationId="dev-local"`로 토큰을 만들지만, `OnboardingController.resolveUserId:60`은 `OAuthProvider.valueOf("DEV-LOCAL".toUpperCase())` 호출 — Java enum 명에 하이픈 불가, `IllegalArgumentException` 경로 |
| 영향 | dev-login 백도어 도입(2026-05-13 오전 세션) 이후 인증 흐름 자동 회귀가 처음 돌아갔을 때 드러난 격리 버그. **운영 코드 무영향** — `@Profile("!prod")` 가드 외 동작 변경 없음 |
| 처리 | 보드 `DEVLOGIN-ONBOARDING-FIX-01`(priority=medium, executionOrder=107) 등록 직후 본 세션에서 즉시 처리 — **옵션 C(공통 리졸버 재사용)** 채택. `SiteOAuth2UserService:48`이 이미 동일하게 `AuthenticatedSiteUser(oauth2User, appUserId)`를 만들고 `CurrentUserIdResolver.resolveAppUserId(auth)`가 이를 읽음. `DevLoginController` javadoc(line 37-38)도 이 추상화를 전제로 명시. `OnboardingController.resolveUserId` 사설 메서드를 `CurrentUserIdResolver.requireAppUserId(authentication)` 한 줄로 대체, 의존성 2개·imports 5개 orphan 정리. `OnboardingControllerTest`는 사설 리졸버용 mock 제거, `AuthenticatedSiteUser`+`OAuth2AuthenticationToken` 실체 사용으로 단순화. 검증: `./gradlew compileJava compileTestJava` 통과(14.3s), `./gradlew test --tests OnboardingControllerTest` 통과. E2E 최종 회귀는 사용자 환경에서 `./startup-and-smoke-test.sh` 재실행 결과로 확정 |
| 보드 처리 | `DEVLOGIN-ONBOARDING-FIX-01` BacklogTasks → ArchivedTasks 이관(`completedAt=2026-05-13T08:05:00Z`, `archivedAt=2026-05-13T08:10:00Z`) |
| 1차 fix 후 사용자 측 회귀 결과 | `21 passed / 1 skipped / 1 failed` — 동일 `:78` 케이스가 `LOTTO-500`으로 여전히 실패 (`error-context.md`) |
| 실 근원 (전수 진단) | **stale Docker image**. `./startup-and-smoke-test.sh`는 기본적으로 `--build` 없으면 이미지 캐시(`mockdanjung/life-saver-lotto:3.2.0-java17`)를 재사용. 사용자의 마지막 실행은 fix가 반영되지 않은 옛 이미지로 돌아갔고, 옛 `OnboardingController.resolveUserId`의 `OAuthProvider.valueOf("DEV-LOCAL")` `IllegalArgumentException`이 `ExceptionHandlerInterceptor`의 generic `Exception` 핸들러로 떨어져 `LOTTO-500`이 된 것. 코드 fix 자체는 처음부터 올바름 |
| 2차 검증 (워크스페이스 측) | `docker compose up -d --build` 후 curl 4종 케이스 모두 302 → /dashboard. `npx playwright test --reporter=list` 전체 23 → **22 passed / 1 skipped(TNB-guest 의도 skip) / 0 failed**. `:78` 케이스 1.0s 만에 green |
| 유사 패턴 전수 검사 | `OAuthProvider.valueOf` 호출 8개 사이트 + `findByProviderAndProviderId` 호출 6개 사이트 모두 점검. `DashboardProfileService:62-68`·`OAuth2RedirectTargetService:121-127`·`SiteOidcUserService:42-46`·`SiteOAuth2UserService:38-43`·`ProfileSetupServiceImpl:50-62` 모두 `try/catch (IllegalArgumentException)` 가드 있음. `SocialUserProfileExtractor:45`·`UserIdentityServiceImpl:44`는 enum 변환 없이 raw 문자열. `SocialIdentityLinkService:50`은 provider를 파라미터로 받음(호출부 가드 적용). → `OnboardingController`가 유일한 미가드 사이트였고 fix는 정확한 한 군데를 짚었음 |
| 운영자 안전장치 (관찰 사항) | `./startup-and-smoke-test.sh`의 기본값이 캐시 재사용이라 소스 변경 직후 인지 못 한 채 옛 이미지 회귀를 돌릴 수 있는 sharp edge가 노출됨. 본 세션에서는 스크립트 default를 건드리지 않음(스코프 외). 별도 운영 개선 태스크 후보로만 기록 |

관련 파일:

- [`DevLoginSeeder`][devLoginSeeder]
- [`DevLoginController`][devLoginController]
- [`OnboardingController`][onboardingControllerJava]

---

### 3.4 사전부터 있던 Playwright HTML reporter 경고 (해소)

| 구분 | 내용 |
|---|---|
| 현상 | `npx playwright test --list` 실행 시 `HTML reporter folder: test-results/html` 가 `test results folder: test-results`와 겹친다는 Configuration Error 경고가 출력됐다. |
| 원인 | `playwright.config.ts`의 `reporter` 항목에서 `outputFolder: 'test-results/html'`가 기본 `test-results/`의 하위에 있어 HTML reporter가 매 실행마다 `test-results/` 전체를 비우려 한다는 경고였다. |
| 처리 | `outputFolder: 'playwright-report'` + `open: 'never'`로 변경. `startup-and-smoke-test.sh` line 130의 `cp -r playwright-report` 구문(원래 `\|\| true`로 silently skip 중이던)이 이제 실제 경로를 찾는다. |
| 검증 | `npx playwright test --list`에서 `Configuration Error` 메시지 사라짐. 테스트 디스커버리 25 tests / 4 files 동일. |

---

### 3.6 prod-profile validate가 잡아낸 스키마 드리프트 — whack-a-mole 직전 정지

| 구분 | 내용 |
|---|---|
| 현상 | `SPRING_PROFILES_ACTIVE=prod`로 띄우자 `application-prod.properties:23`의 `spring.jpa.hibernate.ddl-auto=validate` 가 EntityManagerFactory 초기화 단계에서 schema-validation 에러로 부팅을 차단. 한 번에 한 테이블만 잡고 멈추는 특성상 fix → 재기동마다 다음 미스매치가 줄줄이 노출 |
| 발견 1 | `t_win_base_info.win_count` — `@Column(columnDefinition = "int default 0")`이 SQL Types 코드를 NUMERIC으로 강제, DB는 INTEGER. `expecting [int default 0 (Types#NUMERIC)]` vs `found [int (Types#INTEGER)]` |
| 발견 2 | `t_win_base_info.win_amount_per_each` — Java `BigInteger`의 JPA 기본 매핑은 `decimal(38,0) (NUMERIC)`. DB는 `bigint(20) (BIGINT)`. 기존 `columnDefinition = "bigint default 0"`은 DDL 문자열만 흉내냈을 뿐 Types 코드는 여전히 NUMERIC. 1차 발견의 columnDefinition을 정리하니 본질이 드러남 |
| 발견 3 | `identities.identity_id` — DB CHAR ↔ 엔티티 VARCHAR(255). 위 2건과는 또 다른 종류의 타입 미스매치 |
| 1인 개발 누적 컨텍스트 | local 프로파일은 `ddl-auto=update`라 엔티티 변경이 자동 반영·드리프트 누적이 silent. validate가 처음 도입되는 prod-profile 부팅 시점에 누적분이 한꺼번에 노출되는 구조. **회귀 태스크가 정확히 막아내려는 종류** |
| 본세션 처리 범위 | 1차(2건)만 의미적 변화 없는 정렬로 고침 — `BigInteger → Long`, `columnDefinition` 제거(DB DEFAULT는 V2 migration이 이미 보장). `compileJava`/`compileTestJava` + lottery 패키지 단위테스트 green. 발견 3 이후는 별도 태스크로 분리 |
| 정지 판단 | 발견 3까지 노출된 시점에 patterns가 단일 quirk가 아니라 **시스템 정리 작업** 임이 명확해짐. 한 세션에서 whack-a-mole로 끌고 가는 것은 plan 스코프를 벗어남 + 실수 위험. 1차 수정만 로컬 보존하고 본 태스크는 blocked, 후속 `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` 신설 |
| 장기 후속 sharp edge | 본 태스크가 잡아낸 종류의 회귀를 PR 머지 전 자동으로 막으려면 `startup-and-smoke-test.sh` 또는 별도 CI 스텝에 prod-profile 1회 기동 검증을 끼우는 것이 정공법. 본세션 미반영(스코프 외) — `PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01` AgentNotes에 후보로만 기록 |

---

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- **회귀 의도가 명시적이면 셀렉터도 명시적으로.** 광범위 `form, button` 같은 셀렉터는 로그인 페이지에까지 매칭돼 redirect 가드 회귀를 사실상 무력화한다. 인증이 필요한 라우트는 `page.url()` 또는 라우트별 고유 클래스로 검증해야 의미가 살아난다.

- **스모크 스펙은 데이터, 러너는 코드.** `smoke-tests.json` / `tests.json`은 데이터이므로 라우트만 늘면 JSON에 추가하면 끝이다. 러너 코드를 건드리지 않으면 회귀 영역이 최소화된다.

- **인증 세션 옵셔널 패턴**(`HAS_AUTH = existsSync('auth.json')`)은 CI/로컬 양쪽에서 의미 있는 부분 회귀를 보존해 준다. 6건의 미인증 가드는 항상 돌고, 3건의 인증 렌더는 세션이 있을 때만 돈다.

- **`ddl-auto=update`는 silent driver of drift, `validate`는 exposer.** 1인 개발에서 local 프로파일만 띄워 왔다면 엔티티 ↔ DB ↔ Flyway 3자 정합이 어긋나도 인지 못 할 수 있다. prod-profile validate 1회 기동을 PR 머지 전 자동 가드로 두면 운영 부팅 차단 종류의 회귀를 한 단계 앞에서 잡는다. 본세션은 이 절차 자체의 가치를 실제 사례로 입증한 회차.

- **whack-a-mole 직전 멈춰서 후속 분리.** validate가 한 번에 한 테이블만 잡고 멈추는 특성상 fix-rebuild를 반복하면 한 세션이 무한정 늘어진다. 패턴이 단일 quirk가 아니라 시스템 정리임이 명확해지는 시점(본세션의 경우 3건째)에 멈추고 후속 태스크로 분리하면 1차 수정의 의미는 살리고 다음 세션이 인지된 상태로 시작된다.

---

## 5. 다음 액션

### Planet645

- [x] `feature/tnb-myinfo-drawer-settings` 브랜치 PR 머지 (스모크·E2E 회귀 + dev-login 백도어 포함)
- [ ] **`PHASE17-PROD-VALIDATE-SCHEMA-DRIFT-01`** — 본세션 1차 수정(TWinBaseInfo 정렬)을 흡수 또는 분리 PR로 정리 + 발견 3(`identities.identity_id` CHAR↔VARCHAR) 이후 validate 에러 enumerate·일괄 정렬. 완료 시 회귀 blocked 해제
- [ ] `PHASE17-LOCAL-PROD-REGRESSION-01` — 위 schema 후속 완료 시 `status="blocked" → "planned"`로 풀고 plan ([phase17 local prod regression guide][phase17PlanMd]) Step 1부터 재실행(로컬 prod-profile 기동 → API 스모크·Playwright·수동 UI·dev-login 404 증거). **prod-profile 회귀 본선은 사용자 진행 중**; blocked 해제·보드 정합은 schema 태스크와 연동
- [x] 로컬에서 `./startup-and-smoke-test.sh --build` 통합 실행 — 사용자 확인 **전부 green**(API 스모크 + Playwright). `globalSetup`·`auth.json` / `auth-admin.json` 동작은 동일 회귀에서 함께 확인됨
- [ ] Option D — OAuth Provider WireMock 모킹 (`SiteOAuth2UserService` · `SiteOidcUserService` · `OAuth2LoginSuccessHandler` 회귀) 별도 작업 착수
- [ ] HTML 폴백이 부족할 경우 Selenium 기반 후속 작업 검토 *(어제 잔존)*
- [x] `SECURITY-ENTRYPOINT-SSR-ROUTES-01` — 잔여 SSR 4건(`/onboarding/**`, `/profile/setup/**`, `/subscribe/payment`, `/subscribe/success`)에 `LoginUrlAuthenticationEntryPoint("/login")` 매처 + 스모크 회귀 케이스 4건 추가. `./startup-and-smoke-test.sh --build` 결과 API 30/30 green(신규 4건 모두 302 PASS). Backlog → Active → Archived
- [x] 보드 백로그 소스 기준 재검증 — `MYPAGE-DRAWER-CREDIT-SUMMARY-01`(코드 단일 소스 `EntitlementService.getBalance()` 이미 확정) Archive 이관. Playwright 환경 가드는 `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` 신규 백로그로 분리
- [x] `DEVLOGIN-ONBOARDING-FIX-01` — `OnboardingController`에서 `CurrentUserIdResolver` 재사용으로 사설 resolve 제거. 도커 재빌드 후 Playwright 전체 22 passed / 1 의도 skip / 0 failed로 확정. 유사 미가드 사이트 없음(8 사이트 전수 검사 완료)

### 개발 운영

- [x] `scripts/check-dev-login-guard.sh` 신설 — `app.dev-login.enabled=true`가 `application-local.properties` 외에 있으면 exit 1. `startup-and-smoke-test.sh` pre-flight 단계 연결 (운영 배포 파이프라인에서도 동일 스크립트 호출 가능)
- [x] Playwright `reporter` HTML 출력 폴더를 `playwright-report/`로 분리해 사전부터 있던 Configuration Error 경고 해소
- [x] PR 후 회귀 태스크를 보드 정책에 맞게 반영 — `PHASE17-LOCAL-PROD-REGRESSION-01` (`board.xml` Backlog, verify 단일 Phase·체크리스트·AgentNotes; 이 프로젝트는 staging 환경 없음을 명시)
- [x] `PLAYWRIGHT-CHROMIUM-PREFLIGHT-01` — `scripts/check-playwright-browser.sh` 신설 + `startup-and-smoke-test.sh` pre-flight 연결. cache hit 7ms, cache miss 시 `npx playwright install chromium` 자동 호출. chromium 1217(~285MB) 설치 후 사용자 측 `./startup-and-smoke-test.sh --build`로 **전부 green** 확인. Active → Archived

---

## 6. 기준

이 글은 2026-05-13 세션에서 추가·검증한 스모크/E2E 스펙 변경 범위를 요약한다.  
코드와 불일치할 경우 저장소(`smoke/spec/smoke-tests.json`, `smoke/e2e/mypage-drawer.spec.ts`)의 최신 상태를 기준으로 한다.

---

[smokeTestsJson]: ../../smoke/spec/smoke-tests.json
[smokeRunnerPy]: ../../smoke/runner/smoke_runner.py
[mypageDrawerSpec]: ../../smoke/e2e/mypage-drawer.spec.ts
[phase15DashboardSpec]: ../../smoke/e2e/phase15-dashboard.spec.ts
[playwrightConfig]: ../../playwright.config.ts
[startupSmokeSh]: ../../startup-and-smoke-test.sh
[dockerComposeYml]: ../../docker-compose.yml
[e2eTestsJson]: ../../smoke/e2e/tests.json
[googleLoginSpec]: ../../smoke/e2e/google-login.spec.ts
[authLayoutHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/layouts/auth_layout.html
[onboardingControllerJava]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/identity/api/OnboardingController.java
[globalSetupTs]: ../../smoke/e2e/global-setup.ts
[checkDevLoginGuardSh]: ../../scripts/check-dev-login-guard.sh
[boardXml]: ../../.claude/tasks/board.xml
[phase17Backlog]: ../../artifact/plan/phase17-product-engineering-backlog.md
[saturdayPipelineMd]: ../../artifact/plan/saturday-draw-night-pipeline.md
[roadmapXml]: ../../.claude/tasks/roadmap.xml
[CLAUDE]: ../../CLAUDE.md

[mypageController]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/ui/MypageController.java
[notificationController]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/ui/NotificationController.java
[tnbHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/fragments/tnb.html
[defaultLayoutHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/layouts/default_layout.html
[settingsMenuHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/content/mypage/settings-menu.html
[termsHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/content/mypage/settings/terms.html

[devLoginController]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/security/devlogin/DevLoginController.java
[devLoginSeeder]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/security/devlogin/DevLoginSeeder.java
[authenticatedSiteUser]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/security/AuthenticatedSiteUser.java
[currentUserIdResolver]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/security/CurrentUserIdResolver.java
[securityConfig]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/security/SecurityConfig.java
[appProps]: ../../com.mockdan.life-saver-lotto/src/main/resources/application.properties
[appLocalProps]: ../../com.mockdan.life-saver-lotto/src/main/resources/application-local.properties
[authenticatedLayoutModelAdvice]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/ui/AuthenticatedLayoutModelAdvice.java
[creditController]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/credit/api/CreditController.java
[checkPlaywrightBrowserSh]: ../../scripts/check-playwright-browser.sh

[TWinBaseInfo]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/recommendation/entity/TWinBaseInfo.java
[LotteryWinCrawler]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/lottery/LotteryWinCrawler.java
[lifeSaverLottoApplicationJava]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/LifeSaverLottoApplication.java
[appProdProps]: ../../com.mockdan.life-saver-lotto/src/main/resources/application-prod.properties
[phase17PlanMd]: ../../.cursor/plans/phase17_local_prod_regression_guide_4685523b.plan.md
