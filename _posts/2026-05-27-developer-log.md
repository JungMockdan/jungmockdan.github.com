---
title: "개발일지 — 2026-05-27 (보안 밴드·주소검색·CI 앵커)"
excerpt: "보안 밴드 239·243 runbook·CVE 점검. 프로필 Daum 우편번호·V6. CI 앵커 scaffold(V7). 보드 백로그 비움."
categories: [deVlog]
tags: [planet645, security, identity, profile, supply-chain, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-27 22:00:00 +0900
last_modified_at: 2026-05-27 22:13:48 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** 백로그 **239 → 243 → 250 → 260** 순으로 마무리. **의존성·시크릿** runbook·점검 도구, **프로필 주소검색**(Daum), **CI 무료혜택 어뷰징** scaffold. `artifact/ops/tasks/board.md` **Backlog 비움**.

## 1. 오늘 목표

### Planet645

- [x] `SECURITY-DEPENDENCY-SUPPLY-01` — CVE 게이트 최소선
- [x] `SECURITY-SECRETS-GOVERNANCE-01` — 시크릿 거버넌스
- [x] `PROFILE-ADDRESS-SEARCH-01` — Daum 우편번호·주소 컬럼
- [x] `IDENTITY-CI-FREE-ABUSE-01` — CI 앵커 scaffold (PASS 연동 전)

### 개발 운영

- [x] 보드 4건 아카이브 · `archive-history-2026-06-01.md` · task count 54

---

## 2. 주요 변경 사항

### 2.1 SECURITY-DEPENDENCY-SUPPLY-01 — 의존성·공급망 CVE 게이트 (개발 운영)

| 산출 | 내용 |
|------|------|
| Runbook | [`dependency-supply-chain.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/runbook/security/dependency-supply-chain.md) |
| CI | workspace `dependency-supply-audit.yml` — Spring OWASP report + ML/LLM `pip-audit` (주간, `continue-on-error`) |
| Dependabot | workspace Actions · Spring Gradle · ML/LLM pip |
| Spring | `org.owasp.dependencycheck` — `failBuildOnCVSS=11` (report-only) |

로컬: `./gradlew dependencyCheckAnalyze` · `uv run pip-audit`.

### 2.2 SECURITY-SECRETS-GOVERNANCE-01 — 시크릿 거버넌스 (개발 운영)

| 산출 | 내용 |
|------|------|
| Runbook | [`secrets-governance.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/runbook/security/secrets-governance.md) |
| Guard | `scripts/check-secrets-guard.sh` — main 소스 트리 패턴 스캔 (test 제외) |

env 매핑·OAuth/DB/OpenAI 로테이션 절차. CI hook 연동은 후속.

### 2.3 PROFILE-ADDRESS-SEARCH-01 — 프로필 주소검색 (Planet645)

**선정:** Daum/Kakao **우편번호 팝업** (`postcode.v2.js`) — 승인키·서버 프록시 없음.

| 계층 | 변경 |
|------|------|
| DB | Flyway `V6__profile_address_search.sql` — `postal_code`, `road_address`, `address_detail`, `address_province`, `address_district` |
| 도메인 | `ProfileAddressSupport` — 시/도·구/군 → `address` 정규 라벨 |
| UI | `profile-setup-fields.html` · `profile-address-search.js` |
| 화면 | `/profile/setup`, `/mypage/profile` |
| CSP | prod Report-Only `script-src`에 `https://t1.daumcdn.net` |

SoT: [`11-address-search-design-delta.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/design/user-identity/11-address-search-design-delta.md). 행안부 API·E2E 팝업 자동화는 후속.

### 2.4 IDENTITY-CI-FREE-ABUSE-01 — CI 앵커·FREE_SIGNUP 게이트 (Planet645)

**범위:** PASS/NICE 연동 **전 scaffold**. 기본 **off**.

| 계층 | 변경 |
|------|------|
| DB | `V7__user_ci_anchor.sql` — `ci_hash` PK, `user_id`, `provider` |
| 서비스 | `CiAnchorService` — 동일 `ci_hash`·다른 `user_id` 시 `CiDuplicateFreeSignupException` |
| Entitlement | `grantFreeSignupCredits` → `assertEligibleForFreeSignup` (CI 미전달 시 no-op) |
| 설정 | `app.identity.ci-anchor.enabled=false` (`CI_ANCHOR_ENABLED`) |

SoT: [`10-ci-free-abuse-design-delta.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/design/user-identity/10-ci-free-abuse-design-delta.md).

### 2.5 보드·아카이브 (개발 운영)

| 항목 | 결과 |
|------|------|
| 완료 태스크 | `239`, `243`, `250`, `260` → `archive-history-2026-06-01.md` |
| `board.md` | Active·Backlog **비움** (`lastUpdated` 2026-05-27 KST) |
| task count | **54** |

---

## 3. 문제와 해결

### 3.1 Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| OWASP dependency-check 로컬 1회 실행 장시간 | NVD·첫 다운로드 | CI는 `continue-on-error` + report artifact; 게이트는 점진 강화(runbook §4) |
| `check-secrets-guard` test 리터럴 오탐 | `client-secret=` 테스트 fixture | 스캔 경로에서 `*/test/*` 제외 |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 보안 **239·243**은 제품 코드보다 **runbook + 최소 자동화**(Dependabot·주기 audit·guard)가 먼저면 백로그를 빠르게 비울 수 있다.
- 주소검색은 **팝업-only**가 CSP·키 관리 부담이 작고, 동일인용 **시/도·구/군**은 hidden + `ProfileAddressSupport`로 한곳에 모은다.
- CI 어뷰징 방지는 **플래그 off scaffold**로 스키마·Entitlement 훅만 깔아 두고 PASS 연동 시 켜는 편이 안전하다.

---

## 5. 다음 액션

> SoT: `artifact/ops/tasks/board.md` · 로드맵 `roadmap-current.md` §2 (품질·OAuth 잔여).

### 5.1 미완

#### Planet645

- [ ] 추천·LLM 필터·ML 스코어링 품질 (로드맵 Current Focus)
- [ ] 주소검색 **수동** — `/profile/setup` 팝업 → 저장 → `/mypage/profile` 재조회
- [ ] PASS/NICE 연동 · `CI_ANCHOR_ENABLED` · 동일 CI 재가입 E2E
- [ ] (선택) Ad·미션 베타 노출 — prod-profile 회귀 후

#### 개발 운영

- [ ] `check-secrets-guard.sh`를 quality-gate에 연동 (baseline 정리 후)
- [ ] dependency-check **CVSS 게이트** 단계적 강화
- [ ] (선택) 새 백로그 항목을 보드에 등록 후 `/task-next`

### 5.2 완료 (당일 — 상세 §2)

| §2 | 항목 |
|----|------|
| [2.1](#21-security-dependency-supply-01--의존성공급망-cve-게이트-개발-운영) | SECURITY-DEPENDENCY-SUPPLY-01 |
| [2.2](#22-security-secrets-governance-01--시크릿-거버넌스-개발-운영) | SECURITY-SECRETS-GOVERNANCE-01 |
| [2.3](#23-profile-address-search-01--프로필-주소검색-planet645) | PROFILE-ADDRESS-SEARCH-01 |
| [2.4](#24-identity-ci-free-abuse-01--ci-앵커free_signup-게이트-planet645) | IDENTITY-CI-FREE-ABUSE-01 |
| [2.5](#25-보드아카이브-개발-운영) | 보드 백로그 비움 · 주간 아카이브 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto` — V6 주소 · V7 `user_ci_anchor` · OWASP plugin |
| Runbook | `artifact/runbook/security/` |
| 보드 | [`board.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/ops/tasks/board.md) — Backlog empty |
| 환경 표기 | **로컬 prod-profile 회귀** — staging 표기 없음 |
| 전일 일지 | [2026-05-26-developer-log.md](2026-05-26-developer-log.md) |
