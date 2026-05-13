---
title: "개발일지 — 2026-05-12"
excerpt: "Planet645 — TNB·내 정보 드로어·설정 허브·약관·크레딧, Thymeleaf Security, 레이아웃 header화. 기존: 당첨 수집·DB·추천 UI·collation·artifact·보드."
date: 2026-05-12 12:00:00 +0900
last_modified_at: 2026-05-13 10:00:00 +0900
categories: [deVlog]
tags: [planet645, spring, jekyll, 개발일지]
toc: true
toc_sticky: true
---

# 개발일지 — 2026-05-12

> Planet645 관리자 배치, 동행복권 수집, 추천 UI·DB·collation·문서 `artifact/` 정리에 더해, **TNB·내 정보 드로어·설정 허브·약관 동의 화면·크레딧 요약**을 구현·정리했다.

---

## 1. 오늘 목표

### Planet645

- [x] 관리자 당첨 수집 API와 `curl` 호출 방식 정리
- [x] `win-num-api` 로컬 설정 이슈 확인
- [x] 동행복권 HTML 폴백 및 JSON 보조 경로 정리
- [x] `pst_list_row_json` 길이 이슈 대응
- [x] 추천 결과 UI 번호 가시성 개선
- [x] 추천 번호 저장/표시 순서 오름차순 정렬
- [x] dev 리셋 SQL 및 collation 정규화
- [x] 마이페이지 Drawer PRD 및 백로그 반영
- [x] TNB·내 정보 드로어·설정 허브·약관 동의·크레딧 UI (`feature/tnb-myinfo-drawer-settings` 푸시)

### 개발 운영

- [x] 개발일지 기준 경로 정리
- [x] `CLAUDE.md` 개발 일지 규칙 정리
- [x] GitHub Pages / Jekyll / Minimal Mistakes 스택 확인
- [x] 에이전트 워크플로 및 보드 운영 문구 정리
- [x] 워크스페이스 문서 저장소 `artifact/`로 리팩토링 (앱·ML `docs/` 통합)

---

## 2. 주요 변경 사항

### 2.1 동행복권 당첨번호 수집

기존 HTML 정적 파싱은 동행복권 페이지 구조와 클라이언트 렌더링 특성상 불안정했다.

이에 따라 `selectPstLt645InfoNew.do` JSON 보조 경로를 추가하고, `LotteryWinCrawler` 내부에 `tryClientAlignedFallback` 흐름을 정리했다.

관련 설정은 다음과 같다.

| 항목 | 내용 |
|---|---|
| 설정 키 | `app.lotto.win-num-fetch-fallback-enabled` |
| 환경 변수 | `LOTTO_WIN_NUM_FETCH_FALLBACK_ENABLED` |
| 관리자 API | `POST /auth/admin/batch/lottery-win-html-fallback` |
| 비활성화 시 결과 | `FALLBACK_DISABLED` |

관련 파일:

- [`LotteryWinFetchUriBuilder`][LotteryWinFetchUriBuilder]
- [`LotteryWinCrawler`][LotteryWinCrawler]

---

### 2.2 DB 마이그레이션

`pst_list_row_json` 컬럼에서 `Data too long for column` 오류가 발생할 수 있어 `LONGTEXT` 기준으로 정리했다.

| 항목 | 변경 |
|---|---|
| Flyway | `V24__fix_t_lotto_result_pst_list_row_json_longtext.sql` 추가 |
| Entity | `TLottoResult.pstListRowJson`에 `columnDefinition = "LONGTEXT"` 명시 |

관련 파일:

- [`V24`][V24]
- [`TLottoResult`][TLottoResult]

---

### 2.3 추천 결과 UI

추천 결과 화면에서 번호가 흰 카드 위에 흰 글자로 표시되는 문제가 있었다.

원인은 `.lotto-number` 기본 스타일과 구간별 배경 클래스 매핑이 어긋난 것이었다.  
번호 구간별 색상 클래스를 정리하고, 공통 `lotto-ball` fragment를 재사용하도록 정리했다.

또한 점수 표현은 당첨 확률로 오해되지 않도록 **“추천 점수”**로 고정하고, `title` / `aria-label` 기반 도움말을 추가했다.

