---
title: "개발일지 — 2026-05-26 (Ad·미션 verify · API rate limit)"
excerpt: "BM: 직접결제 없음·광고·미션 REWARD. AD-REWARD verify PASS. SECURITY-API-ABUSE-01 전역 rate limit · 보드 아카이브."
categories: [deVlog]
tags: [planet645, security, missions, adsense, verify, rate-limit, 개발일지]
toc: true
toc_sticky: true
date: 2026-05-26 09:00:00 +0900
last_modified_at: 2026-05-26 21:00:00 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** **BM** — 당분간 **직접결제·PG 없음**(Billing dormant), **광고(AdSense)·미션 REWARD**가 수익·크레딧 축. **Ad·미션** 수동 verify **PASS**. **API abuse rate limit** 전역 본편 · `SECURITY-API-ABUSE-01` 아카이브.

## 1. 오늘 목표

### Planet645

- [x] `AD-REWARD-MISSIONS-01` 수동 verify — runbook R01–R10
- [x] `SECURITY-API-ABUSE-01` 전역 rate limit (미션 slice와 정합)
- [x] `./gradlew test` — ratelimit·mission 패키지 green

### 개발 운영

- [x] verify run record 갱신 · `api-abuse-rate-limit-design-delta.md` · 보드 `238` 아카이브

---

## 2. 주요 변경 사항

### 2.0 제품 BM — 직접결제 중단 · 광고 수익화 (Planet645)

| 항목 | 방향 |
|------|------|
| 직접결제·PG | **당분간 없음** — `BILLING-DORMANT-01` · Sandbox/Refund 보드 **cancelled** |
| 수익·크레딧 | **AdSense** + **미션 REWARD** (`AD-REWARD-MISSIONS-01`, `TERMS-AD-TRACKING-01`) |
| 토스·Payletter 등 | ~~사전 입점 문의~~ — **중단**(5/22 조사는 [이력](2026-05-22-developer-log.md)만 유지) |

### 2.1 AD-REWARD-MISSIONS-01 — 수동 verify (Planet645)

**환경:** `docker compose -f docker-compose.yml`만 사용(`override` 제외 → dev-login 200). `/tmp/ad-reward-compose-env.yml`로 `MISSION_ENABLED` · `ADSENSE_ENABLED` on/off.

| ID | 판정 | 요약 |
|----|------|------|
| R01–R03 | PASS | 기동·미션 on/off·checkin REWARD earn |
| R04·R05 | skip | 일 cap·consume — 단위 테스트 대체 |
| R06–R08 | PASS | AdSense off/on·drawer/nav 무광고 |
| R09–R10 | PASS | prod CSP·`gradlew` mission/entitlement/headers 테스트 |

