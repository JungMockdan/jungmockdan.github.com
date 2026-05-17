---
title: "개발일지 — 2026-05-17 (만 19세 연령 정책·프로필 setup/수정)"
excerpt: "Planet 645 이용 대상을 만 19세 이상(해당 연도 1월 1일 기준)으로 확정. AgePolicy·프로필 setup/수정 UI·서버 이중 검증·TERM-003 Flyway V28/V29."
date: 2026-05-17 12:00:00 +0900
last_modified_at: 2026-05-17 15:30:00 +0900
categories: [deVlog]
tags: [planet645, age-policy, profile, terms-consent, flyway, thymeleaf, mypage, 개발일지]
toc: true
toc_sticky: true
---

> [이전 2026-05-16 일지](/2026-05-16-developer-log)에서 Billing Sub B·C·E2E 회귀를 마친 뒤, **Planet 645 서비스 이용 연령**을 제품·코드·약관에 일관 반영했다.

---

## 1. 오늘 목표

- [x] 「만 19세 이상」제품 정책 확정 — **만 19세가 되는 해의 1월 1일**을 맞이한 사람
- [x] 약관 TERM-003 문구·Flyway V28/V29
- [x] 프로필 최초 입력(`/profile/setup`)·수정(`/mypage/profile`) 생년월일·버튼 강제
- [x] 서버 `@MinAge` + `AgePolicy` + 제출 시 재검증
- [x] PRD `terms-consent.md` §3.1 정합

---

## 2. 연령 판정 규칙

| 항목 | 내용 |
|------|------|
| 정의 | 생년 `Y` → **`(Y+19)년 1월 1일`(KST)** 부터 이용 가능 |
| 예 (기준 2026) | 2007년생 가능 · 2008년생 불가 |
| 생일 월·일 | 판정에 **미사용** (연도 기준) |

`AgePolicy.meetsMinimumAge()` — `birthYear + 19 <= currentYear` (KST `Clock`).

---

## 3. 주요 구현

### 3.1 백엔드 (`com.mockdan.life-saver-lotto`)

| 구분 | 파일·내용 |
|------|-----------|
| 정책 | `identity/policy/AgePolicy.java` |
| 검증 | `@MinAge`, `MinAgeValidator`, `MinimumAgeNotMetException` |
| 최초 가입 | `ProfileSetupController` — 연령 실패 시 폼 복귀 |
| 프로필 수정 | **`ProfileEditController`** `GET/POST /mypage/profile` (신규) |
| 서비스 | `loadProfileForEdit`, `updateUserProfile`, `applyProfile` |
| DB | `V28__terms_minimum_age_19.sql`, `V29__terms_minimum_age_19_year_basis.sql` |

### 3.2 UI

| 화면 | 동작 |
|------|------|
| 랜딩·로그인 | 만 19세 안내 문구 |
| `/profile/setup` | fragment `form-fields`, `profile-age-guard.js`, 버튼 **「만 19세이상입니다」**, `max` 생년월일 |
| `/mypage/profile` | 동일 연령 가드·마이페이지 링크 |

클라이언트: 생년월일 미충족 시 제출 버튼 `disabled` + submit 시 재검증.

### 3.3 문서

- [`artifact/prd/terms-consent.md`](../../artifact/prd/terms-consent.md) — §3.1 C항 「만 19세 이상 확인 (해당 연도 1월 1일 기준)」

---

## 4. 검증

```bash
cd com.mockdan.life-saver-lotto
./gradlew test --tests "com.mockdan.lifesaverlotto.identity.policy.AgePolicyTest" \
  --tests "com.mockdan.lifesaverlotto.identity.service.impl.ProfileSetupServiceImplTest" \
  --tests "com.mockdan.lifesaverlotto.identity.api.ProfileSetupControllerTest" \
  --tests "com.mockdan.lifesaverlotto.ui.ProfileEditControllerTest"
```

- `AgePolicyTest` — 연도·1/1 기준·`latestAllowedBirthDate`
- 프로필 setup/edit 컨트롤러·서비스 단위 테스트 green

---

## 5. PR

| 저장소 | 브랜치 | 비고 |
|--------|--------|------|
| `com.mockdan.life-saver-lotto` | `feat/minimum-age-19-policy` | IMPLEMENT |
| `jungmockdan.github.com` | `docs/devlog-2026-05-17` | 본 일지 |
| `life-saver-lotto-workspace` | (선택) `terms-consent.md` only | PRD SoT |

---

## 6. 다음

- [ ] Playwright — `/profile/setup`·`/mypage/profile` 생년월일 E2E
- [ ] 기존 가입자 중 미성년 생년월일 보유 시 차단 정책(필요 시)
- [ ] TERM-003 개정에 따른 기존 동의자 재동의 여부(법무·운영)

---

## 7. 기준

- **연령 SoT**: `AgePolicy` + PRD `terms-consent.md` §3.1
- **수정 URL**: `/mypage/profile` — `/profile/setup`은 최초 가입(세션 `pending_social_profile`) 전용
- **저장소 경계**: Spring·Flyway = `com.mockdan.life-saver-lotto/`; artifact PRD = 워크스페이스 repo