관련 파일:

- [`result.html`][resultHtml]
- [`lotto-ball.html`][lottoBall]
- [`score-help-tooltip.html`][scoreHelpTooltip]

---

### 2.4 추천 번호 정렬

추천 번호 저장 및 표시 순서를 `1–45` 오름차순으로 정렬했다.

관련 파일:

- [`RecommendationFlowServiceImpl`][RecommendationFlowServiceImpl]
- [`UserRecommendationService`][UserRecommendationService]

---

### 2.5 개발 SQL 및 collation 정리

dev 리셋 SQL 실행 중 다음 오류가 발생했다.

```text
Illegal mix of collations
```

원인은 일부 테이블/컬럼이 `utf8mb4_general_ci`, 일부가 `utf8mb4_unicode_ci`로 섞여 있었기 때문이다.

대응은 다음과 같이 정리했다.

| 구분 | 처리 |
|---|---|
| Flyway | `V23__normalize_collation.sql` 추가 |
| dev 수동 보정 | `normalize-collation.sql` 추가 |
| reset SQL | `reset-target-user.sql` 정리 |
| 운영 대량 테이블 | 유지보수 창 또는 `pt-online-schema-change` / `gh-ost` 검토 |

관련 파일:

- [`V23`][V23]
- [`normalize-collation.sql`][normalizeCollationSql]
- [`reset-target-user.sql`][resetTargetUserSql]

---

### 2.6 제품 문서 및 백로그

마이페이지 Drawer 요구사항을 PRD로 정리하고, 보드에 `MYPAGE-DRAWER-01` 백로그를 반영했다.

관련 파일:

- [`mypage-drawer.md`][mypageDrawer]
- [`board.xml`][boardXml]

---

### 2.7 개발 운영 문서

개발일지와 AI 에이전트 운영 규칙을 정리했다.

| 항목 | 내용 |
|---|---|
| 개발일지 위치 | `jungmockdan.github.com/_posts/` |
| 파일명 규칙 | `YYYY-MM-DD-developer-log.md` |
| 사이트 스택 | GitHub Pages + Jekyll + Minimal Mistakes |
| 운영 규칙 | `CLAUDE.md`의 개발 일지 / Agent-led / handoff 규칙 기준 |

관련 파일:

- [`CLAUDE.md`][CLAUDE]
- [`.claude/tasks/board.xml`][boardXml]

---

### 2.8 문서 저장소 리팩토링 (`artifact/`)

> 작성: 2026-05-12 / 마지막 갱신: 2026-05-13. 본문은 어제 작업을 그대로 두고, 후속 정리는 아래 별도 절(後속)에 분리해 둔다.

앱·ML 프로젝트 `docs/` 트리에 흩어져 있던 문서를 워크스페이스 공통 저장소 `artifact/`로 통합했다. 분류는 수명주기(`prd / design / runbook / plan / devlog / archive`)와 주제(`api, data-model, user-identity, billing-subscription, recommendation-ml, ui, decisions`)를 함께 쓰는 2축 구조로 잡았다.

| 항목 | 내용 |
|---|---|
| ML `docs/` | `ai-llm-plan-v02.md` → `artifact/design/recommendation-ml/llm-explanation-service-design.md`, `v01`은 `archive/recommendation-ml/`로 |
| 앱 `docs/` | PRD·설계·런북·완료 계획·평가·UI 스펙 등 65개 파일을 카테고리별로 이동 |
| 앱 `docs/` 잔존 | `README.md` 한 개만 두고 새 위치 안내 |
| 네이밍 | kebab-case 일관화 (`legacy-docs-index.md`, `v0dev-global-first-prompt.md`, `oauth-conflict-resolution.md`, `artifact-binary-fairy-fix.md` 등) |
| 인덱스 | `artifact/README.md`(분류 기준), `artifact/DOCS_INDEX.md`(전체 매핑) 신설 |
| 참조 수정 | `CLAUDE.md`, `.claude/tasks/board.xml`, `archive-history.xml`, `ex_task.xml`, `schema/structure.xml`, 그리고 이동된 문서들의 내부 상대 링크 일괄 정정 |
| 당시 검증 | 작업 시점 스윕에서 `docs/...`, `ai-llm-plan-v0`, 구 파일명, `[planned]` 등 잔존 참조 0건 |