SoT: [`runs/2026-05-24-AD-REWARD-MISSIONS-01.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/ops/verify/runs/2026-05-24-AD-REWARD-MISSIONS-01.md) (실행일 **2026-05-26 KST**).

### 2.2 SECURITY-API-ABUSE-01 — 전역 API rate limit (Planet645)

**선행:** 미션 slice(`MissionRateLimitInterceptor`) — checkin 10/h · share 20/h · status 60/min.

**전역 본편** (`com.mockdan.lifesaverlotto.cmm.ratelimit`):

| Policy | Path | Limit | Key |
|--------|------|-------|-----|
| LOGIN | `/login`, `/login/**` | 60 / 15m | IP |
| OAUTH2 | `/oauth2/**` | 120 / 1h | IP |
| OPEN | `/open/**` | 200 / 1m | IP |
| API v1 | `/api/v1/**` | 300 / 1m | userId 또는 IP |

**제외:** `/api/v1/missions/**` — slice와 이중 limit 방지.

| 구성 | 역할 |
|------|------|
| `FixedWindowRateLimiter` | Redis INCR + TTL · in-memory 폴백 |
| `ApiAbuseRateLimitFilter` | `SecurityContextHolderFilter` **이후** |
| `MissionRateLimitService` | 동일 limiter **위임** |

**응답:** HTTP 429 · `application/problem+json` · `LOTTO-429`.

**설정:** `app.security.rate-limit.*` (`API_RATE_LIMIT_ENABLED`).

SoT: [`api-abuse-rate-limit-design-delta.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/design/security/api-abuse-rate-limit-design-delta.md), [`mission-api-rate-limit-slice.md`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/design/security/mission-api-rate-limit-slice.md).

### 2.3 보드·design SoT (개발 운영)

| 태스크 | 결과 |
|--------|------|
| `SECURITY-API-ABUSE-01` (`238`) | slice + 전역 완료 → **archived** |
| 백로그 다음 | `SECURITY-DEPENDENCY-SUPPLY-01` (`239`) · `SECURITY-SECRETS-GOVERNANCE-01` (`243`) |

---

## 3. 문제와 해결

### 3.1 Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| verify 시 dev-login 404 | `docker-compose.override.yml`이 prod-profile | runbook대로 **base compose만** + env 오버레이 |
| 미션·전역 limit 이중 적용 | 동일 `/api/v1/missions/**` | 전역 `ApiAbuseRateLimitPaths`에서 **명시 제외** |

### 개발 운영

_(해당 없음)_

---

## 4. 배운 점

- 수동 verify는 **compose 프로파일**이 dev-login·판정을 바꾼다 — runbook에 `override` 제외를 고정할 것.
- Rate limit 스택은 **공용 limiter → slice 인터셉터 + 전역 필터** 순으로 쌓고, 경로 겹침은 코드에서 제외한다.
- **PG 심사 일정**과 제품 일정을 분리했던 5/22 가정은, **광고 BM**으로 바뀌면서 **직접결제 follow-up은 보드에서 제거**하는 편이 SoT와 맞다.

---

## 5. 다음 액션

> SoT: `artifact/ops/tasks/board.md`.

### 5.1 미완

#### Planet645

- [ ] `SECURITY-DEPENDENCY-SUPPLY-01` — CVE·SBOM·dependencyCheck (`239`)
- [ ] `SECURITY-SECRETS-GOVERNANCE-01` — 시크릿·키 로테이션 runbook (`243`)
- [ ] Ad·미션 **베타 노출** 정책(플래그·약관 동의) — prod-profile 회귀 후 판단

#### 개발 운영

- [ ] (선택) 사업자등록 — **광고·서비스 운영**용; 유료 구독·통신판매업 신고는 **PG 재개 시**까지 보류
- [ ] (선택) 전역 rate limit **수동 verify** runbook 1건

### 5.2 완료 (당일 — 상세 §2)

| §2 | 항목 |
|----|------|
| [2.0](#20-제품-bm--직접결제-중단--광고-수익화-planet645) | BM — 직접결제 중단 · 광고·미션 |
| [2.1](#21-ad-reward-missions-01--수동-verify-planet645) | AD-REWARD-MISSIONS-01 verify **PASS** |
| [2.2](#22-security-api-abuse-01--전역-api-rate-limit-planet645) | SECURITY-API-ABUSE-01 전역 본편 |
| [2.3](#23-보드design-sot-개발-운영) | design delta · 보드 아카이브 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| Spring | `com.mockdan.life-saver-lotto` — `cmm.ratelimit.*`, `MissionRateLimitService` 위임 |
| Verify | [`AD-REWARD-MISSIONS-01`](https://github.com/jungmockdan/com.mockdan.life-saver-lotto-workspace/blob/main/artifact/ops/verify/runs/2026-05-24-AD-REWARD-MISSIONS-01.md) |
| 환경 표기 | **로컬 prod-profile 회귀** — staging 표기 없음 |
| 전일 일지 | [2026-05-24-developer-log.md](2026-05-24-developer-log.md) |