관련 파일:

- [`artifact/README.md`][artifactReadme]
- [`artifact/DOCS_INDEX.md`][artifactDocsIndex]
- [`com.mockdan.life-saver-lotto/docs/README.md`][appDocsReadme]
- [`com.mockdan.life-saver-lotto-ml/docs/README.md`][mlDocsReadme]

#### 후속 (2026-05-13)

위 변경분은 같은 날 commit되지 않고 `git stash` 3개로 보존됐다가 다음 날 모두 staging에 재적용했다. 그 사이 별도로 commit된 `artifact/as-built/` Layer 0 토픽 시트 8장(`monorepo-runtime, recommendation-pipeline, auth-oauth, ui-thymeleaf, ml-service, llm-service, billing-entitlement, ops-smoke-e2e`)과 충돌 없이 **공존**하도록 정리했다.

| 항목 | 내용 |
|---|---|
| Layer 모델 | `as-built/`(Layer 0 요약) ↔ `prd / design / runbook`(Layer 1 SoT) ↔ `plan / analyze / devlog / ref / archive`(Layer 2)로 명시 |
| as-built README | Layer 2 인덱스에 `prd/`, `design/`, `runbook/{backend,setup,sql}/`, `devlog/`, `ref/`, `archive/` 추가. 옛 “Planning and tracking”을 “Detailed sources”로 변경하고 `DOCS_INDEX.md` 포인터 추가 |
| 8개 토픽 | 각 시트에 `artifact/design/...`, `artifact/prd/...`, `artifact/runbook/...` SoT cross-link 한 줄씩 보강. `auth-oauth.md`의 끊긴 `plan/oauth-conflict-resolution-complete.md` 참조도 `plan/done/oauth-conflict-resolution.md`로 정정 |
| 인덱스 | `artifact/README.md`에 3-Layer 구조 표 + `as-built/` 카테고리 도입, `artifact/DOCS_INDEX.md`에 `as-built/` 8개 시트 항목 추가 |
| 파일명 | `devlog/technical-improvements-summary.md` → `devlog/2026-04-02-technical-improvements-summary.md`로 날짜 prefix 통일(`git mv`로 history 보존). 참조 4건 동시 정정 (`DOCS_INDEX.md`, `2026-04-02-handoff.md`, `ai-development-detailed-plan.md`. archive 안쪽 옛 표기는 역사적 정확성을 위해 유지) |
| 용어 통일 | `journal` 표기 폐기, **`devlog`로 통일**. `CLAUDE.md`/`AGENTS.md`(심링크)에 워크스페이스 내부 보관 위치 = `artifact/devlog/`로 명시 |
| Submodule 정리 | 워크스페이스 repo가 `jungmockdan.github.com`을 gitlink(mode `160000`)로 들고 있었으나 `.gitmodules`는 없는 비정상 상태였음을 확인. 사용자 결정에 따라 gitlink 제거를 그대로 두고 publish repo는 형제 디렉터리로 분리 운영. `.claude/worktrees/oauth-conflict-resolution`도 동일하게 gitlink 제거 |
| 최종 스윕 | `docs/...`, `ai-llm-plan-v0`, 구 파일명, `[planned]`, `journal/` 등 깨진 참조 0건. 의도적 잔존 2건만 화이트리스트 — `.claude/tasks/board.xml`의 “구 PRD 경로는 비어 있음” 안내 메모, `com.mockdan.life-saver-lotto-ml/docs/README.md`의 “이전 파일 → 새 위치” 매핑표 |
| 현재 staging 규모 | 워크스페이스 89건(artifact 신규/이동·rename + 참조 수정 + gitlink 2건 제거). 앱·ML repo는 clean. publish repo는 본 글 1건 modified + `.gitignore` untracked |

#### 남은 작업

- 워크스페이스 repo · publish repo 두 곳에 분리 commit·push (앱·ML repo는 이전 단계에서 이미 정리됨)
- LLM 서비스(`com.mockdan.life-saver-lotto-llm/`)는 `docs/` 신설 또는 `artifact/` 흡수 정책 결정
- `.cursorignore` 12행 `*.pyc\` 매달린 백슬래시 정리 (Cursor `rg` 도구 정상화)
- `artifact/` 내부 상대 링크 정합성 정기 점검 (월 1회 정도)

---

### 2.9 내 정보(TNB) · 드로어 · 설정 허브

PRD(`mypage-drawer.md`)에 맞춰 **로그인 사용자 상단 바(TNB)**와 **내 정보 드로어**를 구현했다. 표기는 **「마이페이지」→「내 정보」**로 통일했고, `/mypage`는 **설정 허브**(`settings-menu`)로 진입하도록 정리했다.

| 항목 | 내용 |
|---|---|
| 레이아웃 모델 | `AuthenticatedLayoutModelAdvice`에서 미읽 알림 수, 프로필 요약, 크레딧 잔액, 플랜(유료 여부·라벨), 무료 가입 크레딧 미수령 시 안내 노출 여부를 주입 |
| TNB | 브랜드 좌측·아이콘(알림·내 정보) 우측; 알림 배지·벨은 `/notifications`로 연결 |
| 드로어 | 프로필·플랜·크레딧을 한 카드로 묶고, **구분선 + 톱니 + 「설정」** 아래에 설정 바로가기(드로어 전용 `div` 목록)를 두어 허브와 중복을 줄임 |
| 약관 | `GET /mypage/settings/terms` 조회 화면; 허브·드로어·프로필에서 링크; `UserTermsAgreementRepository.findByUserIdWithTermsFetched`로 N+1 완화 |
| 레이아웃 마크업 | 상단 `nav.navbar` 안에 또 다른 `nav`가 중첩되며 스타일이 깨지는 이슈가 있어, `default_layout` 상단을 **`header.navbar.navbar-top`**으로 바꾸고 드로어 설정 목록은 **`div` + `mypage-drawer-settings-*` 클래스**로 분리 |
| 허브 전용 fragment | `fragments/mypage-settings-menu.html`의 `mypageSettingsMenu`를 설정 메뉴 페이지에서 재사용 |
| Spring Security + Thymeleaf | `thymeleaf-extras-springsecurity6` 및 컴파일 `-parameters`로 `sec:authorize`가 실제 DOM에 적용되도록 정리; 로컬에서 dialect 미적용 시 `gradle clean` 후 재기동으로 해결 |
| 기타 | 프로필 이메일 표시의 `th:text` 삼항에서 `${}`가 중간에 닫혀 문자열로 노출되던 문제를 **삼항 전체를 한 `${}` 블록**으로 수정 |

Git: 앱 저장소 `com.mockdan.life-saver-lotto` 브랜치 **`feature/tnb-myinfo-drawer-settings`** 생성 후 `origin`에 푸시했다. (PR은 GitHub에서 앱 repo 기준으로 연다.)

관련 파일:

- [`tnb.html`][tnbHtml]
- [`default_layout.html`][defaultLayoutHtml]
- [`AuthenticatedLayoutModelAdvice`][authenticatedLayoutModelAdvice]
- [`settings-menu.html`][settingsMenuHtml]
- [`mypage-settings-menu.html`][mypageSettingsMenuFragment]
- [`MypageController`][mypageController]
- [`terms.html`][termsHtml]
- [`design-system.css`][designSystemCss]

---

## 3. 문제와 해결

### 3.1 관리자 당첨 수집 API

| 구분 | 내용 |
|---|---|
| 문제 | `POST /auth/admin/batch/lottery-win`에 JSON 본문만 전달하면 요청한 `round`가 반영되지 않았다. |
| 원인 | `AdminLotteryBatchController`가 `round`를 `@RequestParam`으로만 받고 있었다. |
| 결과 | JSON 본문은 바인딩되지 않아 `round == null`이 되고, `resolveNextRound()`가 실행됐다. |
| 해결 | 호출 시 `?round=1222` 쿼리 파라미터를 사용한다. |
| 후속 | JSON 바인딩이 필요하면 `@RequestBody` 병행 처리를 별도 설계한다. |

예시:

```bash
curl -X POST "http://localhost:8080/auth/admin/batch/lottery-win?round=1222"
```

관련 파일:

- [`AdminLotteryBatchController`][AdminLotteryBatchController]

---

### 3.2 로컬 `win-num-api` 설정

| 구분 | 내용 |
|---|---|
| 문제 | `http://127.0.0.1:9/...` 요청 후 연결 거부가 발생했다. |
| 원인 | `application-local.properties`에서 `app.lotto.win-num-api`가 죽은 호스트로 덮어써져 있었다. |
| 해결 | 기본값은 `application.properties`의 동행복권 API를 사용한다. |
| 후속 | 로컬 스텁이 필요하면 WireMock 등 의도된 URL을 사용한다. |

관련 파일:

- [`application.properties`][applicationProperties]
- [`application-local.properties`][applicationLocalProperties]

---

### 3.3 동행복권 HTML 파싱 한계

| 구분 | 내용 |
|---|---|
| 문제 | HTML 정적 파싱만으로 번호 수집이 실패하기 쉬웠다. |
| 원인 | 결과 페이지가 JS 기반으로 데이터를 채우는 구조였다. |
| 해결 | 서버 JSON 보조 경로인 `selectPstLt645InfoNew.do`를 사용한다. |
| 후속 | DOM 렌더 결과가 반드시 필요하면 Selenium 또는 동등한 브라우저 렌더링 도구를 검토한다. |

관련 파일:

- [`DhLotteryWinResponseParser`][DhLotteryWinResponseParser]

---

### 3.4 `pst_list_row_json` 길이 오류

| 구분 | 내용 |
|---|---|
| 문제 | `Data too long for column 'pst_list_row_json'` 오류 발생 |
| 원인 | 일부 환경에서 컬럼이 짧은 타입으로 잡혀 있었다. |
| 해결 | `V24` 마이그레이션과 Entity `LONGTEXT` 명시를 추가했다. |

---

### 3.5 추천 결과 번호 미노출

| 구분 | 내용 |
|---|---|
| 문제 | 추천 결과 번호가 보이지 않았다. |
| 원인 | 흰 카드 배경 위에 흰 글자가 표시됐다. |
| 해결 | 번호 구간별 배경 클래스와 글자색을 명시하고 `lotto-ball` fragment를 재사용했다. |

관련 파일:

- [`design-system.css`][designSystemCss]

---

### 3.6 collation 충돌

| 구분 | 내용 |
|---|---|
| 문제 | dev 리셋 SQL 실행 중 collation 충돌 발생 |
| 원인 | `credit_ledger`, `entitlement`, `payment`, `users` 계열 테이블의 collation이 섞여 있었다. |
| 해결 | `V23`과 dev용 `normalize-collation.sql`로 정규화했다. |
| 주의 | 운영 대량 테이블은 온라인 스키마 변경 도구 또는 유지보수 창을 검토해야 한다. |

관련 파일:

- [`V3__Create_identity_tables.sql`][V3]

---

### 3.7 Hibernate SQL 바인딩 로그

로컬에서 SQL 바인딩 값을 더 자세히 확인해야 할 때는 다음 로그 설정을 사용한다.

```properties
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.orm.jdbc.bind=TRACE
```

관련 파일:

- [`application-local.properties`][applicationLocalProperties]

---

### 3.8 다중 git repo 경계의 문서 이동

| 구분 | 내용 |
|---|---|
| 문제 | 앱·ML 프로젝트가 워크스페이스와 별개 git repo였다. `docs/` → `artifact/` 이동은 repo 경계를 넘어 `git log --follow` 히스토리가 끊긴다. |
| 원인 | `.gitmodules`가 없는 nested repo 구조. `git mv`가 repo 경계 너머로 동작하지 않는다. |
| 해결 | repo별로 삭제·추가가 별개 커밋이 되도록 두고, Phase A(ML 깨끗 → 골격·인덱스), Phase B(앱 변경분 push 완료 후 대량 이동)로 분리해 진행했다. |
| 후속 | 잔존 경로 검증은 `python3` 스크립트로 일괄 grep (`rg`는 `.cursorignore` glob 오류로 실패) → 잔존 0건 확인. |

---

### 3.9 `sec:authorize`가 템플릿에 반영되지 않음

| 구분 | 내용 |
|---|---|
| 문제 | Thymeleaf에서 `sec:authorize`가 그대로 출력되거나 동작하지 않았다. |
| 원인 | Spring Security 6용 dialect·컴파일 `-parameters` 미정렬 또는 캐시된 클래스패스. |
| 해결 | `thymeleaf-extras-springsecurity6`·`-parameters` 적용 후 필요 시 `gradle clean` 및 애플리케이션 재기동. |

---

### 3.10 Thymeleaf `th:text` 삼항과 `${}` 중첩

| 구분 | 내용 |
|---|---|
| 문제 | 이메일 등 표시에 `${...}`가 중간에 끊겨 **문자 그대로** 화면에 노출됐다. |
| 원인 | 삼항 연산자 안에서 표현식 괄호·`${}` 범위가 잘못 잡혔다. |
| 해결 | 삼항 **전체**를 하나의 `${ ... }` 안에 넣어 파싱되도록 수정. |

---

### 3.11 TNB·드로어에서 `nav` 중첩으로 레이아웃 겹침

| 구분 | 내용 |
|---|---|
| 문제 | 상단 바와 드로어 내부 설정 영역이 겹치거나 flex·sticky 동작이 이상했다. |
| 원인 | `nav.navbar` 내부에 또 다른 `nav`(설정 목록 등)가 중첩되며 브라우저·스타일 상 네비게이션 시맨틱이 충돌. |
| 해결 | 최상단을 `header`로 바꾸고, 드로어 설정 링크는 `nav` 대신 **`div` + 전용 클래스**로 분리. |

---

## 4. 배운 점

- Spring MVC에서 `@RequestParam`과 `@RequestBody`는 호출 방식이 다르다.  
  관리자 배치 API는 호출 스크립트와 컨트롤러 바인딩 방식을 반드시 맞춰야 한다.

- 외부 사이트 HTML 스크랩은 구조 변경과 클라이언트 렌더링에 취약하다.  
  가능하면 JSON API를 우선하고, 폴백은 설정 플래그로 통제하는 편이 안전하다.

- MariaDB/MySQL은 테이블 기본 collation뿐 아니라 컬럼 단위 collation도 중요하다.  
  컬럼 collation이 섞이면 조인, 서브쿼리, 비교 조건에서 `1267` 오류가 발생할 수 있다.

- 추천 점수의 `%` 표기는 당첨 확률로 오해될 수 있다.  
  사용자에게 노출되는 라벨은 “추천 점수”처럼 의미를 제한해서 표현하는 것이 낫다.

- 개발일지는 도메인 변경과 개발 운영 변경을 분리해야 나중에 추적하기 쉽다.

- 다중 repo 워크스페이스에서 문서를 다른 repo로 옮기면 `git log --follow`가 끊긴다.  
  대신 분류·접근성·중복 제거 이득을 얻는다. 결정은 트레이드오프 기준으로 명시해 두는 것이 좋다.

- 문서 인덱스(`DOCS_INDEX.md`)에 `[planned]` 같은 미래 상태를 남기면 금세 부정확해진다.  
  이동이 끝나면 곧장 현재 상태로 갱신해 “지금의 SoT”만 남기는 편이 안전하다.

- `.cursorignore`의 한 줄 glob 오류로 Cursor `rg` 도구 전체가 실패할 수 있다.  
  검증 단계에서는 셸 `python3` 스크립트 같은 폴백 수단을 갖춰 둘 가치가 있다.

- Thymeleaf + Spring Security 6는 **dialect·`-parameters`·클린 빌드**를 맞춰야 `sec:*` 속성이 안정적으로 동작한다.

- **`<nav>` 중첩**은 상단 바·드로어 같이 flex·sticky·z-index를 쓰는 UI에서 예기치 않은 겹침을 만들 수 있다. 시맨틱이 필요하면 최상단 `header`와 내부 `div` 목록으로 역할을 나누는 편이 낫다.

---

## 5. 다음 액션

### Planet645

- [ ] `POST /auth/admin/batch/lottery-win`에 `@RequestBody` 병행이 필요한지 검토
- [ ] HTML 폴백이 부족할 경우 Selenium 기반 후속 작업 검토
- [x] `mypage-drawer.md` 기준 1차 구현(TNB·드로어·허브·약관·크레딧) — 브랜치 `feature/tnb-myinfo-drawer-settings` 푸시
- [ ] 위 브랜치 PR 머지 및 스모크·회귀 확인

### 개발 운영

- [ ] `board.xml`의 `metadata/lastUpdated` 점검
- [ ] 스테이징 점검 태스크를 보드 정책에 맞게 반영
- [ ] `artifact/` 리팩토링 결과를 워크스페이스·앱·ML repo에 각각 커밋·푸시
- [ ] LLM 서비스(`com.mockdan.life-saver-lotto-llm/`)도 `docs/` 신설 또는 `artifact/`로 흡수 여부 결정
- [ ] `.cursorignore` 12행 `*.pyc\` 매달린 백슬래시 정리 (Cursor `rg` 도구 정상화)
- [ ] `artifact/` 내부 상대 링크 정합성 추가 점검 (마이그레이션 직후 1회)

---

## 6. 기준

이 글은 2026-05-12 세션에서 논의하고 수정한 범위를 요약한다.  
코드, 마이그레이션, 문서 내용과 불일치할 경우 저장소의 최신 상태를 기준으로 한다.

---

[LotteryWinFetchUriBuilder]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/lottery/LotteryWinFetchUriBuilder.java
[LotteryWinCrawler]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/lottery/LotteryWinCrawler.java
[DhLotteryWinResponseParser]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/lottery/DhLotteryWinResponseParser.java
[AdminLotteryBatchController]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/cmm/api/AdminLotteryBatchController.java

[V3]: ../../com.mockdan.life-saver-lotto/src/main/resources/db/migration/V3__Create_identity_tables.sql
[V23]: ../../com.mockdan.life-saver-lotto/src/main/resources/db/migration/V23__normalize_collation.sql
[V24]: ../../com.mockdan.life-saver-lotto/src/main/resources/db/migration/V24__fix_t_lotto_result_pst_list_row_json_longtext.sql
[TLottoResult]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/recommendation/entity/TLottoResult.java

[resultHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/content/recommend/result.html
[lottoBall]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/fragments/lotto-ball.html
[scoreHelpTooltip]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/fragments/score-help-tooltip.html
[designSystemCss]: ../../com.mockdan.life-saver-lotto/src/main/resources/static/css/design-system.css

[RecommendationFlowServiceImpl]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/recommendation/service/impl/RecommendationFlowServiceImpl.java
[UserRecommendationService]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/recommendation/service/UserRecommendationService.java

[resetTargetUserSql]: ../../artifact/runbook/sql/reset-target-user.sql
[normalizeCollationSql]: ../../artifact/runbook/sql/normalize-collation.sql
[mypageDrawer]: ../../artifact/prd/mypage-drawer.md
[boardXml]: ../../.claude/tasks/board.xml
[CLAUDE]: ../../CLAUDE.md
[applicationProperties]: ../../com.mockdan.life-saver-lotto/src/main/resources/application.properties
[applicationLocalProperties]: ../../com.mockdan.life-saver-lotto/src/main/resources/application-local.properties

[artifactReadme]: ../../artifact/README.md
[artifactDocsIndex]: ../../artifact/DOCS_INDEX.md
[appDocsReadme]: ../../com.mockdan.life-saver-lotto/docs/README.md
[mlDocsReadme]: ../../com.mockdan.life-saver-lotto-ml/docs/README.md

[tnbHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/fragments/tnb.html
[defaultLayoutHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/layouts/default_layout.html
[authenticatedLayoutModelAdvice]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/ui/AuthenticatedLayoutModelAdvice.java
[settingsMenuHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/content/mypage/settings-menu.html
[mypageSettingsMenuFragment]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/fragments/mypage-settings-menu.html
[mypageController]: ../../com.mockdan.life-saver-lotto/src/main/java/com/mockdan/lifesaverlotto/ui/MypageController.java
[termsHtml]: ../../com.mockdan.life-saver-lotto/src/main/resources/templates/content/mypage/settings/terms.html